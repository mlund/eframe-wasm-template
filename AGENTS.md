# Agent instructions

Applies to all coding agents (Claude Code, Codex, ...). `.claude/CLAUDE.md` is a symlink to this file.

<!-- template:start -->
## New project from this template

If `Cargo.toml` still says `name = "eframe_wasm_template"`, the repo was just created
from the template. Before anything else, ask the user for a crate name and title, then run:

```sh
scripts/rename.sh <snake_case_crate_name> "<App Title>"
```

It renames the crate, lib and bin, the repo slug in URLs, and the title in
`src/lib.rs` and `index.html`. It also removes this section and the template notes in
`README.md`, then deletes itself. Replace the demo `src/model/` with the real domain module,
and rewrite `README.md` for the new app.
<!-- template:end -->

Upon startup, load these two skills:

- Deep modules: https://deepwiki.com/mattpocock/skills/4.1.4-designing-deep-modules
- TDD: https://raw.githubusercontent.com/mattpocock/skills/refs/heads/main/skills/engineering/tdd/SKILL.md

## Layout

- `src/lib.rs`: wasm entry (`start`), logging, renderer choice, `provenance()`.
- `src/main.rs`: native entry.
- `src/app.rs`: the egui UI. Keep it thin: it reads and drives the domain module, holds no domain logic.
- `src/model/`: domain logic, testable without a UI. Replace the demo oscillator with the real model.
- `build.rs`: embeds `git describe` as `GIT_REVISION`.

## Design

- Prefer deep modules with a minimal public API (load the skill above).
- Make misuse hard: newtypes, enums, type-state over documented conventions.
- Internal state is the module's job, not the caller's; hide it behind the interface.
- Avoid panics under normal use; return errors. Surface them in the UI or via `log::warn!`.
- Code must build for both native and `wasm32-unknown-unknown`. Gate platform code with
  `#[cfg(target_arch = "wasm32")]`; no threads, filesystem or blocking I/O on wasm.
- I/O must work on macOS, Linux and Windows.

## Features

- `plot` (default): `egui_plot` charts. Code using it must compile without it (`#[cfg(feature = "plot")]`, with a fallback).
- `wgpu` (off): renders with wgpu/WebGPU instead of glow. It also gives GPU **compute**:
  `cc.wgpu_render_state` (`egui_wgpu::RenderState`) holds the `device` and `queue`.
  Use them for compute shaders (WGSL) and paint results with an egui paint callback or texture.
  Keep a CPU path for builds without the feature, and test numerics on the CPU path against the GPU path.
  WebGPU needs a secure context (`https` or `127.0.0.1`) and is still missing in some browsers.

## UI

- Every interactive element gets a tooltip (`.on_hover_text(...)`): say what it does and, where relevant, units or valid range.
- Use unicode for symbols in labels (ω, ζ, Å, ⟨x⟩).
- Show `crate::provenance()` wherever results leave the app (exported files, screenshots, the About area), for reproducibility.
- Log through the `log` macros; messages appear in the in-app Log window (`egui_logger`).

## Rust

- Idiomatic Rust, descriptive names.
- Use the LSP tool / rust-analyzer where possible.

## Docs and comments

- Terse. Explain *why*, not *what*; minimal jargon; unicode for math.
- Doc comments on public items say what a caller needs to know, not how it works inside.
- No comments restating the code.

## Testing

- Test-driven: one failing test, then the code that passes it (load the skill above).
- Test through the public interface. Expected values come from analytical results, literals or the
  spec, never from re-implementing the logic in the test.
- No weak tests. Before adding one, ask "which line of *our* code fails if this regresses?" If the answer
  is a std/egui/rng line or nothing, don't write it. Avoid asserting `is_finite()`, `> 0` or "doesn't panic"
  where a specific value is knowable.
- UI code (`app.rs`) is not unit tested; keep logic out of it so it needs none.

## Workflow

- Ask before committing.
- Keep commit, PR and issue messages brief. Never append co-authorship lines, "Generated with" banners or similar.
- Finish complex tasks with `/code-review`, then `/simplify`.

## Before committing

- `cargo fmt`
- `cargo clippy --tests --no-deps --all-features -- -D warnings`
- `cargo test`
- `cargo check --target wasm32-unknown-unknown`, both with and without `--features wgpu`.
