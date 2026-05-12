---
name: flutter-arb-sorter
description: Sort every Flutter localization .arb file in an l10n directory alphabetically and validate localization integrity. Use when Codex is asked to organize, sort, reorder, clean, or standardize Flutter ARB localization files, app localization resources, or files under lib/l10n, l10n, or configured Flutter gen-l10n arb-dir paths. After sorting, run flutter pub get and flutter analyze to confirm generated localization inputs are complete and no keys are missing.
---

# Flutter ARB Sorter

## Workflow

1. Locate the Flutter project root by finding `pubspec.yaml`.
2. Locate the ARB directory:
   - Prefer `arb-dir` from `l10n` when present.
   - Otherwise use `lib/l10n` when it exists.
   - Otherwise use `l10n` when it exists.
   - If multiple likely directories exist and no config chooses one, ask the user which directory to sort.
3. Sort every `*.arb` file in the chosen directory alphabetically.
4. Preserve valid JSON formatting and write files with a trailing newline.
5. Run `flutter pub get`.
6. Run `flutter analyze`.
7. Report any validation failures with the exact command and relevant error summary.

## Sorting Rules

Sort top-level ARB entries alphabetically by message key, case-insensitive with stable ordering for ties.

- Keep each metadata entry with its message entry. For key `loginButton`, keep `@loginButton` immediately after `loginButton`.
- If a metadata entry appears without a matching message key, sort it alphabetically as its own key and flag it in the summary.
- Keep special global metadata keys, such as `@@locale`, sorted alphabetically with the rest unless the project already keeps them at the top consistently.
- Do not rename keys, change values, translate strings, delete placeholders, or alter metadata contents.
- Do not sort nested JSON objects inside metadata unless the user explicitly asks for deeper normalization.
- Preserve the project's indentation style when obvious; otherwise use two spaces.

Example:

```json
{
  "@@locale": "en",
  "cancel": "Cancel",
  "save": "Save",
}
```

## Completeness Checks

After sorting, compare ARB keys across all locale files before running Flutter commands.

- Treat message keys and metadata keys separately.
- For each locale file, list missing message keys compared with the union of all message keys.
- List metadata entries that do not have a matching message key in the same file.
- Do not invent missing translations or metadata unless the user explicitly asks.
- If missing keys already existed before sorting, report them as pre-existing rather than caused by sorting when the diff confirms no deletion.

## Validation Commands

Run from the Flutter project root:

```bash
flutter pub get
flutter analyze
```

If the project uses a wrapper or version manager, use the project convention instead, such as `fvm flutter pub get` and `fvm flutter analyze`.

If either command is unavailable or fails for environment reasons, explain the blocker and include any completed static ARB key comparison.

## Safety

Only edit `.arb` files in the selected l10n directory.

Before editing, inspect existing diffs when the repository is dirty. Do not revert unrelated changes. After editing, verify the diff only reorders ARB entries and preserves values.
