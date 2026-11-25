# Contributing to Mission Control

Thanks for helping improve Mission Control. Bug reports, focused feature
proposals, documentation fixes, and code contributions are welcome.

## Before you start

- Search existing issues and pull requests for related work.
- Open an issue before starting a large behavioral or architectural change.
- Never include API keys, gateway tokens, `.env`, SQLite databases, or private
  workspace data in an issue or commit.

## Local development

Install a current stable Rust toolchain, clone the repository, and run:

```bash
cargo build --locked
cargo run --locked
```

Offline Demo mode is the easiest way to exercise the UI without external
credentials or services.

## Required checks

Before opening a pull request, run:

```bash
cargo fmt --check
cargo clippy --all-targets -- -D warnings
cargo test --locked --all-targets
```

If a change affects the TUI, test keyboard navigation in both a narrow and a
wide terminal. Do not use color as the only way to communicate state.

## Pull requests

- Keep each pull request focused on one change.
- Explain the motivation and user-visible behavior.
- Add or update tests for behavioral changes.
- Update the README or configuration examples when applicable.
- Include a screenshot or terminal recording for visible UI changes when useful.

By contributing, you agree that your contribution is licensed under the
project's MIT License.
