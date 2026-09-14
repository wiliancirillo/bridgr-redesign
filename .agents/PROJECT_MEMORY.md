# Project memory

> Version-controlled source of truth for durable project context. Keep it current and free of secrets.

## Workspace

- Paths here are for the `ecosystem` workspace (`/home/wilian/Projects/ecosystem`), not the earlier personal machine (`/home/alatar/Projetos`), whose paths do not exist here.
- Siblings: `agents/`, `bridgr-pro/`, `bridgr-services/{medlens,nfepack,spook}/`. The described app is `~/Projects/bridgr`, outside the workspace.
- Example folders in `docs/` (`~/Projetos/medlens`, …) are sample data the app displays, not paths of this workspace.

## Project purpose

- Primary goal: the functional brief for redesigning **Bridgr Pro**'s look in Claude Design — every screen described from the code, with the rules a new look must keep ([001](decisions/001-a-functional-brief-read-from-the-code.md)).
- Explicit non-goals: no application code; the exported design files are references, not the app's implementation; it does not replace Bridgr Pro's own `docs/telas.md`, which stays the app's spec.
- Intended users: the operator (uploads `docs/`, sets the design direction), Claude Design (reads `docs/`), and agents keeping the brief in step with the app.

## Source stamp

- Described code: `~/Projects/bridgr` at `8aa7c35` (2026-08-28, "feat: the dashboard, whole (040)"), read on 2026-09-10.
- When a screen changes in the app, its document changes and this stamp moves in the same change-set.

## Architecture

- `docs/` is the canonical upload set, numbered in reading order: brief → product → invariants → platform → shell → components → sample data → `screens/NN-*.md` (sidebar order).
- `design/` contains the reviewed Claude Design exports: one `.dc.html` per canvas, its `uploads/` assets, `support.js`, and the auxiliary `github.md`; it is reference material, not the source of functional truth.
- Existing-screen briefs follow the template described in `docs/00-brief.md`. Documents `docs/screens/19-*.md` through `25-*.md` are proposed flows, not implemented behavior.
- Original export names (`Bridgr Reborn - …`) remain unchanged for import compatibility; the repository name is Bridgr Redesign.
- `docs/06-sample-data-reborn.md` supplements the fictional design data; `docs/inventario-de-instancias.md` records component coverage and known gaps, not completed validation.
- `insights/` is developer-facing and never uploaded: engineering and project findings from reading the code.

## Invariants

- No real patient, person, hospital, order, NF or pedido identifier anywhere; examples and design exports use the cast in `docs/06-sample-data.md`.
- On-screen text is quoted verbatim from Bridgr Pro's `src/lib/i18n/locales/pt-BR.ts`; better wording is marked as a proposal.
- The cross-screen rules a redesign must keep are in `docs/02-invariants.md`; they come from Bridgr Pro's `AGENTS.md` domain rules.
- `docs/` and `design/` must not contain duplicate copies of the same imported content; compare hashes when importing a Claude Design archive.

## Conventions

- Prose in English, screen text in pt-BR, `README.md` in Portuguese.
- `**[functional]**` marks an arrangement the work depends on; everything else in an anatomy is today's presentation.
- Links between documents are relative; each screen file names its source files in its header.
- Prettier (`proseWrap: preserve`) owns Markdown layout; markdownlint-cli2 owns structure, and `.markdownlint-cli2.jsonc` states why each disabled rule is off. `design/` exports and `design/github.md` are never formatted or linted (`.prettierignore`).

## Commands

```text
Install tooling: bun install
Lint Markdown:   bun run lint:md
Format Markdown: bun run format          (check only: bun run format:check)
Policy check:    ~/Projects/ecosystem/agents/check.sh .
```

Verified on 2026-09-12, bash: `bun run lint:md` and `bun run format:check` pass; `check.sh` passes with 0 warnings.

## Repository

- Repository: `alatarithron/bridgr-redesign`; local directory: `~/Projects/ecosystem/bridgr-redesign`.
- Remote: `git@github.com:alatarithron/bridgr-redesign.git`, private — the same visibility as `alatarithron/bridgr`, whose internal workflow and ERP details the brief describes.

## External integrations

| Integration                        | Purpose                              | Contract or documentation                                                | Failure considerations                       |
| ---------------------------------- | ------------------------------------ | ------------------------------------------------------------------------ | -------------------------------------------- |
| Bridgr Pro (`~/Projects/bridgr`)   | The app being described              | Its code; `AGENTS.md`, `.agents/decisions/`, `docs/telas.md` for the why | Its docs are partly stale — the code wins    |
| Claude Design                      | Draws the new look from `docs/`      | Uploaded by hand by the operator                                         | Never receives captures or `insights/`       |
| Claude Design project "Bridgr Pro" | The current look (Nocturne + Breeze) | `https://claude.ai/design/p/4654ab58-8272-4f62-945c-87ea3d6b494a`        | Predates Spook, NFe Pack and several screens |

## Active architectural decisions

- [001 — A functional brief, read from the code, with the look left free](decisions/001-a-functional-brief-read-from-the-code.md)

## Known constraints and pitfalls

- The brief decays as the app changes; the source stamp above is how old it is.
- Bridgr Pro's `.agents/PROJECT_MEMORY.md` has lines of tens of thousands of characters; page through a folded copy rather than reading it whole.

## Maintenance

Update this file in the same change-set that makes it inaccurate. Remove obsolete information instead of keeping an informal changelog; Git already stores the history.
