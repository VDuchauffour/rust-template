# AGENTS.md

Guidance for OpenCode sessions working in this repository.

## What this repo is

A **Copier template** for scaffolding Rust projects. It is not itself a Rust
application — there is no `Cargo.toml` or `src/` at the root. Do not run
`cargo build`, `cargo test`, or `cargo clippy` at the repo root; they will fail.

All renderable project files live under `template/` as Jinja templates
(suffix `.jinja`). The template is defined by `copier.yml` at the root, which
sets `_subdirectory: template`.

## Two-tier layout — do not conflate

| Location                      | Purpose                                                                            |
| ----------------------------- | ---------------------------------------------------------------------------------- |
| `.github/workflows/` (root)   | CI for **this template repo** (PR title checks, template linting)                  |
| `template/.github/workflows/` | CI that **ships to generated projects** (cargo fmt/clippy/test, coverage, publish) |
| `.gitignore` (root)           | Ignores for the template repo itself                                               |
| `template/.gitignore`         | Ignores that ship to generated projects                                            |

The same split applies to `.pre-commit-config.yaml`, `.yamlfix.toml`,
`.vscode/`, `.devcontainer/` — all live in `template/` and are rendered into
generated projects, not used by the template repo itself.

## Testing template changes locally

```sh
# Render the template into a temp dir. All three flags are mandatory:
#   --with jinja2-time  → copier.yml uses jinja2_time.TimeExtension
#   --vcs-ref=HEAD      → test the working tree, not the latest git tag
#   --trust             → _tasks run shell commands (rm -f ...)
uvx --with jinja2-time copier copy . /tmp/render \
	--defaults --vcs-ref=HEAD --trust

# Then verify the rendered project compiles:
cd /tmp/render && cargo check && cargo +nightly fmt --check && cargo clippy -- -D warnings
```

To test a specific variant (e.g. no license, or with crates.io publish):

```sh
uvx --with jinja2-time copier copy . /tmp/render \
	--defaults --vcs-ref=HEAD --trust \
	--data license=None --data copyright_holder=
```

## Copier `_tasks` (post-render conditional deletion)

`copier.yml` defines two tasks that run after rendering:

- `release=false` → removes `.github/workflows/release.yml`
- `license=='None'` → removes `LICENSE`

These are shell commands, which is why `--trust` is required when copying.

## Toolchain quirks

- **rustfmt requires nightly**: `cargo +nightly fmt`. The `rustfmt.toml` sets
  `group_imports = "StdExternalCrate"`, which is a nightly-only option. The
  CI and `justfile` both pin `+nightly` for formatting.
- **pre-commit requires `uv`**: `just pre-commit-install` runs
  `uvx pre-commit install`. `uv` must be on PATH.
- **No `Cargo.lock` committed** — gitignored (library convention) at both
  root and template levels.

## `just` recipes (in `template/justfile`)

Available in rendered projects (or in the template repo if you copy a
`justfile` out). Key shortcuts:

| Command            | What it does                                                      |
| ------------------ | ----------------------------------------------------------------- |
| `just ci`          | `fmt-check` + `lint-strict` + `test`                              |
| `just fmt`         | `cargo +nightly fmt`                                              |
| `just lint-strict` | `cargo clippy -- -D warnings`                                     |
| `just machete`     | Detects unused dependencies (installs `cargo-machete` if missing) |

## PR conventions

Enforced by `.github/workflows/pr-enhancement.yml` at the repo root:

- PR titles must follow **conventional commits** (e.g. `feat:`, `fix:`, `chore:`).
- Subject line must **not start with an uppercase letter**.
- PRs labeled `dependencies` bypass title validation (for Renovate).
