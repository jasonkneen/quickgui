---
name: quickgui-app-development
description: Build or change QuickGUI desktop applications in Go, TypeScript with Bun and Solid, or Rust. Use for application components, reactive state, native services, and app build or test workflows.
---

# QuickGUI application development

Locate the QuickGUI source checkout and the target application's directory separately. In this repository read `AGENTS.md`, the application's manifest and configuration, and the relevant language guide before editing. When invoked outside the checkout, locate it from the application's local dependency references or use its installed QuickGUI documentation; do not assume relative paths below belong to the application.

## Choose the language path

- **Go:** read `docs/go.md` and a matching example under `examples/`. UI constructors return nodes. Containers start empty and attach `.Child(...)` or `.Children(...)`; text uses `ui.Text(value)`. Reusable styles use `ui.Style()` and `.Merge(...)`. Compound instances expose `.Root()` and `.Trigger()`. Use the view compiler through the CLI: raw `go build` bypasses automatic reactive-expression compilation. Signals belong to the UI goroutine; use `native.Dispatch` or component-scoped `ui.Async` for worker results.
- **TypeScript:** read `docs/typescript.md`, `examples/counter-typescript/app.tsx`, and a matching component example. Use the locally pinned Solid version and universal compiler. JSX visual declarations belong in camelCase `style`, including fixed presets; DOM classes and direct style attributes are unsupported. Components construct once and each window owns its reactive root. Input text is `event.value`. Keep AppKit/Winit on the process main thread and Solid/application I/O in the worker.
- **Rust:** read `docs/ui.md`, `docs/component-api.md`, and a matching Rust example. Rust applications compile the crate into their executable. Compound parts use `.root()` and `.trigger()`; check current constructors in source before copying an older example.

Resolve concrete component and native-service behavior from the matching page in `docs/README.md` and the installed/source API. Application state and construction belong in the application language; shared native interaction behavior belongs in Rust. This is an unreleased API: update consumers directly rather than introducing compatibility aliases.

## Build and validate

Run repository commands from its root and pass the actual app directory to `--project`:

```sh
bun install --frozen-lockfile
bun packages/cli/src/cli.ts check --project examples/counter
bun packages/cli/src/cli.ts test --project examples/counter
bun packages/cli/src/cli.ts dev --project examples/counter
```

Substitute `examples/counter-typescript` or the requested application as appropriate. Install dependencies when absent, not on every edit. For Go/TypeScript app-only edits, reuse the matching shared library. If absent or the Rust host changed, build and stage it with `bun run build:native` before launching. Optional terminal/updater extensions have separate native artifacts; consult `docs/architecture/extensions.md` when needed. Do not add CGO or a separate host process.

Choose checks for the changed application rather than automatically running every repository suite. For SDK changes, use the check matrix in the core-change skill or `docs/architecture/performance.md`. Headless tests and compilation do not establish native interaction or visual correctness. For UI acceptance, run the actual app and inspect the affected behavior; report the library/build used and any untested platform. Respect an explicit user request to perform their own build or visual test.
