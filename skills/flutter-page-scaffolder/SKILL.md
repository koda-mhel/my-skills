---
name: flutter-page-scaffolder
description: Scaffold a new Flutter feature page following the BLoC pattern used in this project
---

Scaffold a new feature page under `lib/pages/` following the project's BLoC conventions.

## Step 1 — Get feature name

If the user did not provide the feature name as an argument, ask for it now. Accept any casing (e.g. `tradingBot`, `trading_bot`, `TradingBot`). Normalize to:
- **snake_case** for directory/file names (e.g. `trading_bot`)
- **PascalCase** for class names (e.g. `TradingBot`)

## Step 2 — Create all 6 files

Create these files under `lib/pages/<snake_name>/`:

### `lib/pages/<snake_name>/<snake_name>.dart`
```dart
export 'bloc/bloc.dart';
export 'view/view.dart';
```

### `lib/pages/<snake_name>/bloc/bloc.dart`
```dart
export 'state/<snake_name>_state.dart';
export '<snake_name>_bloc.dart';
export '<snake_name>_event.dart';
```

### `lib/pages/<snake_name>/bloc/<snake_name>_event.dart`
```dart
sealed class <Pascal>Event {
  const <Pascal>Event();
}

class <Pascal>ScreenCreated extends <Pascal>Event {
  const <Pascal>ScreenCreated();
}
```

### `lib/pages/<snake_name>/bloc/state/<snake_name>_state.dart`
```dart
import 'package:freezed_annotation/freezed_annotation.dart';
import 'package:sindbad_flutter/models/models.dart';

part '<snake_name>_state.freezed.dart';

@freezed
sealed class <Pascal>State with _$<Pascal>State {
  const factory <Pascal>State({
    @Default(RequestStatus.waiting) RequestStatus screenRequestStatus,
  }) = _<Pascal>State;
}
```

### `lib/pages/<snake_name>/bloc/<snake_name>_bloc.dart`
```dart
import 'package:bloc/bloc.dart';

import '../../../models/models.dart';
import 'bloc.dart';

class <Pascal>Bloc extends Bloc<<Pascal>Event, <Pascal>State> {
  <Pascal>Bloc({
    required <Pascal>State initialState,
  }) : super(initialState) {
    on<<Pascal>ScreenCreated>(_screenCreated);
  }

  Future<void> _screenCreated(
    <Pascal>ScreenCreated event,
    Emitter<<Pascal>State> emit,
  ) async {
    emit(state.copyWith(screenRequestStatus: RequestStatus.inProgress,));
    // TODO: implement
    emit(state.copyWith(screenRequestStatus: RequestStatus.success,));
  }
}
```

### `lib/pages/<snake_name>/view/view.dart`
```dart
export '<snake_name>_page.dart';
```

### `lib/pages/<snake_name>/view/<snake_name>_page.dart`

Before writing this file, read `lib/main.dart` and check whether a `SafeArea` widget already wraps the router's shell or navigator (e.g. inside a `MaterialApp`, `Scaffold`, `ShellRoute`, or any top-level widget in the tree). 

- **If `SafeArea` is present in `main.dart`** — omit the `SafeArea` wrapper in the page; render `Scaffold` directly as the root:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

import '../bloc/bloc.dart';

class <Pascal>Page extends StatelessWidget {
  static const route = '/<snake_name>';

  const <Pascal>Page({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (context) => <Pascal>Bloc(
        initialState: const <Pascal>State(),
      )..add(const <Pascal>ScreenCreated()),
      child: Scaffold(
        body: Placeholder(),
      ),
    );
  }
}
```

- **If `SafeArea` is NOT present in `main.dart`** — wrap `Scaffold` in `SafeArea` as usual:

```dart
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';

import '../bloc/bloc.dart';

class <Pascal>Page extends StatelessWidget {
  static const route = '/<snake_name>';

  const <Pascal>Page({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (context) => <Pascal>Bloc(
        initialState: const <Pascal>State(),
      )..add(const <Pascal>ScreenCreated()),
      child: SafeArea(
        top: false,
        bottom: true,
        child: Scaffold(
          body: Placeholder(),
        ),
      ),
    );
  }
}
```

## Step 3 — Add route to lib/main.dart

Read `lib/main.dart` first. Find the existing `GoRoute` list and insert a new entry:
```dart
GoRoute(
  path: <Pascal>Page.route,
  builder: (context, state) {
    return <Pascal>Page();
  },
),
```
Also add the import for the new page at the top of main.dart if not already present:
```dart
import 'pages/<snake_name>/<snake_name>.dart';
```

## Step 4 — Run code generation

```bash
dart run build_runner build --delete-conflicting-outputs
```

## Step 5 — Run flutter analyze

```bash
flutter analyze
```

Report any errors or warnings to the user.