# Commit message template — mf2figma

This repository uses a Conventional Commits style with a few repo-specific guidelines. Use this template when writing commit messages to keep history consistent, enable automated changelogs/versioning, and make reviews easier.

---

# Format

<type>(<scope>): <short subject>

[optional body — motivation, implementation notes]

[optional footers — BREAKING CHANGE, Closes #issue, Refs #issue]

Rules
- Use the imperative mood in the subject (e.g. "add", not "added" or "adds").
- Keep the subject ≤ 50 characters if possible.
- Capitalize the subject only when it makes sense.
- Separate subject from body with a blank line.
- Use the body to explain "why" and any implementation decisions or trade-offs.
- Use footers for metadata: `BREAKING CHANGE: ...`, `Closes #123`, `Refs JIRA-456`.

---

# Allowed types (recommended)
- feat: new feature
- fix: bug fix
- docs: documentation only changes
- perf: performance improvements
- refactor: code change that neither fixes a bug nor adds a feature
- test: adding or updating tests
- build: changes that affect the build system or external dependencies (e.g., webpack, npm)
- ci: changes to CI configuration and scripts
- chore: other changes that don't modify src or tests (e.g., tooling)
- style: formatting, whitespace, etc.
- deps: dependency upgrades (preferred format `chore(deps): ...` or `deps: upgrade ...`)

# Suggested scopes
Use a scope to make it clear which area the change affects. Examples for this repo:
- plugin
- plugin(main)
- plugin(ui)
- shared
- shared(mapping)
- capture-server
- extension
- docs
- tests
- build
- ci
- scripts
- deps

Example: `fix(plugin(ui)): correct message bridge serialization` or `feat(shared): add mapPaints helper`

---

# Checklist (apply before committing)
- [ ] Ran unit tests: `npm test` (or targeted tests)
- [ ] Ran lint: `npm run lint` and fixed issues
- [ ] Built changed artifacts if relevant: `npm run build` / `npm run server:build`
- [ ] Updated docs/README/AGENTS.md entries when behaviour, manifest, networking, or privacy changed
- [ ] If changing data shapes (`CaptureData`, `CapturedNode`, etc.), bump `CURRENT_SCHEMA_VERSION` and add migration notes in `shared/migrations/`
- [ ] If changing plugin manifest output paths or build outputs, update `scripts/build-manifest.js` and `docs/NETWORKING.md` per AGENTS.md rules
- [ ] Add/adjust tests for the changed behavior

Put a checked item in the template before finalizing the commit when applicable.

---

# Breaking changes
- Indicate breaking changes either by appending a `!` to the type/scope (e.g., `feat(plugin)!: ...`) or by adding a footer starting with `BREAKING CHANGE:` followed by the description and migration instructions.
- When a breaking change affects schema shapes (shared schema), remember to bump `CURRENT_SCHEMA_VERSION` and add a migration entry.

Example:

```
feat(shared)!: change CaptureData.node shape to include 'source'

BREAKING CHANGE: CaptureData now requires a 'source' field on nodes. Update capture-server serialize.ts to populate the field. Bump CURRENT_SCHEMA_VERSION from 1 to 2.

Closes #789
```

---

# Examples

feat(plugin(main)): add font substitution UI for missing fonts

fix(capture-server): handle image download timeout and record asset failures

chore(deps): upgrade puppeteer to 20.x

docs: update README with build output guidelines

---

# How to use this template
- To make Git open this template when committing locally (repo-only):

  git config commit.template .github/COMMIT_MESSAGE_TEMPLATE.md

- To set it globally (optional):

  git config --global commit.template ~/path/to/.gitmessage.txt

- If you use PR-based workflows with squash merging, maintainers can clean up/squash commit messages into a clean Conventional Commit when merging; nevertheless, prefer writing a clear message for each commit.

---

# Notes specific to mf2figma (AGENTS.md rules)
- When you modify capture data shapes, mapping contracts in `shared/mapping/`, or other shared types, update schema versioning in `shared/schema/` and record migration notes.
- Manifest / build output alignment: if you change build outputs or manifest paths, update `scripts/build-manifest.js` and document the change in `docs/NETWORKING.md` — mention this in the commit body.
- Privacy/network changes: if a change affects network domains, local capture behavior, or data retention, note it clearly in the commit message and update `docs/PRIVACY.md`.

---

Thank you for keeping commit history clear and useful.
