---
name: flutter-constants-sorter
description: Sort a single specified Flutter/Dart constants file alphabetically. Use when Codex is asked to organize, sort, reorder, or clean constants in a Flutter constants directory, especially files containing app constants, asset path constants, image constants, icon constants, font constants, or other static const declarations. Requires the user to provide the exact target file name before editing.
---

# Flutter Constants Sorter

## Required Target

Before doing any file inspection or edit, require an exact file name from the user.

- Accept only one specific file name, such as `app_assets.dart` or `route_constants.dart`.
- The target file must be inside a constants directory, usually named `constants`, `constant`, `const`, or a project-specific equivalent.
- If the user does not provide a file name, stop and ask for it.
- If more than one matching file is found, stop and ask the user to choose the exact path.
- Do not sort every file in the constants directory unless the user explicitly provides each file name.

## Workflow

1. Locate the target constants file by exact file name.
2. Read the file and identify the sortable declarations.
3. Preserve file-level directives and structure, including `import`, `export`, `part`, `part of`, `library`, `ignore_for_file`, class declarations, constructors, and surrounding braces.
4. Sort only peer declarations inside the relevant constants scope.
5. Preserve each declaration's leading documentation or inline comments with that declaration.
6. Run the project's normal formatter when available, usually `dart format <file>`.
7. Verify the diff only reorders constants and removes obsolete asset grouping comments when needed.

## General Sorting

Sort constants alphabetically by declared identifier.

- Sort `const`, `final`, `static const`, and `static final` declarations only when they are simple peer constants.
- Use case-insensitive alphabetical order, with stable ordering for names that differ only by case.
- Keep unrelated members, methods, constructors, and nested types in place.
- Preserve existing meaningful section comments unless they conflict with a single alphabetical constants list.

Example order:

```dart
static const String apiBaseUrl = '...';
static const String appName = '...';
static const String supportEmail = '...';
```

## Asset Constants

When the target file is an asset constants file, sort all asset declarations together alphabetically by declared identifier. Do not group by file type and do not add file type comments.

Treat a file as an asset constants file when the file name, class name, or constant values clearly refer to assets such as images, icons, animations, fonts, audio, video, or files under an `assets/` path.

Example order:

```dart
static const String appLogo = 'assets/images/app_logo.png';
static const String chevronDown = 'assets/icons/chevron_down.svg';
static const String emptyState = 'assets/images/empty_state.png';
static const String profilePhoto = 'assets/images/profile_photo.jpg';
```

Asset sorting rules:

- Sort all asset constants in one list by identifier.
- Do not separate constants by extension, folder, or asset category.
- Do not add comments such as `// .png`, `// .svg`, or `// other`.
- Remove existing file type grouping comments when they only label extension groups.
- Preserve documentation comments that describe a specific declaration.

## Safety

Do not infer a target file from broad requests such as "sort constants" or "clean assets". Ask for the exact file name first.

If sorting would require parsing complex expressions, conditional declarations, generated code, or mixed business logic, keep the edit conservative and explain the skipped sections.
