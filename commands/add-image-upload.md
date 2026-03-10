---
description: Add image upload with camera/gallery picker, cropping, compression, and multipart API submission — uses the project's ImagePickerContainer and ApiService.postMultipart.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
argument-hint: "<feature_path> <field_name>"
---

# Add Image Upload

Wire complete image upload flow: pick → crop → compress → upload via multipart.

## Arguments

- `$ARGUMENTS`: Feature path and the upload field name.

## Step 1: Check Available Widgets

Check for `ImagePickerContainer` or `MueblyImagePicker` in core widgets.

## Step 2: Add State Fields

```dart
const FeatureState({
  this.selectedImage,
  this.uploadData = const DataState.initial(),
});

final File? selectedImage;
final DataState<String> uploadData; // Returns uploaded URL
```

## Step 3: Add Cubit Methods

```dart
void setImage(File image) {
  emit(state.copyWith(selectedImage: image));
}

void clearImage() {
  emit(state.copyWith(selectedImage: null));
}

Future<void> uploadImage() async {
  if (state.selectedImage == null) return;

  emit(state.copyWith(uploadData: const DataState.loading()));
  final result = await repository.uploadImage(file: state.selectedImage!);
  if (result.isSuccess && result.data != null) {
    emit(state.copyWith(uploadData: DataState.loaded(data: result.data)));
  } else {
    emit(state.copyWith(uploadData: DataState.failure(error: result.message)));
  }
}
```

## Step 4: Add Repository Method

```dart
// Interface
Future<RepositoryResponse<String>> uploadImage({required File file});

// Implementation
@override
Future<RepositoryResponse<String>> uploadImage({required File file}) async {
  try {
    final response = await _apiService.postMultipart(
      Endpoints.uploadImage,
      fileMap: {'image': file},
    );
    final result = ResponseModel.fromApiResponse(
      response, UploadResponseModel.fromJson,
    );
    if (result.isSuccess) {
      return RepositoryResponse(isSuccess: true, data: result.response?.data?.url);
    }
    return RepositoryResponse(isSuccess: false, message: result.error);
  } on AppApiException catch (e) {
    return RepositoryResponse(isSuccess: false, message: e.message);
  }
}
```

## Step 5: Add UI

Using project's ImagePickerContainer:
```dart
ImagePickerContainer(
  selectedImage: state.selectedImage,
  networkImageUrl: existingImageUrl,
  onImageSelected: (file) => context.read<Cubit>().setImage(file),
  onRemove: () => context.read<Cubit>().clearImage(),
)
```

Or manual camera/gallery selection:
```dart
GestureDetector(
  onTap: () async {
    await PermissionManager.requestCameraAndGalleryPermission(
      onGranted: () async {
        final source = await _showImageSourceSheet(context);
        if (source == null) return;
        final picker = ImagePicker();
        final picked = await picker.pickImage(
          source: source,
          maxWidth: 1080,
          maxHeight: 1080,
          imageQuality: 80, // Compress
        );
        if (picked != null) {
          context.read<Cubit>().setImage(File(picked.path));
        }
      },
      onDenied: (msg) => ToastHelper.showErrorToast(msg),
    );
  },
  child: _buildImagePreview(state.selectedImage),
)
```

## Step 6: Verify

Run `dart analyze lib/` and fix any issues.

## Rules

- Always request camera/gallery permissions before picking.
- Compress images before upload (imageQuality: 80, maxWidth: 1080).
- Use `postMultipart` for create, `putMultipart` for update.
- Show upload progress if the API supports it.
- Handle both network images (edit mode) and local files (new upload).
