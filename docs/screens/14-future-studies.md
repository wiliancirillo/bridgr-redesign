# Inpart Saúde · Duplicidades · Costantini — `/inpart` · `/duplicates` · `/costantini`

> Where: Sidebar › Ideias futuras › "Inpart Saúde", "Duplicidades", "Costantini" — each wearing the "estudo" badge · Status: empty study · Code: `src/routes/inpart/+page.svelte`, `src/routes/duplicates/+page.svelte`, `src/routes/costantini/+page.svelte`, `src/lib/components/EmptyScreen.svelte`, `src/lib/nav.ts`, `src/lib/i18n/locales/pt-BR.ts` (`nav`, `question`, `screen`)

## The question

Each room states the question its study would answer, verbatim:

| Route         | Label (sidebar and title) | Question on screen                                          |
| ------------- | ------------------------- | ----------------------------------------------------------- |
| `/inpart`     | Inpart Saúde              | "Estudo: o que a Inpart Saúde precisa daqui."               |
| `/duplicates` | Duplicidades              | "Estudo: procurar duplicidade para trás, não só no portão." |
| `/costantini` | Costantini                | "Estudo: o que o Costantini precisa daqui."                 |

## Where it sits in the work

Nowhere yet. These are **explorations, not features**: work on them is well in
the future, and a study may never be built — the badge says so on the sidebar
item, so nobody has to click into an empty room to find out. Two former studies
became real screens and lost the badge when they did (Opmenexo,
NFePackMonitor), which is why the kicker "Ideias futuras" now mixes built
screens and studies ([../04-shell.md](../04-shell.md)).

## Anatomy

- **Sidebar item** (group "Ideias futuras", in this order: NFePackMonitor,
  **Inpart Saúde**, Opmenexo, **Duplicidades**, **Costantini**): icon
  (`ph-first-aid-kit`, `ph-copy-simple`, `ph-buildings`), label, and the
  quiet **"estudo"** badge at the right (mono, quiet fill). In the folded rail
  only the icon remains — the badge is clipped away. The items are never
  disabled.
- **The room** (the shared empty screen, [../05-components.md](../05-components.md)):
  one column centred vertically and horizontally in the screen area —
  1. the label as the page title (23 px, weight 500);
  2. the question (muted ink, at most ~52 characters per line);
  3. a quiet line: **"Esta tela ainda não foi construída."** (11 px).

  Nothing else: no data, no button, no illustration.

- The shell is complete around it: the item lit in the sidebar with its badge
  still showing, the titlebar context "· Duplicidades" (etc.), the window title
  "Bridgr Pro · Duplicidades".

## Data shown

None. The room shows only fixed text.

## States

One. The only variation is the shell's: in the folded rail the "estudo" badge
is not visible.

## Actions

None inside the room; only the shell's gestures.

## Business rules

1. **The badge says "exploration" on the item itself** — before the click, not
   only once the person is inside an empty room (`nav.ts`).
2. **An empty room states its question, not just its name** — a blank route
   tells nobody where they are or what will be there; the question is the
   shortest honest answer to both, and the one a built screen would keep
   answering (`EmptyScreen.svelte`).
3. **No invented content.** No numbers, no sample rows, no buttons that cannot
   do anything: an invented number in a room that does not exist would become an
   expectation of something finished ([../02-invariants.md](../02-invariants.md) § 2).
4. **A study that becomes a screen loses the badge** — kept, it would say "may
   never be built" about something that answers (the Opmenexo and
   NFePackMonitor precedents).
5. **Duplicidades is not the Dashboard's duplicate card.** The Dashboard draws a
   raw comparison — the same patient name and the same date of use in two or more
   different files — which blocks and discards nothing, says so, and offers a
   door to `/duplicates` as the study, honestly labelled unbuilt
   ([01-dashboard.md](01-dashboard.md)). The global search draws no
   "Duplicidades" group, because the study does not exist
   ([18-search.md](18-search.md)).

## What each study is about

**From the code**: only the three questions above. **From the project memory**:
"well in the future", "may never be built". **From the original design (design
only — the app draws none of this)**, each study had an "intention page": a
kicker "ideia · nada construído", where the need comes from, a title, a short
account of the problem, two columns — "O que ela responderia" (three
questions) and "De que ela depende" (three dependencies, each marked ready or
not) — and a closing next step under the note "Tela de intenção, de propósito
sem dado e sem botão: um número inventado aqui viraria expectativa de coisa
pronta." Its content, summarized:

- **Inpart Saúde — "a second dock".** Part of the orders arrive through a
  portal instead of e-mail, and become work only when somebody remembers to
  download and re-send them. It would answer: which portal order still has no
  usage report; where the authorization got stuck, and since when; what was
  already downloaded. Depends on: the dock accepting any piece and keeping its
  provenance (ready); reading the portal with a credential, with the same care
  as the robot's read-only access (not ready); a duplicate rule so one order
  cannot enter twice by two paths (not ready). Next step: measure how many
  orders per week arrive by portal rather than e-mail.
- **Duplicidades — "the same bill entering twice".** The same admission
  arrives by e-mail, by portal, sometimes re-sent by hand; today the operator
  notices mid-conferência, and what slips through becomes a double invoice found
  later, at cancellation. It would answer: has this piece entered before by
  another path; which bills in flight point to the same admission; what was
  invoiced twice last month. Depends on: the dock keeping each piece's
  provenance and arrival time (ready); an identity key for a bill — admission +
  patient + date — which does not exist (not ready); deciding the gesture on a
  match — block the entry, or mark both and let the operator choose (not
  ready). Next step: compare admission and date across last week's pieces and
  count the real collisions.
- **Costantini — "a client with its own handling".** Its orders, material
  codes and informe format differ from every other client's, and today that
  lives as an exception in the head of whoever confers. It would answer: what of
  theirs is stuck and at which stage; which of their codes matches ours, and who
  confirmed it; whether the informe left in the format they accept. Depends on:
  Triagem separating pieces by client (ready); rules per client — today the
  adapters learn for everyone (not ready); an export model with their layout
  (not ready). Next step: start with the output format.

Nothing else is known: no decision record, no data model, no route beyond the
empty room.

## Must survive the redesign

- The study nature is visible **on the navigation item**, before the click.
- The room states its question and says, in words, that it is not built.
- No invented data, controls or progress in a study room.
- The items stay reachable (never disabled or hidden).

## Free to redesign

Everything else: the room's composition, an illustration, the badge's form,
whether studies keep their own group — and whether the rooms adopt the original
design's intention page (below).

## Sample content for mockups

Use the three rows of **The question** verbatim, with "Esta tela ainda não foi
construída." under each. The rooms hold no patient or client data.

## Design opportunities

1. **Adopt the intention page.** The original design's data-free page (where
   the need comes from, the three questions, the dependencies with ready marks,
   the next step) says far more than today's three lines, without inventing a
   number. It needs its texts moved into the app's language files — and a rule
   for who keeps each "ready" mark true, since a stale "ready" would assert what
   nobody verified.
2. **Explain the Duplicidades door.** The Dashboard's duplicate card sends
   people here; the room could say how its future answer differs from that raw
   comparison, so the click does not read as a dead end.
3. **Keep "estudo" visible in the rail** (a small non-amber marker) and
   **separate studies from built screens** in the sidebar
   ([../04-shell.md](../04-shell.md) § Design opportunities).

## Open questions

- Which of the three studies will ever be built, and in what order?
- If the intention page is adopted, who updates its dependency marks as the app
  changes?
