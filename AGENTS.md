# Repository Guidelines

## Project Structure & Module Organization

This repository is a Chinese-language notes site built with Zola and the Bear theme.

- `content/_index.md` defines the homepage; `content/blog/` contains articles and the blog section's `_index.md`.
- `config.toml` controls site metadata, Chinese translations, menus, feeds, taxonomies, and syntax highlighting.
- `templates/` contains local Tera template overrides. `custom_head.html` customizes fonts and dark-mode colors.
- `themes/bear/` is the upstream Git submodule. Prefer local template overrides over editing theme files.
- `.github/workflows/deploy.yml` builds and deploys GitHub Pages on pushes to `main`.
- `public/` is generated output; do not commit it. No dedicated test directory or asset directory currently exists.

## Build, Test, and Development Commands

Use Zola **0.23.6**, matching CI. No Node.js toolchain is required.

- `git submodule update --init --recursive`: initialize the Bear theme after cloning.
- `zola serve`: start the local preview with automatic rebuilds.
- `zola build`: generate the site in `public/` and validate content and templates.
- `zola check`: check the site and links; external checks require network access.
- `git diff --check`: detect whitespace errors before committing.

## Coding Style & Naming Conventions

Write articles in concise Chinese. Use lowercase, hyphen-separated filenames such as `ssh-socks-proxy-on-windows.md`. Preserve existing URL aliases when renaming published articles.

Use TOML front matter delimited by `+++`, with `title`, `date`, and relevant `taxonomies.tags`. Follow existing two-space indentation in HTML, CSS, and nested configuration examples. Label fenced code blocks with supported languages; use `text` for generic configuration snippets. No formatter or linter is configured.

## Testing Guidelines

There is no automated unit-test framework or coverage threshold. Run `zola build` for content, configuration, and template changes. Use `zola check` when changing links. Preview layout changes at narrow and wide widths and in both system color modes. For metadata changes, inspect generated feeds and sitemap URLs, especially when changing `base_url`.

## Commit & Pull Request Guidelines

History includes concise imperative subjects and prefixes such as `docs:` and `fix:`. Follow that style, for example `docs: clarify SSH proxy configuration`. Keep commits focused and exclude generated files.

In pull requests, describe the change, list validation performed, and link related issues when applicable. Include screenshots for visible layout or color changes. Mention any theme-submodule or deployment changes explicitly.
