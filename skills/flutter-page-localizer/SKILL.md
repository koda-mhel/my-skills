---
name: flutter-page-localizer
description: Add user-facing strings from a Flutter page's view and widget files to all ARB localization files, then refactor the files to use Sindbad Flutter's AppLocalizations import. Use this skill when Codex needs to localize a page, extract strings for ARB files, or replace hardcoded strings in Flutter views and widgets while leaving globally understood numbers, money, currency, percentages, abbreviations, and similar symbolic content unlocalized.
---

Add localization strings for a page to all `lib/l10n/app_*.arb` files, then refactor the page view and widget files to use `AppLocalizations`.

## Step 1 - Identify the page

Resolve the page directories from the page name:
- `lib/pages/<snake_name>/view/`
- `lib/pages/<snake_name>/widgets/` (if it exists)

Read all Dart files inside **both** directories before proceeding. If `widgets/` does not exist, continue with view files only.

## Step 2 - Clarify preferences before extracting

Before scanning strings, ask the user these two questions in a single message:

**Question A — Number/value localization**
> Some strings contain numbers, percentages, amounts, or other numeric values (e.g. `"25%"`, `"$99.99"`, `"42 days"`). Should I localize the surrounding wording and keep the value as a placeholder, or skip these entirely and leave the whole string hardcoded?
>
> - **A1**: Localize the wording, keep the value as a `{placeholder}` *(default)*
> - **A2**: Skip all strings that contain numeric or symbolic values entirely

**Question B — Non-English ARB translation**
> I found the following non-English ARB files: `<list discovered app_*.arb files excluding app_en.arb>`. For new keys added to these files, should I:
>
> - **B1**: Copy the English value as-is and leave translation as a manual follow-up *(default)*
> - **B2**: Attempt to translate each value into the file's target language

Wait for the user's answers before continuing. If the user says "default" or skips the question, apply A1 and B1.

## Step 3 - Find all user-facing strings

Scan every Dart file collected in Step 1 — both view files and widget files. Extract strings from `Text()`, `RichText`, `TextField` labels and hints, button children, `AppBar` titles, snack bars, dialogs, tooltips, validation messages, and other user-visible UI copy.

### Extract

- Static display text: labels, titles, subtitles, hints, button text, empty states, warnings, helper text, error messages, tooltips
- Strings with dynamic interpolated parts when the surrounding static text carries UI meaning
- Sentences, phrases, and short UI copy
- Strings containing numeric values — apply the rule from **Question A** answer

### Skip

- Pure standalone proper names such as person names, brand names, app names, and product identifiers with no surrounding static copy
- Pure numbers or numeric-looking values with no surrounding static copy
- File paths, routes, API keys, enum values, database fields, asset names, font family names, and other technical identifiers
- Code-like strings and strings used only for analytics, logging, or programmatic lookup
- If **A2** was chosen: any string that contains a number, percentage, currency symbol, measurement, date, time, duration, phone number, OTP, ID, or version number

Rule of thumb: localize wording that explains, instructs, labels, or warns. Do not localize raw values or symbolic content that is already understandable globally.

Examples:
- Extract: `"Enter your mobile number"`, `"Your transfer was successful"`, `"Total amount"`
- Skip: `"42"`, `"$99.99"`, `"25%"`, `"v1.0.2"`, `"+9715551234"`
- Mixed case (A1): localize the wording, keep the value as a placeholder. `"Discount 25%"` → key `discountValue`, value `"Discount {value}"`
- Mixed case (A2): skip the whole string, leave it hardcoded

## Step 4 - Read existing ARB files

1. Read `lib/l10n/app_en.arb` to collect existing keys.
2. Discover all other ARB files matching `lib/l10n/app_*.arb`.
3. Reuse an existing key from `app_en.arb` when it already matches the needed text. Do not add duplicate entries.

## Step 5 - Generate keys and values

### Key rules

- Convert the text to `camelCase`
- Strip punctuation from the key
- **Mimic the value**: the key should read like a shortened natural version of the string, preserving word order — do not reorder, abstract, or rename words arbitrarily
- **Maximum 10 words** in the key
- For strings under 10 words, use every meaningful word from the value
- For strings over 10 words, trim from the end, keeping the first 9–10 meaningful words
- Remove only filler words (articles, conjunctions, prepositions) when needed to stay within the 10-word limit, and only from the tail

### Static string examples

| String | Key | Value |
|---|---|---|
| `"Update"` | `update` | `"Update"` |
| `"Cancel"` | `cancel` | `"Cancel"` |
| `"Enter your mobile number"` | `enterYourMobileNumber` | `"Enter your mobile number"` |
| `"Don't have an account"` | `dontHaveAnAccount` | `"Don't have an account"` |
| `"How much are you willing to invest?"` | `howMuchAreYouWillingToInvest` | `"How much are you willing to invest?"` |
| `"Please review and confirm your details before submitting the form"` | `pleaseReviewAndConfirmYourDetailsBeforeSubmitting` | `"Please review and confirm your details before submitting the form"` |

### Dynamic strings

For strings containing dynamic parts such as `$variable`, `${expr}`, or concatenated values:

- Replace each dynamic part with a named `{placeholder}` whose name matches the variable or describes the value
- Generate the key from the static words only, following the same mimic + 10-word rules
- Keep globally understood values as placeholders instead of localizing the value itself
- Add `@<key>` metadata with a `placeholders` block immediately after the entry

| String | Key | ARB Value |
|---|---|---|
| `"Hello, $name"` | `helloName` | `"Hello, {name}"` |
| `"Version $version"` | `version` | `"Version {version}"` |
| `"Total: $amount"` | `totalAmount` | `"Total: {amount}"` |
| `"Discount 25%"` (A1) | `discountValue` | `"Discount {value}"` |

ARB format:
```json
"helloName": "Hello, {name}",
"@helloName": {
  "placeholders": {
    "name": { "type": "String" }
  }
}
```

Dart usage:
```dart
AppLocalizations.of(context)?.helloName(name) ?? ''
```

## Step 6 - Update all ARB files

Apply the same new entries to every `lib/l10n/app_*.arb` file found in Step 4.

Rules:
- Use the original English string in `app_en.arb`
- For non-English ARB files, apply the rule from **Question B** answer:
  - **B1**: Copy the English value as-is into non-English files; mark each copied entry with a trailing comment `// TODO: translate` in the report (ARB files themselves do not support comments — note these in the Step 10 report only)
  - **B2**: Translate each value into the target language inferred from the filename (e.g. `app_ar.arb` → Arabic, `app_fr.arb` → French); if uncertain about any translation, flag it in the Step 10 report
- Keep the same placeholders in every ARB file
- Keep keys in alphabetical order
- Keep punctuation in values even when punctuation is removed from keys
- Place each `@<key>` metadata block immediately after its string entry

## Step 7 - Regenerate localizations

Run:

```bash
flutter pub get
```

## Step 8 - Refactor the files

Return to every Dart file read in Step 1 — both view files and widget files. Replace each extracted hardcoded string with the matching `AppLocalizations` call, whether the key was newly added or reused.

### Replacement patterns

Static string:

```dart
// Before
Text('Enter your mobile number')

// After
Text(AppLocalizations.of(context)?.enterYourMobileNumber ?? '')
```

Dynamic string:

```dart
// Before
Text('Hello, $name!')

// After
Text('${AppLocalizations.of(context)?.helloName(name) ?? ''}!')
```

Hint or label text:

```dart
// Before
InputDecoration(hintText: 'Enter your mobile number')

// After
InputDecoration(
  hintText: AppLocalizations.of(context)?.enterYourMobileNumber ?? '',
)
```

App bar title:

```dart
// Before
AppBar(title: Text('Settings'))

// After
AppBar(title: Text(AppLocalizations.of(context)?.settings ?? ''))
```

### Refactor rules

- Do not change skipped strings
- Do not modify logic, layout, or unrelated imports
- **Never assign `AppLocalizations.of(context)` to a local variable.** Do not write `final l10n = AppLocalizations.of(context);` or any equivalent reusable alias. Every call must be written out inline, in full, at the point of use.
- **Always append `?? ''` to every `AppLocalizations` call**, even when the string is used as a non-nullable parameter. Never omit the null fallback.
- Add this import to every file that now uses `AppLocalizations`, if missing:

```dart
import 'package:sindbad_flutter/l10n/app_localizations.dart';
```

- Remove the generated l10n import if the file used `package:flutter_gen/gen_l10n/app_localizations.dart` only for this purpose
- Pass `BuildContext` down only when necessary; otherwise report it as a manual follow-up

## Step 9 - Analyze

Run:

```bash
flutter analyze
```

## Step 10 - Report

Provide:

1. **Preferences applied** — which options were chosen for Question A and Question B
2. **New keys added** — key name, whether static or dynamic, and English value
3. **Existing keys reused**
4. **ARB files updated** — for non-English files under B1, list keys that still need translation
5. **Translations flagged** — any B2 translations that were uncertain or approximated
6. **Files refactored** — file name, whether it is a view or widget file, and count of replacements
7. **Skipped strings** — the string, the file it came from, and the reason it was skipped
8. **Manual follow-ups** — any string that could not be automatically refactored (e.g. missing `BuildContext`)