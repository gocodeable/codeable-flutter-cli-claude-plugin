---
description: Generate a form screen with validation, text controllers, form key, and submission wired to a cubit method. Supports text fields, dropdowns, date/time pickers, and image pickers.
allowed-tools: Read, Write, Edit, Bash, Glob, Grep, AskUserQuestion
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
- `text` — standard text field
- `email` — email text field with validation
- `password` — password field with toggle
- `phone` — phone number field
- `number` — numeric input
- `multiline` — multi-line text area
- `date` — date picker
- `time` — time picker
- `dropdown` — dropdown selector
- `image` — image picker
- `switch` — toggle switch
- `checkbox` — checkbox

If fields not provided, ask the user.

## Step 1: Detect Project Configuration

Read the feature's existing cubit, state, and available core widgets.

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
      appBar: mueblyAppBar(context: context, title: '{Form Title}'),
      body: BlocConsumer<{Prefix}Cubit, {Prefix}State>(
        listenWhen: (prev, curr) => prev.submitData != curr.submitData,
        listener: (context, state) {
          if (state.submitData.isLoaded) {
            ToastHelper.showSuccessToast('Saved successfully');
            context.pop();
          }
          if (state.submitData.isFailure) {
            ToastHelper.showErrorToast(state.submitData.errorMessage ?? 'Failed');
          }
        },
        builder: (context, state) {
          return SingleChildScrollView(
            padding: const EdgeInsets.all(16),
            child: Form(
              key: _formKey,
              child: Column(
                children: [
                  MueblyTextField(
                    controller: _nameController,
                    labelText: 'Name',
                    validator: (v) => v?.isEmpty ?? true ? 'Required' : null,
                  ),
                  const SizedBox(height: 16),
                  MueblyTextField(
                    controller: _emailController,
                    type: MueblyTextFieldType.email,
                    labelText: 'Email',
                  ),
                  const SizedBox(height: 24),
                  MueblyButton(
                    text: 'Submit',
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
- Use `BlocConsumer` for forms (listener for success/error feedback, builder for loading).
- Use `MueblyTextField` from core widgets.
- Use `MueblyButton` with `isLoading` for submit.
- Dispose all controllers in `dispose()`.
- Use `Form` with `GlobalKey<FormState>` for validation.
- Trim text inputs before submitting.
