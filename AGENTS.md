# Repository Guidelines

## Project Structure & Module Organization

This is a multilingual Hugo blog configured in `config.yaml`. Markdown lives in `content/`. Project templates in `layouts/` override the Devise Git submodule in `themes/devise`; keep site changes in `layouts/`, `assets/`, `i18n/`, and `static/`. Treat `public/` and `resources/` as generated output.

## Build, Test, and Development Commands

- `git submodule update --init --recursive` — fetch the Devise theme after cloning.
- `hugo server --disableFastRender --noHTTPCache` — serve the site locally at `http://localhost:1313` with full rebuilds.
- `hugo --minify` — create the production site in `public/`, matching CI.
- `hugo --destination /tmp/vasylchenko-build --cleanDestinationDir --printPathWarnings --printI18nWarnings` — perform a clean diagnostic build without modifying repository output.

Use Hugo Extended 0.165.0, the version configured in `.github/workflows/build.yml`.

## Adding Blog Posts

Create an English draft with `hugo new content post/my-topic.md`. Add matching translations as `my-topic.ua.md`, `my-topic.ru.md`, `my-topic.es.md`, `my-topic.fr.md`, and `my-topic.de.md`; identical base paths let Hugo associate all six pages automatically. Update `title`, `date`, `tags`, and `categories` in front matter. Preview drafts with `hugo server -D`, then set `draft: false` when ready to publish. Put shared images in `static/images/` and reference them as `/images/example.png`. Keep translations structurally aligned and verify links, code blocks, and taxonomy pages in every language.

## Coding Style & Naming Conventions

Use two-space indentation for YAML and SCSS and four spaces for HTML templates. Prefer small, kebab-case partials, for example `layouts/partials/language-switcher.html`. Name posts with lowercase kebab-case paths.

## Testing Guidelines

There is no unit-test suite; require a clean Hugo build. For UI changes, inspect every supported language at desktop and mobile widths. Verify translation destinations, missing-translation behavior, console errors, and horizontal overflow. The theme's `npm test` is only a placeholder.

## Commit & Pull Request Guidelines

Use short imperative subjects such as `Fix warnings`; bilingual content uses prefixes like `[UA][EN] Concurrent Collections in Java (#12)`. Keep commits focused. PRs should explain the change, list affected languages and validation commands, link issues, and include before/after screenshots for visual changes. Ensure GitHub Actions passes.

## Agent Run Log

After each major iteration, append a timestamped entry to `codex_run_log.md` covering the milestone, files changed, commands, failures, fixes, current results, and next action.
