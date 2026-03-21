---
description: Generate an Android release keystore (.jks) and key.properties file for a Flutter project — detects project name, checks for existing keystores, configures build.gradle signing, and ensures .gitignore entries.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
argument-hint: "(no arguments needed)"
---

# Create Android Release Keystore

Generate a `.jks` keystore and `key.properties` for Android release signing, following Codeable Flutter CLI conventions.

## Step 1: Detect Project Context

1. Read `pubspec.yaml` to get the project name (snake_case `name:` field)
2. Check if `android/app/build.gradle` or `android/app/build.gradle.kts` exists
3. Search for existing `*.jks` files in `android/app/`
4. Check if `android/key.properties` already exists

## Step 2: Handle Existing Keystore

If a `.jks` file OR `key.properties` already exists, ask:

```
Question: "A keystore already exists in this project. What would you like to do?"
Header: "Existing Keystore Found"
Options:
  - label: "Overwrite"
    description: "Delete the existing keystore and create a new one (WARNING: you will lose access to any apps signed with the old keystore)"
  - label: "Cancel"
    description: "Keep the existing keystore, do nothing"
```

If "Cancel": stop. If "Overwrite": delete old `.jks` files in `android/app/` and old `key.properties`.

If no existing keystore, skip this step.

## Step 3: Generate Keystore

Derive from `pubspec.yaml` name:
- **Keystore file**: `android/app/{name}-keystore.jks`
- **Alias**: `{name}-alias`
- **Passwords**: `android` (both store and key)
- **Org name**: derive from `applicationId` in build.gradle (e.g. `com.muebly.app` → `Muebly`), or capitalize project name

Run:
```bash
keytool -genkey -v \
  -keystore android/app/{name}-keystore.jks \
  -keyalg RSA -keysize 2048 -validity 10000 \
  -alias {name}-alias \
  -dname "CN={name},OU=Dev,O={orgName},L=Unknown,ST=Unknown,C=US" \
  -storepass android -keypass android
```

If `keytool` is not found, tell the user to install Java JDK.

## Step 4: Create key.properties

Write `android/key.properties`:
```
storePassword=android
keyPassword=android
keyAlias={name}-alias
storeFile={name}-keystore.jks
```

## Step 5: Verify build.gradle Signing Config

Read the project's build.gradle(.kts). Check if it already has:
1. `keystoreProperties` / `key.properties` loading block
2. `signingConfigs` with `release` config
3. Release build type using `signingConfigs.release`

**If already present**: report no changes needed.

**If missing**: add signing config. For **build.gradle.kts**:
```kotlin
import java.util.Properties
import java.io.FileInputStream

val keystoreProperties = Properties()
val keystorePropertiesFile = rootProject.file("key.properties")
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(FileInputStream(keystorePropertiesFile))
}

// Inside android { }:
signingConfigs {
    create("release") {
        if (System.getenv("ANDROID_KEYSTORE_PATH") != null) {
            storeFile = file(System.getenv("ANDROID_KEYSTORE_PATH"))
            keyAlias = System.getenv("ANDROID_KEYSTORE_ALIAS")
            keyPassword = System.getenv("ANDROID_KEYSTORE_PRIVATE_KEY_PASSWORD")
            storePassword = System.getenv("ANDROID_KEYSTORE_PASSWORD")
        } else {
            keyAlias = keystoreProperties["keyAlias"] as String?
            keyPassword = keystoreProperties["keyPassword"] as String?
            storeFile = keystoreProperties["storeFile"]?.let { file(it) }
            storePassword = keystoreProperties["storePassword"] as String?
        }
    }
}
```

For **build.gradle** (Groovy):
```groovy
def keystoreProperties = new Properties()
def keystorePropertiesFile = rootProject.file('key.properties')
if (keystorePropertiesFile.exists()) {
    keystoreProperties.load(new FileInputStream(keystorePropertiesFile))
}

signingConfigs {
    release {
        if (System.getenv("ANDROID_KEYSTORE_PATH")) {
            storeFile file(System.getenv("ANDROID_KEYSTORE_PATH"))
            keyAlias System.getenv("ANDROID_KEYSTORE_ALIAS")
            keyPassword System.getenv("ANDROID_KEYSTORE_PRIVATE_KEY_PASSWORD")
            storePassword System.getenv("ANDROID_KEYSTORE_PASSWORD")
        } else {
            keyAlias keystoreProperties['keyAlias']
            keyPassword keystoreProperties['keyPassword']
            storeFile keystoreProperties['storeFile'] ? file(keystoreProperties['storeFile']) : null
            storePassword keystoreProperties['storePassword']
        }
    }
}
```

## Step 6: Ensure .gitignore Entries

Check `.gitignore` and add if missing (no duplicates):
```
*.jks
**/android/key.properties
```

## Step 7: Report

```
Keystore created successfully!

  Keystore:    android/app/{name}-keystore.jks
  Alias:       {name}-alias
  Password:    android (both store and key)
  Validity:    10,000 days (~27 years)
  Algorithm:   RSA 2048-bit
  Properties:  android/key.properties
  Signing:     build.gradle {already configured / updated}
  Gitignore:   {already had entries / entries added}

Both files are excluded from git. For CI/CD, use environment variables:
  ANDROID_KEYSTORE_PATH, ANDROID_KEYSTORE_ALIAS,
  ANDROID_KEYSTORE_PASSWORD, ANDROID_KEYSTORE_PRIVATE_KEY_PASSWORD
```
