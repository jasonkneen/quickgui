---
name: quickgui-core-change
description: Review or change QuickGUI framework internals, including Rust rendering and scheduling, the shared host ABI, Go and TypeScript bindings, and CLI packaging. Use to select ownership boundaries, generators, and relevant regression checks.
---

# QuickGUI core changes and review

Find the QuickGUI checkout and read its `AGENTS.md`. Paths and commands below are relative to that checkout. Start with the requested behavior, current diff, and affected consumers; preserve unrelated local work. A review request calls for source-grounded findings and validation, not automatic production fixes.

## Route the work

| Concern | Source and guidance |
| --- | --- |
| Retained identity, layout, input and invalidation | `src/ui_tree/`, `docs/architecture/input.md`, `docs/architecture/rendering.md` |
| Scheduling, restart scopes and lifecycle | `src/runtime/`, `docs/architecture/runtime.md`, `docs/architecture/scheduling.md` |
| GPU resources, text and compositing | `src/renderer/`, `docs/architecture/rendering-optimization-plan.md` |
| Shared host protocol, commands and events | `crates/quickgui-host/src/`, especially `capi.rs`, `queued_events.rs`, `tree.rs`, and `events.rs` |
| Go construction and compiler | `go/ui/`, `go/reactive/`, `go/native/`, `go/host/`, `go/internal/`, `docs/architecture/go-views.md` |
| Bun/Solid bindings and worker lifecycle | `packages/native/src/`, `packages/solid/src/`, `docs/typescript.md` |
| CLI build, resources and distribution | `packages/cli/src/`, `docs/cli.md`, `docs/releasing.md` |
| Optional extension backends | `crates/quickgui-terminal/`, `crates/quickgui-updater/`, `docs/architecture/extensions.md` |

Read `docs/architecture/performance.md` before changes involving scheduling, reactivity, native mutations, layout, rendering, or reuse. Follow its linked rendering/coverage guides for the affected mechanism. Native capabilities live in Rust; Go and TypeScript expose those capabilities while owning their language-specific construction and reactivity.

Trace edits across the ABI and consumers. Preserve clean-window sleep, equivalent-write no-ops, smallest-phase invalidation, stable retained identity, bounded queues/caches, and cleanup at owner disposal. Validate mutation batches before applying them. Keep borrowed callback spans inside the callback or copy them before return; only the notification crosses to the Bun worker. Review shutdown against every pending-request collection, including late replies and resource disposal. For packaging, check writer/reader round trips at encoded and decoded size boundaries as well as path validation.

## Generate from the source of truth

Do not hand-edit generated bindings. Inspect each generator's targets before running it:

```sh
bun scripts/generate-typescript.ts
bun scripts/generate-style-helpers.ts
go -C go generate ./protocol
```

Rust wire constants drive the protocol. Style helpers align Rust, Go and TypeScript. Additional Go component/options generators are checked by `scripts/check-go.sh`; follow their source directives when changing those APIs. Update the relevant examples and docs to the current unreleased surface.

## Select checks

| Changed area | Checks from repository root |
| --- | --- |
| CLI/build/packaging | `bun run test:js`, `bun run typecheck:js` |
| TypeScript runtime/components | `bun run test:typescript`, `bun run typecheck:js`, `bun scripts/generate-typescript.ts --check` |
| Fluent style surface | `bun scripts/generate-style-helpers.ts --check` |
| Go SDK/compiler/integration | `bash scripts/check-go.sh` (includes formatting, generators, SDK and compiled example tests with CGO disabled) |
| Rust core/host | Relevant filtered regressions first; for substantial rendering changes, `cargo test --lib` and `cargo test -p quickgui-host --lib` |
| Rust formatting | `cargo fmt --all -- --check` |
| Real TypeScript host lifecycle | After staging the native library, `bun scripts/check-typescript.ts`; application coverage uses `bun scripts/check-typescript-examples.ts` |
| Documentation/skill-only changes | Skill validation when applicable, links and `git diff --check`; no native rebuild needed |

On macOS prefix the Rust test commands with `scripts/with-macos-ghostty-zig.sh`; that wrapper requires Zig 0.15.2 and handles the Xcode SDK workaround. A missing prerequisite is an environment blocker, not a test failure in the implementation. Use the repository lockfile and installed manifests for toolchain/version claims.

For performance work, test both correctness and avoided work using the existing regressions in `src/ui_tree/tests/layout_invalidation.rs`, `src/runtime/test_context/tests/scopes.rs`, `crates/quickgui-host/src/tests/incremental.rs`, and `src/renderer/tests.rs`. Headless event tests, GPU screenshots, native lifecycle checks and live visual acceptance are different evidence. Rebuild/stage with `bun packages/native/build.ts` before claiming native source changes are running. See `.github/workflows/ci.yml` for broader platform/release checks; publishing and signing remain separate from local validation.

Report reproducible findings with priority, concrete trigger, impact and exact source location. Separate confirmed failures from risks and untested behavior. Record actual checks and blockers without implying a full platform audit from a passing binding suite.
