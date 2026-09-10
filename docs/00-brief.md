# Bridgr Pro — redesign brief

This folder is the brief for giving **Bridgr Pro** a new look. It says what
every screen **does** — read from the application's code, not from its current
drawing — and which rules any new look must keep. **How it looks is yours to
reinvent.**

## The app in one paragraph

Bridgr Pro is the workbench of a _faturista_: a billing professional at a
distributor of surgical materials (OPME) in Brazil. Hospitals e-mail
consumption reports (_informes_); the app triages the attachments, has an AI
service read them, lets the operator confer every bill against the document,
matches the text to the ERP's registries, and hands the finished bill to a
robot that types it into the ERP — one bill at a time, with confirmation.
Around that conveyor it is also the console of the services it depends on, a
read-only window onto the ERP's database, a local mirror of the ERP's
registries, and the operator's desk for a hospital purchasing portal. One
person, one machine, all day, and nearly every screen shows patient data.
Details: [01-product.md](01-product.md).

## What we are asking for

1. **A visual language expressed as tokens.** The app lets the operator switch
   palettes and write custom themes, and every colour in the code goes through
   a CSS custom property — a look that exists only as colours painted in
   mockups cannot be implemented. Deliver the roles in
   [03-platform.md](03-platform.md) § Theming (base, accent ramp, state roles
   with their three faces, desk and paper, code ink, type scale, spacing for two
   densities, radii, elevation), in **dark and light**.
2. **The shell** — titlebar with search, bell and user menu; the sidebar
   expanded and folded into a rail; the status bar; toasts; the locked window.
   [04-shell.md](04-shell.md), [screens/16-profile-and-lock.md](screens/16-profile-and-lock.md).
3. **The shared components, in all their states** —
   [05-components.md](05-components.md). The grid is on most screens; get it
   right once.
4. **Every screen in [screens/](screens/), with its states** — not only the
   happy path. Each document lists the states worth drawing: not asked yet, in
   flight, every kind of empty, partial, refused, service down. Suggested order,
   highest leverage first: [Revisão](screens/03-review.md) (the densest and most
   important screen), [Triagem](screens/02-triage.md),
   [Faturamento](screens/04-billing.md), [Dashboard](screens/01-dashboard.md),
   [OPMENEXO](screens/13-opmenexo.md), [Consulta](screens/06-query.md),
   [Catálogos](screens/08-catalogs.md), [Configurações](screens/10-settings.md),
   then the rest.

## Rules for every mockup

- **Fictitious data only** — the cast in [06-sample-data.md](06-sample-data.md).
  Never a capture of the current app: every screen of it shows real patients.
- **On-screen text in Brazilian Portuguese**, using the labels the documents
  quote. They are the app's real strings (it has an i18n layer); a better
  wording is welcome as a **proposal**, marked as such.
- **Dates `dd/mm/aaaa`** with four-digit years; ERP codes, lots, references and
  SQL in **monospace, verbatim** — never re-cased, never reformatted.
- **Canvas: a desktop window of 1440 × 900**, the default size. Check the
  1024 × 768 minimum for anything dense.

## What is fixed and what is free

| Fixed — it is function                                                                                                   | Free — it is look                                                           |
| ------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------- |
| The rules in [02-invariants.md](02-invariants.md) and each screen's **Must survive the redesign**                         | Palette, typography, iconography, shape, elevation, motion                  |
| Arrangements marked **[functional]** in a screen's anatomy (e.g. the document beside the fields it is conferred against) | The arrangement of everything else; how each component is styled           |
| State semantics: amber = a human must decide here; green = somebody checked; accent = selection; red = destructive/failed | How each state is drawn — as long as it never relies on colour alone        |
| The platform constraints in [03-platform.md](03-platform.md)                                                              | Illustration, the tone of empty states, the feel of density                 |
| The conveyor's order: Triagem → Revisão → Faturamento                                                                     | Navigation grouping and naming beyond that — proposals welcome              |

## How to read a screen document

Every file in `screens/` follows one template:

- **The question** — the one thing the screen exists to answer. If a new
  layout makes that question harder to answer at a glance, it is worse.
- **Anatomy** — region by region, as the code draws it today. **[functional]**
  marks what the work depends on; the rest is today's presentation.
- **Data shown**, **States**, **Actions** — what to draw, in every state, and
  every gesture with its confirmation and feedback.
- **Business rules** — why things are the way they are. Most were paid for by
  a real incident.
- **Must survive the redesign** / **Free to redesign** — the line, per screen.
- **Sample content for mockups** — ready-made fictitious rows.
- **Design opportunities** — proposals for the new design, observed in the
  code. They are suggestions, not decisions.
- **Open questions** — what nobody has decided yet.

## Direction the operator still has to set

The brief does not decide these; the design may propose, the operator decides:

- **Native or its own?** Today the app wears KDE's Breeze palette to look
  native on the desktop it runs on. A new identity can keep that kinship or
  leave it.
- **Dark first?** Dark is the default today, with a light variant and four
  palettes to choose from.
- **Which typeface** — it must be bundled with the app (no network), and
  today's font stack names Inter without bundling it.
- **Which icon set** — Phosphor today, bundled as a web font.
- **How dense** — today a compact scale plus an even more compact mode, and a
  separate density for grid rows.
