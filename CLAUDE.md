# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

**Revolver** is an immutable event-based state management framework for **Kotlin Multiplatform** (Android + iOS). It enforces unidirectional data flow using a State/Event/Effect pattern.

## Commands

### Build
```bash
./gradlew build
./gradlew publishToMavenLocal   # or: ./scripts/build_deploy_to_local_maven.sh
```

### Tests
```bash
./scripts/run_test_suite.sh          # All platforms (Android + iOS simulator)
./gradlew testDebugUnitTest          # Android only
./gradlew iosSimulatorArm64Test      # iOS only
```

### Lint
```bash
./gradlew ktlintCheck
./gradlew ktlintFormat
```

### Publish to GitHub Packages
```bash
GH_USERNAME=<user> GH_TOKEN=<token> ./gradlew publishAllPublicationsToGitHubPackagesRepository
```

## Architecture

### Core Pattern: Event → Handler → State/Effect

```
Client → emit(Event) → RevolverViewModel → EventHandler → Emitter.state() / Emitter.effect()
                                                         → StateFlow<STATE> / SharedFlow<EFFECT> → Clients
```

### Key Abstractions (all in `revolver/src/commonMain/`)

| Class | Role |
|-------|------|
| `RevolverViewModel` | Abstract base. Holds `state: StateFlow<STATE>` and `effect: SharedFlow<EFFECT>`. Clients call `emit(event)` to trigger handlers. |
| `RevolverEvent` | Sealed interface — messages sent from UI to ViewModel |
| `RevolverState` | Sealed interface — immutable snapshot of UI state |
| `RevolverEffect` | Sealed interface — one-shot side effects (navigation, toasts, etc.) |
| `EventHandler` | Suspend function registered in the ViewModel to process a specific event type via `Emitter` |
| `ErrorHandler` | Registered exception handler; matches exception type, emits error state/effect |
| `Emitter<STATE, EFFECT>` | Passed to handlers; provides `state()` and `effect()` emit functions |

### Platform Modules

- `commonMain` — all framework logic
- `androidMain` — integrates with AndroidX `ViewModel` lifecycle
- `iosMain` — exposes `CFlow`, `CStateFlow`, `CSharedFlow` wrappers for Swift consumption

### Module Layout

```
revolver/          # Library module (published artifact)
examples/basic/    # Minimal Android integration example
```

### Testing Conventions

Tests use **Turbine** for Flow assertions and **Mockative** for mocking. Test files live in `revolver/src/commonTest/`. Prefer testing via the public `emit()` API and asserting on emitted states/effects using Turbine's `test {}` block.
