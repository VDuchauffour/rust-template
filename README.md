# Rust Copier Template

A [Copier](https://copier.readthedocs.io) template for scaffolding Rust projects with batteries included: CI/CD, dev container, pre-commit hooks, automated dependency updates, and a `just` task runner.

## Usage

```sh
copier copy gh:VDuchauffour/rust-template path/to/destination
```

> [!WARNING]
> This template relies on the [`jinja2-time`](https://pypi.org/project/jinja2-time/) extension (declared via `_jinja_extensions` in `copier.yml`). Copier will fail to render with an error about `jinja2_time.TimeExtension` if it is not installed. Install it beforehand, either as an ephemeral `uvx` run or a persistent install via `uv` / `pipx`:
>
> ```sh
> # Option 1 — inject into an ephemeral uvx run
> uvx --with jinja2-time copier copy gh:VDuchauffour/rust-template path/to/destination
>
> # Option 2 — install globally once with uv
> uv tool install copier --with jinja2-time
>
> # Option 3 — install globally once with pipx
> pipx install copier
> pipx inject copier jinja2-time
> ```

Answer the prompts (project name, license, GitHub owner, etc.) and Copier renders a ready-to-develop project into `path/to/destination/`.

## What you get

- **Cargo** project (`edition 2024`, release profile tuned for LTO + strip)
- **CI/CD** via GitHub Actions — format check, clippy (`-D warnings`), tests, coverage (tarpaulin → Codecov), release drafter, conventional PR titles
- **Dev container** with Rust, Node, Python, and a post-create script
- **Pre-commit** hooks — trailing whitespace, yamlfix, taplo, mdformat, prettier
- **Renovate** config for automated dependency updates
- **Just** recipes for common tasks (`just ci`, `just fmt`, `just lint-strict`, ...)
- Optional **crates.io publish** workflow (toggle via `publish_to_crate`) and **binary build** workflow for multiple targets (toggle via `build_binaries`)

## Template options

| Prompt                | Type   | Default           | Notes                                             |
| --------------------- | ------ | ----------------- | ------------------------------------------------- |
| `project_name`        | str    | `my-rust-project` | Cargo package name                                |
| `project_description` | str    | `A Rust project`  |                                                   |
| `github_owner`        | str    | `myusername`      |                                                   |
| `github_repo`         | str    | `= project_name`  |                                                   |
| `author_name`         | str    | _(empty)_         |                                                   |
| `author_email`        | str    | _(empty)_         |                                                   |
| `license`             | choice | `MIT`             | MIT, Apache-2.0, GPL-3.0, Unlicense, None         |
| `copyright_holder`    | str    | `= author_name`   | Asked only for MIT / GPL-3.0                      |
| `publish_to_crate`    | bool   | `false`           | Adds a workflow to publish the crate to crates.io |
| `build_binaries`      | bool   | `false`           | Adds a workflow to build and upload binaries      |

## Repository layout

```text
.
├── copier.yml                 # template definition (prompts, excludes, tasks)
├── template/                  # everything that gets rendered into a project
│   ├── Cargo.toml.jinja
│   ├── LICENSE.jinja
│   ├── README.md.jinja
│   ├── justfile
│   ├── rustfmt.toml
│   ├── src/main.rs
│   ├── .github/               # workflows, renovate, release-drafter, ...
│   ├── .devcontainer/
│   ├── .vscode/
│   └── ...
└── .github/workflows/
    └── pr-enhancement.yml     # CI for this template repo (PR title/label rules)
```
