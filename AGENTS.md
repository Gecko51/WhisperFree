---
description: Guidelines and context for AI coding agents operating in this repository.
alwaysApply: true
---

# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, Cursor, Copilot) when working in this repository.

## Development Commands

### Prerequisites

- [Rust](https://rustup.rs/) (latest stable)
- [Bun](https://bun.sh/) package manager

### Build and Run

```bash
bun install              # Install dependencies
bun run tauri dev       # Run in development mode
bun run tauri build     # Build for production
bun run dev             # Frontend-only dev server
bun run build           # Build frontend (TypeScript + Vite)
bun run preview         # Preview built frontend
```

### Linting and Formatting

```bash
# Frontend
bun run lint            # Run ESLint
bun run lint:fix        # Fix ESLint issues
bun run format:frontend # Run Prettier

# Backend
bun run format:backend  # Run rustfmt
cd src-tauri && cargo clippy --all-targets --all-features -- -D warnings
```

### Testing Commands

#### Frontend (Playwright)

```bash
bun run test:playwright        # Run all tests
bun run test:playwright:ui     # Run with UI
bunx playwright test tests/app.spec.ts  # Single test file
bunx playwright test -g "test name"     # Single test by title
```

#### Backend (Rust)

```bash
cd src-tauri && cargo test              # Run all tests
cd src-tauri && cargo test <name>       # Run single module
cd src-tauri && cargo test -- --exact <test_name>  # Exact match
cd src-tauri && cargo test -- --nocapture  # Show print output
```

### Model Setup (Required)

```bash
mkdir -p src-tauri/resources/models
curl -o src-tauri/resources/models/silero_vad_v4.onnx https://blob.handy.computer/silero_vad_v4.onnx
```

## Code Style Guidelines

### General Principles

- Write concise, technical code focusing on modularity and declarative patterns
- Follow existing project conventions strictly before inventing new patterns
- Use `grep` to understand context before editing

### Frontend (React / TypeScript)

#### Types and Interfaces

- Use TypeScript; prefer `interface` over `type`
- Avoid enums; use maps or constant objects instead
- Avoid `any`; use `unknown` if type is truly unknown
- Shared types: `src/bindings.ts` or `src/lib/types.ts`

#### Components and Hooks

- Use Functional Components with React Hooks
- Group components logically (e.g., `components/settings/`)
- Use Zustand for global state (`src/stores/`)

#### Naming Conventions

- Directories/Files: `lowercase-kebab-case` (e.g., `model-selector.tsx`)
- Components/Interfaces: `PascalCase`
- Variables/Functions: `camelCase`
- Constants: `UPPER_SNAKE_CASE`

#### Imports

- External libraries first, then internal modules
- Prefer named imports over default imports

#### Styling

- Use Tailwind CSS exclusively
- Use `clsx` and `tailwind-merge` via `cn()` utility for conditional classes

### Backend (Rust / Tauri)

#### Rust Best Practices

- Format with `cargo fmt`, lint with `cargo clippy`
- Use proper `Result<T, E>` error handling
- Avoid `.unwrap()` or `.expect()` unless absolutely certain; use `?` operator
- Tauri Commands must return serializable `Result` types
- Structure business logic in `managers/` and `audio_toolkit/`

#### Error Handling

- Always return `Result<T, Error>` from public functions
- Use `?` for error propagation
- Create custom error types with `thiserror` for complex error scenarios
- Log errors appropriately before returning

## Architecture Overview

**Backend (Rust - `src-tauri/src/`):**

- `lib.rs` / `main.rs` - Entry point, Tauri setup, tray, managers
- `managers/` - Audio, Model, Transcription business logic
- `audio_toolkit/` - Device enumeration, VAD using Silero
- `commands/` - Tauri command handlers
- `settings.rs` - App settings management
- `shortcut.rs` - Global keyboard shortcuts

**Frontend (React/TypeScript - `src/`):**

- `App.tsx` - Main component with onboarding
- `components/` - UI components
- `stores/` - Zustand state
- `i18n/` - Internationalization

**Patterns:**

- Manager Pattern: Core functionality in managers initialized at startup
- Command-Event: Frontend calls Tauri commands; backend emits events
- Pipeline: Audio → VAD → Whisper → Text output

### Technology Stack

- Backend: `whisper-rs`, `cpal`, `vad-rs`, `rdev`, `rubato`, `rodio`
- Frontend: React, TypeScript, Tailwind CSS, Zustand, Playwright
- Platform: Metal (macOS), Vulkan (Windows/Linux)

### Application Flow

1. Initialize: Start minimized to tray, load settings
2. Model Setup: Download preferred Whisper model
3. Recording: Global shortcut triggers recording with VAD
4. Processing: Audio → Whisper for transcription
5. Output: Text pasted to active app via clipboard

### Single Instance

App enforces single instance - launching when running brings settings window to front.
