---
description: Generate a form screen with validation, text controllers, form key, and submission wired to a cubit method. Supports text fields, dropdowns, date/time pickers, and image pickers.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion, TodoWrite
argument-hint: "<feature_path> <form_name> --fields 'name:text,email:email,date:date,image:image'"
---

# Generate Form Screen

Create a form screen with validation, controllers, and cubit submission for a codeable_cli feature.

## Arguments

Parse `$ARGUMENTS` for:
- **Feature path**: e.g., `lib/features/customer/customer_orders`
- **Form name**: e.g., `create_order`, `edit_profile`
- **Fields**: Comma-separated `name:type` pairs

Supported field types:
- `text` -- standard text field
- `email` -- email text field with validation
- `password` -- password field with toggle
- `phone` -- phone number field
- `number` -- numeric input
- `multiline` -- multi-line text area
- `date` -- date picker
- `time` -- time picker
- `dropdown` -- dropdown selector
- `image` -- image picker
- `switch` -- toggle switch
- `checkbox` -- checkbox

If fields not provided, ask the user.

## Step 1: Detect Project Configuration

Read the feature's existing cubit, state, and available core widgets (CustomTextField, CustomButton, etc.).

## Step 2: Create Form Screen

`presentation/views/{prefix}_{form}_screen.dart`:

```dart
class {Prefix}{Form}Screen extends StatefulWidget {
  const {Prefix}{Form}Screen({super.key});

  @override
  State<{Prefix}{Form}Screen> createState() => _{Prefix}{Form}ScreenState();
}

class _{Prefix}{Form}ScreenState extends State<{Prefix}{Form}Screen> {
  final _formKey = GlobalKey<FormState>();
  late final TextEditingController _nameController;
  late final TextEditingController _emailController;
  File? _selectedImage;

  @override
  void initState() {
    super.initState();
    _nameController = TextEditingController();
    _emailController = TextEditingController();
  }

  @override
  void dispose() {
    _nameController.dispose();
    _emailController.dispose();
    super.dispose();
  }

  void _onSubmit() {
    if (!_formKey.currentState!.validate()) return;
    context.read<{Prefix}Cubit>().submitForm(
      name: _nameController.text.trim(),
      email: _emailController.text.trim(),
      image: _selectedImage,
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: CustomAppBar(context: context, title: context.l10n.formTitle),
      body: BlocConsumer<{Prefix}Cubit, {Prefix}State>(
        listenWhen: (prev, curr) => prev.submitData != curr.submitData,
        listener: (context, state) {
          if (state.submitData.isLoaded) {
            ToastHelper.showSuccessToast(context.l10n.savedSuccessfully);
            context.pop();
          }
          if (state.submitData.isFailure) {
            ToastHelper.showErrorToast(
              state.submitData.errorMessage ?? context.l10n.somethingWentWrong,
            );
          }
        },
        builder: (context, state) {
          return SingleChildScrollView(
            padding: const EdgeInsetsDirectional.symmetric(
              horizontal: 16,
              vertical: 16,
            ),
            child: Form(
              key: _formKey,
              child: Column(
                children: [
                  CustomTextField(
                    controller: _nameController,
                    labelText: context.l10n.name,
                    validator: (v) =>
                        v?.isEmpty ?? true ? context.l10n.fieldRequired : null,
                  ),
                  const SizedBox(height: 16),
                  CustomTextField(
                    controller: _emailController,
                    type: CustomTextFieldType.email,
                    labelText: context.l10n.email,
                  ),
                  const SizedBox(height: 24),
                  CustomButton(
                    text: context.l10n.submit,
                    isLoading: state.submitData.isLoading,
                    disabled: state.submitData.isLoading,
                    onPressed: state.submitData.isLoading ? null : _onSubmit,
                  ),
                ],
              ),
            ),
          );
        },
      ),
    );
  }
}
```

## Step 3: Add State Field & Cubit Method

Add `DataState<void> submitData` to existing state and submission method to cubit.

## Step 4: Wire Route

Add route for the form screen.

## Step 5: Verify

Run `dart analyze lib/` and fix any issues.

## Rules

- Use `StatefulWidget` for forms (needs controllers + dispose).
- Use `BlocConsumer` for forms: **listener** for side effects (success toast, navigation, error toast), **builder** for rendering loading state on the submit button.
- Use `ToastHelper.showSuccessToast()` / `ToastHelper.showErrorToast()` for user feedback.
- Use `EdgeInsetsDirectional` for all padding/margin (never `EdgeInsets.only(left:)` or `EdgeInsets.only(right:)` -- supports RTL).
- Use `context.l10n.keyName` for ALL user-facing strings (never hardcoded strings in UI).
- Use `CustomTextField` / `CustomButton` from core widgets.
- Dispose all controllers in `dispose()`.
- Use `Form` with `GlobalKey<FormState>` for validation.
- Trim text inputs before submitting.
