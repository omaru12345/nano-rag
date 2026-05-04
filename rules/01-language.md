# 01 — Language Policy

The end goal of this project is acquisition by Google (Android AI Core / Pixel team).
To be discoverable and credible to non-Japanese decision makers (engineers, recruiters,
M&A teams), every public-facing artifact must be in English.

## English required (public-facing)

- `README.md`
- `docs/architecture.md`, `docs/roadmap.md`, and any other doc not listed below
- All source code identifiers and inline comments (Rust, Kotlin, JNI)
- Commit messages, PR titles, PR bodies, issue titles and bodies
- Public release notes, changelogs, Maven Central artifact descriptions, blog posts

## Japanese allowed (internal-only)

- `CLAUDE.md`
- `docs/acquisition-thesis.md` (internal strategy — never to be shown to a buyer)
- `rules/` themselves stay in English (they are part of the public repo)
- Personal notes kept outside the repo

## When in doubt → English

If a file might ever be seen by a Google engineer, an OSS contributor, or a Hacker News
reader, write it in English. Translation later is more expensive than writing in English now.

## Tooling expectations

- `cargo clippy -W clippy::non_ascii_literal` to discourage non-ASCII string literals.
- KDoc / Rustdoc must be English.
- A pre-commit or CI check that flags Japanese characters in `README.md` and `docs/*.md`
  (excluding `acquisition-thesis.md`) is encouraged once the codebase grows.
