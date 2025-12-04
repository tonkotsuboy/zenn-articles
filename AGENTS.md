# Repository Guidelines

## Project Structure & Module Organization
- `articles/` Japanese articles; filenames are kebab-case slugs (e.g., `typescript-as-const-satisfies.md`).
- `articles_en/` English counterparts; mirror slugs from `articles/` when available.
- `books/` Zenn book projects; keep each book isolated in its own folder.
- `images/` Assets under `images/<slug>/file.png` and referenced with `/images/<slug>/...` in Markdown.
- `.kiro/` AI-assisted specs (`design.md`, `requirements.md`, `tasks.md`) to consult before drafting.
- Root config: `package.json`, `pnpm-lock.yaml`, `index.html`, Volta-pinned Node (25.2.1).

## Build, Test, and Development Commands
- Install dependencies: `pnpm install` (or `npm install` if pnpm unavailable).
- Live preview: `npm run preview` → starts Zenn preview server (default http://localhost:8000) with auto-reload.
- Optional formatting check: `npx prettier --check "articles/**/*.md" "articles_en/**/*.md"`.
- Format a file when needed: `npx prettier --write <path>`.

## Coding Style & Naming Conventions
- Markdown with Zenn frontmatter (`title`, `emoji`, `type`, `topics`, `published`).
- Language: Japanese by default; use English only in `articles_en/`.
- Headings: sentence case, depth up to `###`; avoid very short sections.
- Code fences: include language tag (e.g., ```ts); keep snippets minimal and runnable.
- Filenames: lowercase kebab-case; align with intended URL slug.

## Testing Guidelines
- No automated test suite; rely on manual preview.
- Before pushing, run `npm run preview`, navigate affected pages, and verify links, images, and code blocks render correctly.
- For image changes, confirm paths stay under `/images/<slug>/` and use optimized formats (PNG/WebP).

## Commit & Pull Request Guidelines
- Commits: short imperative summaries; Japanese descriptions are common; group related edits per article/slug. Examples: `chore: update dependencies`, `記事に画像を追加`.
- PRs: describe purpose, touched slugs/paths, and preview URL or notes. Add screenshots for layout/image updates. Link related Zenn page or issue when applicable.

## Security & Content Hygiene
- Do not commit secrets or unpublished client data; scrub logs and screenshots.
- Use licensed or original images; prefer CC0/own assets.
- Keep dependency bumps intentional and consistent with `pnpm-lock.yaml`; avoid unrelated upgrades in content-only changes.
