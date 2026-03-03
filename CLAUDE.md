# CLAUDE.md

## Project

mdserve is a markdown preview server built as a companion for AI coding agents.
See the [README](README.md) for project overview and the
[architecture doc](docs/architecture.md) for design details.

## Build and test

All commands must be run via `nix develop` to get the correct Rust
toolchain (the system Cargo may not support Cargo.lock v4):

```bash
nix develop -c cargo build --release
nix develop -c cargo test             # all tests
nix develop -c cargo test --test integration_test  # integration tests only
```

Rust 1.82+, 2021 edition. Templates are embedded at compile time via
minijinja-embed (changes to `templates/` require a rebuild).

## Testing in the browser

Use the Playwright MCP tools (`browser_navigate`, `browser_snapshot`,
`browser_take_screenshot`, etc.) to verify UI changes. A good test
directory with many nested markdown files:

```bash
nix develop -c cargo run --release -- ~/src/github.com/citrusleaf/par-eng-team
```

Then navigate to `http://127.0.0.1:3000/` with Playwright.

## Project structure

- `src/main.rs` - CLI parsing and entry point
- `src/app.rs` - Axum router, handlers, state management, file watcher
- `src/lib.rs` - Markdown rendering
- `templates/` - MiniJinja templates (Jinja2 syntax), embedded at compile time
- `tests/integration_test.rs` - Integration tests using axum-test

## Design constraints

- **Agent-companion scope.** mdserve renders markdown that AI agents produce
  during coding sessions. Features that push it toward a documentation platform,
  configurable server, or deployment target are out of scope.
- **Zero config.** `mdserve file.md` must work with no flags or config files.
- **Recursive scanning.** Directory mode recursively scans and watches
  subdirectories. Gitignored files/directories are skipped; `.git/` is always
  skipped. Non-gitignored dot-prefixed directories (e.g. `.claude/`) are included.
- **Pre-rendered in memory.** All tracked files are rendered to HTML on startup
  and on change. Serving is always from memory.
- **Minimal client-side JS.** Most logic is server-side. Client JS handles
  theme selection and WebSocket reload only.

## Changelog

Generated with [git-cliff](https://git-cliff.org/) using `cliff.toml`. To
update `CHANGELOG.md`:

```bash
git cliff -o CHANGELOG.md
```

## Commits

Use conventional commits: `type: lowercase description` (e.g. `feat:`, `fix:`,
`chore:`, `docs:`, `refactor:`, `test:`). No scopes, no emojis. Subject line
max 72 chars, imperative mood. Body optional, wrap at 72 chars, explain why not
what.
