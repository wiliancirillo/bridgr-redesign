# Architectural decision: a functional brief, read from the code, with the look left free

- Status: accepted
- Date: 2026-09-10
- Supersedes: none
- Superseded by: none

## Context

The operator wants a new look for Bridgr Pro (`~/Projects/bridgr`) and will ask
Claude Design to draw it. Claude Design never sees the code; it sees what is
uploaded to its project.

Bridgr Pro already has a screen spec (`docs/telas.md`) and a style manual, both
written for the **current** design. They are the wrong input for a new one, for
three reasons:

- `telas.md` is partly stale — sections stamped weeks apart, some in English and
  some in Portuguese, and statements the code has since overtaken (e.g. the
  Revisão's Aprovar "drawn disabled", the README's "the conveyor does not move
  yet").
- It mixes the spec with the history of how each screen got there, which a
  designer cannot tell apart from what is true now.
- The style manual describes the look being replaced; handing it over anchors
  the new design to the old one.

## Decision

1. **A separate repository holds the brief**: `alatarithron/bridgr-redesign`,
   documents only, following the agents structure (`AGENTS.md`, `.agents/`).
2. **The code is the source**: every screen document is read from the code of
   Bridgr Pro at a stamped commit, names its source files, and quotes the
   on-screen text verbatim from the pt-BR locale. Bridgr's docs and decisions
   are used for the _why_, never as the description of behaviour.
3. **Function is fixed, the look is free**: each screen document separates what
   the work depends on (**[functional]** arrangements, business rules, "Must
   survive the redesign") from today's presentation ("Free to redesign"), and
   the cross-screen rules live in `docs/02-invariants.md`.
4. **Two audiences, two folders**: `docs/` is what the operator uploads to
   Claude Design, numbered in reading order; `insights/` holds engineering and
   project findings for the developer and is not uploaded.
5. **Prose in English, screen text in pt-BR**, as in the rest of the
   ecosystem's technical artifacts; `README.md` in Portuguese for the operator.
6. **Fictitious data by construction**: one cast (`docs/06-sample-data.md`)
   used by every example, and no capture of the current app anywhere.

## Alternatives considered

### Update Bridgr Pro's `docs/telas.md` and upload it

- Advantages: one spec instead of two; no new repository.
- Disadvantages: the file carries history and the current look's grammar; its
  stale parts would be drawn as truth; rewriting it in place changes a document
  the app's own agents depend on, in the middle of a design exploration.
- Reason not chosen: the operator asked for a new directory, and the input for
  a new look must be free of the old one.

### Upload the current design canvas and captures, and ask for a "restyle"

- Advantages: fastest; the design tool sees everything at once.
- Disadvantages: anchors the new look to the old one; captures of the current
  app carry real patient data; the canvas predates several screens (Spook,
  NFe Pack, the Avisos screen's growth).
- Reason not chosen: it asks for a repaint, not a new look, and it leaks.

### One single document

- Advantages: one upload.
- Disadvantages: the design is iterated screen by screen; a monolith makes each
  prompt drag every other screen along, and makes maintenance a merge.
- Reason not chosen: per-screen files can be uploaded all at once anyway.

## Consequences

### Positive

- Claude Design can draw every state of every screen without seeing the code.
- The "Must survive" lists keep the expensive, silent domain rules out of the
  reach of a purely visual change.
- The developer gets a list of findings from reading the whole code base.

### Negative or trade-offs

- A second description of the screens exists and **decays** as the app
  changes; the commit stamp in the memory is what tells how old it is.
- English prose around Portuguese labels asks Claude Design to keep the screen
  text in Portuguese; `docs/00-brief.md` says so explicitly.

## Verification

- Every file in `docs/screens/` names its source files in the bridgr repo, and
  its quoted labels exist in `src/lib/i18n/locales/pt-BR.ts`.
- No real person, hospital, order or document number appears in `docs/`.
- `~/Projects/ecosystem/agents/check.sh .` passes.

## References

- `docs/00-brief.md` — the ask, the rules for mockups, and the direction still open.
- Bridgr Pro `AGENTS.md` — the domain rules the "Must survive" sections protect.
- Bridgr Pro `.agents/decisions/006-departing-from-the-design.md` — the current
  design's departures, context for what the old deck no longer matches.
