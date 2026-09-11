# Revisão — `/review` (antechamber) and `/review/[id]` (the bill screen)

> Where: sidebar Esteira › Revisão (both routes keep that item lit) · Status: built · Code: `src/routes/review/+page.svelte`, `src/routes/review/[id]/+page.svelte`, `src/lib/review.svelte.ts`, `src/lib/spotlight.ts`, `src/lib/components/DatePicker.svelte`, `src/lib/viewer.svelte.ts` + `src/lib/components/ViewerZoom.svelte`; Rust `src-tauri/src/review.rs`, `resolution.rs`, `queue.rs`, `cheatsheet.rs`, and the commands `list_extractions` … `approve_bill` in `src-tauri/src/lib.rs`

Two screens, one station. The **antechamber** (`/review`) answers _which file
still needs me_ and gets out of the way. The **bill screen** (`/review/[id]`)
is the densest and most important screen in the app: the operator confers
MedLens's reading against the document, bill by bill, and **Aprovar** is the
conveyor's exit door toward the Fantasma.

All on-screen text below is quoted verbatim from
`src/lib/i18n/locales/pt-BR.ts`; `{n}` marks what the screen fills in. Where
the code prints no singular form (`1 arquivos`), the quote shows that.
Pointers like "decision 018 § 1" name records in the bridgr repository's
`.agents/decisions/`; "[013]" and "[016]" are the predecessor app's records,
quoted in bridgr's `AGENTS.md`.

| Term (screen word)             | Meaning                                                                                                                                                       |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| arquivo / file                 | one extraction: one document MedLens read (a PDF, an e-mail body, a photo, a crop)                                                                            |
| conta / bill                   | one patient + one atendimento inside a file; an _informe_ carries several                                                                                     |
| a conferir                     | a bill still here: not marked "não enviar", not in the queue, not worked by the robot                                                                         |
| departed bill                  | a bill whose queue file waits in the Pasta-fila, or whose Faturamento record says finished/failed — it lives on [Faturamento](04-billing.md), not here         |
| não enviar                     | this app's own reversible "do not send" mark on a bill                                                                                                        |
| atendimento / prontuário       | the admission number / the medical-record number (its fallback)                                                                                               |
| convênio, médico               | insurer and doctor; MedLens reads them as TEXT, this screen matches the text to an ERP registry code                                                           |
| catálogo / mirror              | the local copy of an ERP registry, collected in [Catálogos](08-catalogs.md)                                                                                   |
| colinha                        | per-client defaults the robot needs (deposit, order header), assembled in Configurações → Adapters ([Configurações](10-settings.md))                           |
| portão / gate                  | the completeness check that must be clear before a bill can be approved                                                                                       |

Shared pieces used here: `EmptyState` (antechamber), `DatePicker` (the date
field) — see [components](../05-components.md); the photo viewer and its zoom
row are the same ones [Triagem](02-triage.md) uses; toasts are the shell's
([shell](../04-shell.md)). **Not** used here: the `Grid` (the items table is a
plain table), `ConfirmButton` (Aprovar's confirmation is hand-built), and
`RegistryCodeField` (the Colinha dialog's code field — a second "pick a
registry code" pattern that lives in Configurações).

---

## Part A — The antechamber — `/review`

> Where: sidebar Esteira › Revisão; also the target of the bell's "extraction finished" notice and of the global search's patient rows · Status: built · Code: `src/routes/review/+page.svelte`, `src/lib/review.svelte.ts`, `list_extractions` (`src-tauri/src/lib.rs`)

### The question

"Qual arquivo abrir?" — which extracted file still has bills that need me.

### Where it sits in the work

- **Upstream**: [Triagem](02-triage.md) sends a piece to MedLens. When the
  extraction finishes, the bell's notice points **here** (the antechamber,
  not the file) — see [Avisos](17-notices.md).
- **Downstream**: a row opens the bill screen (Part B). Bills approved there
  leave for [Faturamento](04-billing.md); this row keeps only a quiet count of
  them.
- **Other doors**: the global search's patient rows land here; its bill rows
  land directly on `/review/{id}` ([Busca](18-search.md)). Per docs/telas.md
  the Dashboard's "A conferir" funnel counts the same population
  ([Dashboard](01-dashboard.md)); not verified in code for this file.
- **Needs**: MedLens running (Configurações → MedLens).

### Anatomy

1. **Header bar** (full width, divider under it).
   - Left: title "Revisão de contas"; under it the subtitle
     "{N} arquivos · {M} contas pendentes" (absent until the list first
     answers).
   - Right, in this order: the **work filter** dropdown button (current
     label + caret), the **client** dropdown button ("Todos os clientes" or
     the chosen client's label + caret), and **Reindexar** (circular-arrows
     icon + "Reindexar"; tooltip "Faz o MedLens reler a pasta vigiada — custa
     uma varredura inteira; é gesto seu, nunca automático.").
   - Dropdown panels float under their button, right-aligned, at least 180 px
     wide, at most 260 px tall (scroll); the active item carries a check.
2. **Service strip** (conditional, amber, full width): "A Revisão lê o
   MedLens — ligue em Configurações → MedLens."
3. **Failure strip** (conditional, red, full width): the last failure's
   message.
4. **The list**: one row per file, vertical, rows capped at 1000 px wide;
   **[functional]** the scroll container spans the full width so the
   scrollbar sits at the window's edge (operator's call, 09/08/2026).

**Row anatomy** — the whole row is one button; hover tints it with the accent.

| Part       | Content                                                                                                                                  |
| ---------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| Tile       | 30 px rounded square with an icon: a PDF icon when the file name ends in `.pdf`; a **table** icon for everything else (e-mail, photo, crop) |
| Name       | the file name, monospace, one line, ellipsized                                                                                            |
| Client pill | the adapter's label                                                                                                                     |
| Warn pill  | "extração parcial" (amber) — only when MedLens's reading status is `partial_success`                                                    |
| Meta line  | "{n} conta" / "{n} contas" · "{n} item" / "{n} itens", then " · {n} itens sem lote" in amber when that count is above zero               |
| Chip       | "{n} a conferir" (accent tint) — only when above zero                                                                                    |
| Quiet chip | "{n} no Faturamento" (neutral grey, no arrow, not separately clickable) — only when above zero                                           |
| Dot        | 7 px: amber while the file has bills a conferir, green otherwise                                                                         |
| Caret      | right-pointing caret                                                                                                                     |

### Data shown

| Field                     | Meaning                                                                                                                         | Presentation rule                                                                                                  | Example                             |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ | ----------------------------------- |
| File name                 | the name MedLens holds for the extraction                                                                                       | monospace, ellipsized                                                                                              | `informe_vale_verde_2026-09-02.pdf` |
| Client                    | the extraction's adapter                                                                                                        | the adapter's label from MedLens; the raw adapter id when that list is not in memory                               | Hospital Vale Verde (raw `vale_verde`) |
| Reading status            | MedLens: `success`, `partial_success`, `needs_review` (errored extractions are never listed)                                    | only `partial_success` draws a pill; `needs_review` draws nothing                                                  | extração parcial                    |
| Bills                     | every bill in the file, whatever its fate                                                                                       | "{n} conta(s)"                                                                                                     | 5 contas                            |
| Items                     | MedLens's item count for the file                                                                                               | "{n} item/itens"                                                                                                   | 10 itens                            |
| Items without lot         | MedLens's count of items with no lot                                                                                            | amber, only when above zero, always plural. unverified: whether MedLens excludes struck items from it              | 1 itens sem lote                    |
| "{n} a conferir"          | bills of this file not marked não enviar, not in the queue, not worked                                                          | accent chip                                                                                                        | 3 a conferir                        |
| "{n} no Faturamento"      | bills of this file that departed: queue file waiting, or a Faturamento record saying finished or failed                         | quiet chip — a location, never a verdict                                                                           | 1 no Faturamento                    |
| Dot                       | does this file still need me                                                                                                    | amber if "a conferir" > 0; green otherwise (all departed, all discarded, or a mix)                                 | —                                   |
| "{N} arquivos"            | every file listed, **ignoring both filters**                                                                                    | hidden while not asked yet; no singular                                                                            | 4 arquivos                          |
| "{M} contas pendentes"    | across **all** files (filters ignored), bills not discarded, not queued, not worked                                             | same                                                                                                               | 5 contas pendentes                  |

**The work filter** (default "A enviar" on every visit; not remembered):

| Label               | A file is listed when…                   |
| ------------------- | ---------------------------------------- |
| "A enviar"          | it has at least one bill a conferir      |
| "No Faturamento"    | it has at least one departed bill        |
| "Não enviar"        | it has at least one bill marked não enviar |
| "Todos os arquivos" | always                                   |

The **client filter** combines with it (both must hold). Its menu: "Todos os
clientes" first (never checked, even when active), then one entry per adapter
present in the whole list, sorted by adapter id, labelled by adapter label.
Rows come in MedLens's order; there is no sort on this side.

### States

| State                               | Trigger                                              | What is drawn                                                                                                                                                                   |
| ----------------------------------- | ---------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Not asked yet                       | the list never answered in this session              | header without subtitle; list area **blank** — no rows, no empty state                                                                                                          |
| MedLens state unknown               | the app has not learned MedLens's state yet          | as above, no strip, Reindexar disabled, nothing requested                                                                                                                       |
| MedLens stopped or starting         | state known, not running                             | amber strip "A Revisão lê o MedLens — ligue em Configurações → MedLens."; Reindexar disabled; no request. The list area keeps the last list of this session (not marked stale), or stays blank |
| MedLens foreign                     | a MedLens this app did not start is answering        | treated as up                                                                                                                                                                   |
| MedLens comes up while here         | state turns running                                  | the list is requested on its own                                                                                                                                                |
| Rows                                | the list answered                                    | rows as above                                                                                                                                                                   |
| Nothing passes the filters          | answered, filtered set empty                         | `EmptyState` (files icon): "Nada para conferir" + "O que a Triagem extrair aparece aqui, arquivo por arquivo, com o documento ao lado." — the **same** sentence for every filter |
| List failed                         | the request failed                                   | red strip with the message (for example `MedLens: HTTP 401`, `MedLens: no credential held`); earlier rows stay; with no earlier rows the empty state above appears under the strip |
| Reindexing                          | Reindexar clicked                                    | button disabled, its icon spinning; rows stay until the new list lands; no success toast                                                                                        |
| Menu open                           | a dropdown clicked                                   | the panel; it closes on a choice or a click elsewhere (Esc does not close it)                                                                                                   |
| Exactly one file                    | —                                                    | nothing special: docs/telas.md's "gets out of the way when there is a single file" is **not built**                                                                              |
| New extraction finishes while here  | —                                                    | nothing: the list reloads only on a visit, when MedLens comes up, or on Reindexar                                                                                               |
| Narrow window                       | —                                                    | the header row has no wrap rule and simply compresses; the list stays one column                                                                                               |

### Actions

| Gesture              | Where            | Preconditions / disabled reason                                                   | Effect                                                                                                          | Confirmation | Reversible | Feedback                          | Shortcut |
| -------------------- | ---------------- | --------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- | ------------ | ---------- | --------------------------------- | -------- |
| Open a file          | click a row      | none                                                                              | goes to `/review/{id}`                                                                                          | no           | —          | navigation                        | —        |
| Pick a work filter   | work dropdown    | none                                                                              | filters rows; resets to "A enviar" on the next visit                                                            | no           | yes        | label changes, check moves        | —        |
| Pick a client        | client dropdown  | none                                                                              | filters rows; resets to all on the next visit                                                                   | no           | yes        | label changes                     | —        |
| Reindexar            | header, right    | disabled while MedLens is not running/foreign, and while already reindexing (no tooltip for either) | MedLens re-hashes every watched file, then the list is re-read (up to a minute)                                  | no           | —          | spinning icon; failure in the red strip | —        |

### Business rules

1. **Only what crossed this app's dock is listed.** MedLens is shared with the
   predecessor app during the transition and holds every extraction ever made
   on the machine; the filter is this app's own piece→extraction record
   (decision 018 § 6).
   Errored extractions never appear — the failed piece says so in Triagem.
2. **The bill is a token** — it walks forward and back, never duplicates. "A
   conferir" means _still here_; a queued or worked bill lives on the
   Faturamento, and this row only points at it (023 § 6).
3. **The filter speaks work, not extraction status, and starts on "A
   enviar"**: the question is "which file still needs me"; a file whose every
   bill is queued or discarded is done here (006 § 2, operator's call
   16/08/2026). MedLens's reading status stays on the row's pill.
4. **Reindexar is the expensive, manual gesture** — MedLens hashes every
   watched file; it never rides an automatic reload
   (`review.svelte.ts`, `medlens.rs`).
5. **A long command shows a sign of life before it is invoked** (AGENTS.md,
   "The machine"): the spinning icon is set before the call.
6. **Not asked yet asserts nothing**: before the first answer the list draws
   neither rows nor "Nada para conferir".
7. **"No Faturamento" is location, never state** — the Faturamento says how
   each departed bill is doing, so the chip is deliberately quiet (never green
   or red).

### Must survive the redesign

- Default filter "A enviar", and the four work filters in the faturista's
  vocabulary (not MedLens's statuses).
- The two counts per file with exactly the meanings above; the departed count
  stays quiet.
- A blank, assertion-free state before the list answers.
- Reindexar visibly marked as the expensive gesture, with its sign of life,
  disabled while MedLens is not up.
- The MedLens strip naming where to fix it.
- The scrollbar at the window's edge.

### Free to redesign

Row styling, tile icons, chip and dot shapes, dropdowns vs segmented filters,
header composition, empty-state art, the 1000 px row cap (the deck's number).

### Sample content for mockups

Header: "Revisão de contas" · "4 arquivos · 5 contas pendentes"; filter
"Todos os arquivos", client "Todos os clientes".

| Tile  | Name                                   | Client                           | Pills             | Meta                                  | Chips                              | Dot   |
| ----- | -------------------------------------- | -------------------------------- | ----------------- | ------------------------------------- | ---------------------------------- | ----- |
| PDF   | `informe_vale_verde_2026-09-02.pdf`    | Hospital Vale Verde              | —                 | 5 contas · 10 itens · 1 itens sem lote | 3 a conferir · 1 no Faturamento   | amber |
| table | `etiquetas_sala3.jpg`                  | Instituto Ortopédico Horizonte   | —                 | 1 conta · 2 itens                     | 1 a conferir                       | amber |
| table | `mensagem_santa_aurora_2026-09-04.eml` (e-mail file-name shape unverified) | Hospital Santa Aurora | extração parcial | 2 contas · 4 itens            | 1 a conferir · 1 no Faturamento    | amber |
| PDF   | `informe_sol_nascente_2026-09-01.pdf`  | Hospital Infantil Sol Nascente   | —                 | 1 conta · 3 itens                     | 1 no Faturamento                   | green |

Under "A enviar" the last row disappears.

### Design opportunities

1. **One empty sentence per filter.** Today "Nada para conferir" + the Triagem
   note shows for "Não enviar" and "No Faturamento" too, where it is wrong
   ("nothing marked não enviar" is a different fact).
2. **A cheap "atualizar" beside Reindexar.** The only in-place reload is the
   expensive re-hash; a file extracted while the operator sits here appears
   only after leaving and coming back. (The data path exists: the plain list
   call.)
3. **An icon per document kind** (PDF, e-mail body, photo, crop). Today every
   non-PDF wears a table icon, which reads as "spreadsheet".
4. **Singular forms** ("1 arquivo", "1 conta pendente", "1 item sem lote").
5. **The single-file shortcut** docs/telas.md asks for — see Open questions.
6. **A stale marker** when MedLens is down and the list shown is the session's
   last one.
7. **Keyboard on the menus** (Esc, arrows) and a check on "Todos os clientes"
   when active, like "Todos os arquivos" has.
8. The quiet "no Faturamento" chip could open the Faturamento filtered to this
   file — only if the operator wants a second door; today the whole row opens
   the bill screen.

### Open questions

- docs/telas.md says the antechamber "gets out of the way when there is a
  single file". Unbuilt: should it auto-open, or only make the single row
  prominent? An auto-open would bypass the filters' answer.
- Should rows be sorted (arrival, client)? Today MedLens's order is used, and
  the summaries carry no arrival stamp (unverified: what order MedLens
  answers in).

---

## Part B — The bill screen — `/review/[id]`

> Where: reached from an antechamber row or a global-search bill row, never from a menu; the sidebar keeps Revisão lit · Status: built (Aprovar live since 14/08/2026) · Code: `src/routes/review/[id]/+page.svelte`, `src/lib/review.svelte.ts`, `src/lib/spotlight.ts`, `src/lib/components/DatePicker.svelte`, `src/lib/viewer.svelte.ts`, `src/lib/components/ViewerZoom.svelte`; Rust `resolution.rs` (matching), `review.rs` (não enviar), `queue.rs` (gate + queue file), `cheatsheet.rs` (colinha), `approve_bill` / `set_bill_discarded` / `extraction_source` / `extraction_original` in `lib.rs`

### The question

"The document is beside it; are the extracted fields correct?" (docs/telas.md)
— and, per bill, "Falta para esta conta ir": what is still missing before this
bill can go to the robot.

### Where it sits in the work

- **In**: an antechamber row; a bill row of the global search
  ([Busca](18-search.md)). Every visit re-reads the extraction (the last copy
  shows at once while it re-reads).
- **Out**: "Voltar" → the antechamber. **Aprovar** writes the bill's queue
  file into the Fantasma's Pasta-fila, creates its [Faturamento](04-billing.md)
  record and tries to hand it to the Fantasma (that hand-off may fail; the
  Faturamento's "Entregar ao Fantasma" is the fallback). The bill then
  **leaves this screen**. The Faturamento's "Retirar da fila" walks it back,
  and it reappears here under "A conferir". Processing (the robot writing to
  the ERP) is a separate gesture on the Faturamento, never here.
- **Depends on**: MedLens (the extraction, corrections, a fallback source);
  this app's spool and links (the source document and the originals behind a
  crop); the insurers and doctors mirrors from [Catálogos](08-catalogs.md); the
  matching memory; the colinha (Configurações → Adapters) and the Pasta-fila
  (Configurações → Fantasma) — see [Configurações](10-settings.md).

### Anatomy

**Overall layout [functional]** — header bar; optional red failure strip; then
a body split in two columns: the **work column** (bills) on the left, the
**document pane** on the right, with a draggable divider. The work column
starts at 50% and can be dragged between **40% and 50%** of the body width, so
the document always gets at least half (operator's rule, 09/08/2026). The
position is not remembered.

1. **Header bar**: outlined "Voltar" button with a left arrow; title "Revisão de
   contas"; subtitle "{adapter label} · {file name} · {N} conta(s)" (N = every
   bill in the file). The subtitle is absent until the extraction loads.
2. **Failure strip** (conditional, red): the store's last failure — the same
   strip as the antechamber's.
3. **Work column**
   - **Tools bar** (wraps on narrow widths): uppercase kicker "Exportar
     lista"; "Copiar tudo" (copy icon); "CSV" (download icon, always disabled,
     tooltip "O CSV chega com a fila — o desenho já o traz desabilitado.");
     spacer; the pointer "{n} no Faturamento" + right arrow (quiet pill, only
     when n > 0); the segmented control "A conferir ({n})" · "Não enviar ({n})"
     · "Todas ({n})" (active segment: accent ring and accent text).
   - **Bills list** (scrolls): one collapsed row per bill in view, in
     MedLens's order, and **at most one open card**. **Nothing is open on
     arrival** (operator's call, 17/08/2026); clicking a row opens it and
     collapses the previous one. The open card has no close gesture: another
     row's click is the only way to change it.
4. **Divider**: 5 px grab area drawn as a 1 px line, column-resize cursor,
   accent while hovered or dragged.
5. **Document pane**
   - **Viewer strip** (left to right): file chip (icon + name); the zoom row
     (photos only); spacer; a right-aligned column with the gesture on top and
     its meaning under it **[functional]** (operator's call, 12/08/2026).
   - **Stage**: the document, filling the rest on the desk background.

**Collapsed bill row** (one button): caret-right; patient name (heading font)
or "—"; a pill with the **insurer text as read** (only if one was read); the
pill "não enviar" when discarded; meta line "{dd/mm/aaaa} · {atendimento or
prontuário or —} · {n} item/itens" (items counted including struck ones); a
7 px dot at the right — amber while the gate holds, green when it is clear,
neutral grey for a discarded bill.

**Open card** (accent border), top to bottom:

1. **Card head** (accent-tinted band): caret-down (accent), patient, insurer
   pill, meta "{dd/mm/aaaa} · {atendimento} · {n} itens" (always plural here),
   dot amber/green (discarded bills get no pill and no grey dot here).
2. **Fields grid** — three columns, label above value. Row 1: "Paciente",
   "Data da cirurgia" (with a calendar icon button right of the value),
   "Atendimento". Row 2: "Convênio", "Médico". Labels are small uppercase;
   the two matched labels carry " · {text as read}" in a quieter tone and in
   the text's own case. **[functional]** the read text stays visible beside
   whatever code it was matched to.
3. **Items table** — columns "Ref." (12%, monospace) · "Material" (41%) ·
   "Qtde" (7%, right-aligned, monospace) · "Lote" (15%, monospace) · "ANVISA"
   (18%, monospace, muted, read-only) · an action column (7%). One line per
   cell, ellipsized, no tooltip.
4. **Add-item row** — at rest: ghost button "+ Acrescentar item" and the hint
   "clique na célula para editar · Enter grava · Esc desfaz".
5. **Gate box** (only when something is missing and the bill is not
   discarded; amber-tinted panel) — kicker "Falta para esta conta ir", then one
   line per demand, each with a filled amber warning icon and amber text.
   **[functional]** it lives on the bill, right above its actions.
6. **Action row** — "Aprovar" (primary: accent outline and text) or, after
   its click, the inline confirmation in its place **[functional]**; "Copiar"
   (copy icon); "Não enviar" (danger: red outline and text, × icon) or, on a
   discarded bill, "Voltar a conferir" (counter-clockwise arrow).

**Matched field (Convênio, Médico)** — the value area takes one of these
shapes:

| Situation                                  | Drawn                                                                                                                                                                                                                       |
| ------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| No text read                               | quiet "—", inert                                                                                                                                                                                                            |
| Matched (automatic or remembered)          | registry code (monospace, accent) + registry name (muted, ellipsized) + green check-circle whose tooltip tells which: "Casou sozinho com o catálogo — releitura a cada abertura, nada foi gravado." / "Casamento seu, lembrado — o app não pergunta de novo." + a pencil button (tooltip "Refazer o casamento — escolher outro cadastro") |
| Not found                                  | the field **is** a search input (placeholder "buscar no catálogo…") on an amber underline, with an amber warning icon at its right end                                                                                      |
| Found two (homonyms)                       | as "not found", plus an amber note under it: "Mais de um cadastro atende por este nome. Diga qual antes de aprovar — acertar por sorte é pior que perguntar."                                                              |
| Redo in progress (pencil clicked)          | the search input on the amber underline, **without** the warning icon — a redo is a choice, not an alert                                                                                                                  |
| List open                                  | a floating list under the input (≥ 260 px wide, ≤ 260 px tall, scroll): up to 30 rows "code  name", filtered as the operator types; if the mirror is not in memory or empty, instead the note "Este catálogo ainda não foi coletado — colete em Catálogos para ter contra o que casar. O campo segue livre."; if the typed text matches nothing, an **empty box** (no sentence) |

**Plain field (Paciente, Data da cirurgia, Atendimento)** — the value sits on a
dashed underline (the "click to edit" cue); empty is "—". While editing it is an
input on an accent underline. The calendar icon beside the date stays visible
while editing.

**Items table cells** — hover on an editable cell draws a faint 1 px accent
ring; editing draws a 2 px accent ring on a tinted background with the caret
(hover and editing must not look alike — operator's call, 09/08/2026). An
empty "Lote" shows "sem lote" in amber; other empty cells show a quiet "—". A
**struck** row keeps its place, every cell but the action at 45% opacity with
line-through, cells no longer editable; its action icon is the
counter-clockwise arrow ("Desriscar item") instead of the × ("Riscar item",
red on hover).

**Add-item row, expanded** — inputs with placeholders "Material" (grows),
"Ref.", "Qtde" (narrow, prefilled "1"), "Lote"; then "Gravar item" (primary;
disabled until Material has text, and while a save is in flight) and
"Cancelar".

**Viewer strip, in detail**

- File chip: a crop icon when the crop is on screen, otherwise a PDF icon for
  every kind (HTML, photo, PDF alike); the name is the original's file name
  when "Original" is on screen, the extraction's file name otherwise.
- Zoom row (only when a photo is on screen — never over a PDF or an e-mail
  body): "−" (tooltip "Afastar"), the level "{n}%" as a button (tooltip
  "Enquadrar (voltar ao início)"), "+" ("Aproximar"), and the turn button
  ("Girar 90°").
- Right column, only for a reading born of a **crop**: a pager "‹ {i}/{n} ›"
  (aria "Original anterior" / "Próximo original"; only while "Original" is
  on screen and there is more than one original), then a two-segment toggle
  "Recorte" · "Original". Under it, always, the note:
  - not a crop: "Documento de origem" (quiet grey);
  - crop, "Recorte" on screen: "Recorte enviado — é isto que o MedLens leu"
    (accent);
  - crop, "Original" on screen: "O documento como chegou — não é o que o
    MedLens leu" (accent).

**Stage, by document kind**

| Kind                      | Drawn                                                                                                                                                                                                                                               |
| ------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| PDF                       | the native embedded PDF viewer filling the pane, with its own toolbar, zoom and scroll; nothing can be drawn over it                                                                                                                               |
| E-mail body (HTML)        | a fully sandboxed frame filling the pane on a paper background; no script ever runs in it; the spotlight below                                                                                                                                     |
| Photo, or the crop image  | a centred framed image with a shadow, fitted on arrival; wheel zooms (×1.12 per notch), "−"/"+" by ×1.25, range 5%–800% where 100% is one photo pixel per screen pixel; drag pans only when zoomed past fit (grab / grabbing cursor); rotation in 90° steps; the level button returns to fit and zero rotation; every document swap resets it |

**The spotlight** (e-mail bodies only):

- Waits for a **click**: it lights only after the operator opened a bill by
  clicking its row in this file — never on arrival (operator's call,
  16/08/2026).
- Finds the open bill's atendimento (or prontuário), trimmed, as the **whole
  text of a table cell** (never a substring — one number must not claim a
  longer one's row).
- Draws a dark veil (about 62% near-black) over the whole document and lifts
  the matching rows above it, cell by cell, on white with a 2 px blue outline;
  the document scrolls so the first lit row sits about 38% down the pane
  (smoothly where the engine honours it). Links under the veil keep working.
- No atendimento, no match, a PDF or a photo: the plain document, no veil.
- Its colours are fixed values, not theme tokens: they are written into the
  foreign document, which cannot read the app's theme.

**The calendar** (`DatePicker`, see [components](../05-components.md)): the
icon button (tooltip "Abrir o calendário") opens a 252 px popover under it,
flipped above when there is no room and kept inside the window; month title
from the locale, capitalised ("Setembro de 2026"); previous/next month arrows
(aria "Mês anterior" / "Mês seguinte"); weekday initials Sunday first (the
locale's short names); a 6 × 7 grid with the neighbouring months dimmed and
the current value highlighted; no "today" marker. It closes on a pick, on Esc,
on a click outside, and on any scroll or window resize.

### Data shown

**Header fields**

| Label              | Source                                                   | Presentation                                                                    | Edit                                                                  | Example                                          |
| ------------------ | -------------------------------------------------------- | ------------------------------------------------------------------------------- | --------------------------------------------------------------------- | ------------------------------------------------ |
| "Paciente"         | patient name as read                                     | as read; "—" when empty                                                         | click → text                                                          | MARTA LEMOS FERREIRA                             |
| "Data da cirurgia" | use date, ISO on the wire                                | `dd/mm/aaaa`; a value that is not a plain ISO date shows raw                    | click → type `dd/mm/aaaa`, or the calendar                            | 01/09/2026                                       |
| "Atendimento"      | admission number, **or the prontuário when it is absent** | as read; nothing says which of the two is shown                                 | click → text; always saved as the admission number                   | 2026004517                                       |
| "Convênio"         | insurer text as read + its match                         | label " · {text}", value per the matched-field table                            | the text is **not** editable; only the match                           | "Vida Plena Saúde" → 000412 VIDA PLENA SAÚDE     |
| "Médico"           | doctor text as read + its match                          | same                                                                            | same                                                                  | "DR. RAFAEL NOGUEIRA" → 004512 DR. RAFAEL NOGUEIRA |

MedLens also reads a hospital and a procedure per bill, and an expiry date per
item; this screen does not show them.

**Matching outcomes** (per read text, per catalog):

| Outcome        | How it arises                                                                                             | Stored?                                     | Gate                         |
| -------------- | --------------------------------------------------------------------------------------------------------- | ------------------------------------------- | ---------------------------- |
| Remembered     | the operator picked an entry for this text before (any bill, any file)                                     | yes, keyed by the **normalized** read text  | clear                        |
| Automatic      | exactly one mirror entry whose name or alias equals the text (accents, case, spacing folded; never "contains") | no — re-derived on every opening          | clear                        |
| Found two      | two or more different entries equal the text                                                               | no                                          | holds                        |
| Not found      | no equal entry, or the catalog was never collected                                                         | no                                          | holds                        |

The memory wins over the catalog. One pick answers **every** bill whose text
normalizes the same, in every file — "Amparo  saúde" and "AMPARO SAUDE" are one
memory.

**Items**

| Column    | Source                     | Presentation                                   | Example (sample)                              |
| --------- | -------------------------- | ---------------------------------------------- | --------------------------------------------- |
| "Ref."    | supplier reference         | monospace; "—" when empty; editable            | PC35-24T                                      |
| "Material" | product description       | ellipsized; editable                           | PARAFUSO CORTICAL 3,5 MM X 24 MM TITÂNIO      |
| "Qtde"    | quantity, a whole number   | right-aligned monospace; editable              | 4                                             |
| "Lote"    | lot as read                | monospace; "sem lote" (amber) when empty; editable | 451288 (the ERP stores it as 0000451288 — matching is the robot's) |
| "ANVISA"  | ANVISA registration        | monospace, muted, read-only                    | 10000000001                                   |
| state     | struck / added / as read   | struck = faded + line-through; an added item looks like any other | —                              |

**Counters on this screen**

| Counter                        | Counts exactly                                                                                                   |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------- |
| Subtitle "{N} contas"          | every bill in the file                                                                                           |
| "A conferir ({n})"             | bills not marked não enviar and not departed                                                                     |
| "Não enviar ({n})"             | bills marked não enviar                                                                                          |
| "Todas ({n})"                  | **every** bill in the file, departed included — but the "Todas" view lists only non-departed bills, so the number can exceed the rows |
| "{n} no Faturamento →"         | departed bills: queue file waiting in the Pasta-fila, or a Faturamento record saying finished or failed          |
| Row / card "{n} itens"         | every item of the bill, struck included                                                                          |
| Gate "Lote — {n} …: linhas …"  | non-struck items with an empty lot; line numbers are 1-based table positions, struck rows included in the numbering |

Views: "A conferir" = not discarded, not departed; "Não enviar" = discarded,
not departed; "Todas" = not departed. A departed bill is never drawn here.

**The gate, exact lines, in this order** (the Rust re-check at approval demands
the same things):

1. "Paciente — o cabeçalho veio sem nome" — patient empty.
2. "Data da cirurgia — o cabeçalho veio sem data" — date empty.
3. "Atendimento — sem número de atendimento nem prontuário" — both empty.
4. Only when an insurer text was read: found two → "Convênio → código — mais
   de um cadastro atende; escolha um"; otherwise unmatched → "Convênio →
   código — case o texto lido com o cadastro".
5. Only when a doctor text was read: the same pair with "Médico → código — …".
6. The file's colinha (one answer for every bill of the file; the line is
   absent until the colinhas were read): no colinha → "Colinha do adapter —
   este cliente não tem colinha: monte em Configurações → Adapters";
   incomplete → "Colinha do adapter — falta {fields} (Configurações →
   Adapters)", with the missing ones among "Depósito", "Comissionado", "Tipo
   Pedido", "Tabela", "Depósito do pedido", "Cond. de Venda", "Local de Uso",
   comma-separated. "Cliente" is never required.
7. "Lote — {n} item(ns) não riscado(s) sem lote: linhas {1, 3}".

**The Rust refusal's wording** (toast after "A checagem onde o arquivo é
escrito recusou — ", parts joined by " · "): the same sentences for patient,
date and atendimento; "Convênio → código — case o texto lido com o cadastro"
and "Médico → código — case o texto lido com o cadastro" (also when the cause
is homonyms); the colinha sentences, one per missing field; and "Lote — {n}"
(the count, without the line numbers).

**Clipboard format** ("Copiar" = this bill; "Copiar tudo" = every bill of the
file, departed and discarded included): tab-separated lines, no column
headings. First line: patient · insurer text · date **in ISO** · atendimento
(or prontuário) · doctor text. Then one line per non-struck item: reference ·
material · quantity · lot · ANVISA.

### States

| State                                   | Trigger                                                  | What is drawn                                                                                                                                                                                    |
| --------------------------------------- | -------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Opening                                 | first read of this file in the session                   | bills column: centred quiet "Abrindo a extração…"; no subtitle; document pane: empty stage, strip with no name and "Documento de origem"                                                        |
| Revisit                                 | read before in this session                              | the previous copy at once, replaced when the re-read lands                                                                                                                                       |
| Open failed                             | MedLens down or refusing                                 | red strip (for example `MedLens: HTTP 404`, `MedLens: no credential held`); "Abrindo a extração…" **stays**; the document pane stays empty (the source is never asked without the file name)     |
| Nothing in this view                    | the chosen segment has no bill                           | centred quiet "Nada nesta vista."                                                                                                                                                                |
| Every bill departed                     | —                                                        | "A conferir (0)", "Nada nesta vista.", and the "{n} no Faturamento →" pointer                                                                                                                    |
| Matches not answered yet (or failed)    | just opened / matching call failed (silently)            | both matched fields look "not found", and the gate lists them — the safe direction                                                                                                              |
| Catalog never collected                 | no mirror                                                | fields look "not found"; the list shows the "ainda não foi coletado" note; the gate holds                                                                                                         |
| Colinhas not read yet (or read failed)  | —                                                        | the gate says nothing about the colinha — which can leave the dot **green** and Aprovar **enabled**; the Rust re-check still refuses                                                             |
| Correction in flight                    | after Enter/blur/pick                                    | no visible mark on the field or cell; the old value shows until the re-read lands ("Gravar item" is the only control that disables)                                                              |
| Correction saved                        | —                                                        | green toast "Correção gravada no MedLens." (header fields, cells, calendar); no toast for strike/restore                                                                                         |
| Correction failed                       | MedLens refused                                          | red strip with the message (for example `MedLens: HTTP 422`, unverified code); the field shows the stored value again — what was typed is gone                                                   |
| Second edit while one saves             | Enter on another cell before the first save returns      | the second edit is **dropped with no sign**                                                                                                                                                     |
| Impossible date                         | "31/02/2026" typed                                       | red toast "Data impossível — use dd/mm/aaaa."; the input stays open with the typo                                                                                                               |
| Discarded bill open                     | in "Não enviar" or "Todas"                               | no gate box; Aprovar disabled (no tooltip); "Voltar a conferir" instead of "Não enviar"; fields and cells still editable                                                                          |
| Aprovar blocked                         | the gate holds                                           | Aprovar at 45% opacity, tooltip "O portão segura — veja “Falta para esta conta ir”."                                                                                                              |
| Confirming                              | Aprovar clicked                                          | in its place: "Enviar esta conta para a fila do Fantasma?" + "Enviar" + "Cancelar"; the confirmation survives collapsing and reopening that bill                                                 |
| Approving                               | "Enviar" clicked                                         | that bill's button reads "…"; every Aprovar on the screen is disabled until the answer                                                                                                          |
| Approved                                | the file was written                                     | green toast "Conta enfileirada — ela agora vive no Faturamento."; the bill vanishes from every view; the pointer count grows; nothing else opens; the e-mail spotlight keeps lighting the departed bill |
| Refused by the re-check                 | the Rust gate found something                            | red toast "A checagem onde o arquivo é escrito recusou — {reasons}"; the bill stays                                                                                                             |
| Approval error                          | a hard failure                                           | red toast with the message, e.g. `doca: a Pasta-fila não está configurada — Configurações → Fantasma`, `doca: esta conta está marcada "não enviar"`, `MedLens: HTTP 500`; the bill stays          |
| Não enviar                              | clicked                                                  | the bill moves to "Não enviar" (vanishes from "A conferir") with **no toast** — unless a queue file was actually withdrawn: amber toast "“Não enviar” também retirou a conta da fila."             |
| Source loading                          | —                                                        | stage: quiet "Carregando o documento de origem…"                                                                                                                                                |
| Source failed                           | not in the spool and MedLens has no copy, or MedLens down | stage: red inline box "Não consegui mostrar o documento: {reason}" + "Tentar de novo" (circular arrows) inside it, e.g. `Não consegui mostrar o documento: MedLens: HTTP 404`                    |
| Crop reading                            | the file was composed from marked regions                | toggle + accent note; **"Original" is the default** (operator's call, 14/08/2026)                                                                                                               |
| Original failed                         | the original left the disk, or a watched file changed    | same red inline box + retry, e.g. `doca: o documento original saiu do disco`, `doca: o arquivo na pasta vigiada mudou desde a leitura — este original não existe mais`, `doca: nenhum original registrado para esta leitura` |
| Several originals                       | the crop crossed several photos                          | pager "‹ 1/2 ›" (buttons disabled at the ends and while loading)                                                                                                                                 |
| Adapter list not in memory              | reached without passing the antechamber or Triagem in this session (only those two load it) | subtitle shows the raw adapter id                                                                                                                                                                |
| Narrow window                           | —                                                        | tools bar wraps; the work column clips its own overflow (nothing may bleed over the divider); the fields grid stays three columns; table cells ellipsize; the calendar stays inside the window    |

### Actions

| Gesture                  | Where                         | Preconditions / disabled reason                                                                                         | Effect                                                                                                                                         | Confirmation                                           | Reversible                                                              | Feedback                                                                 | Shortcut                     |
| ------------------------ | ----------------------------- | ----------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------ | ----------------------------------------------------------------------- | ------------------------------------------------------------------------ | ---------------------------- |
| Voltar                   | header                        | —                                                                                                                       | to the antechamber                                                                                                                             | —                                                      | —                                                                       | —                                                                        | —                            |
| Open a bill              | collapsed row                 | —                                                                                                                       | opens it, collapses the other, arms the spotlight                                                                                              | —                                                      | open another                                                            | —                                                                        | —                            |
| Switch view              | segmented control             | —                                                                                                                       | filters the bills; resets on the next visit                                                                                                    | —                                                      | yes                                                                     | —                                                                        | —                            |
| Edit Paciente / Atendimento | field                      | —                                                                                                                       | trimmed; unchanged = nothing; else a correction to MedLens and a re-read                                                                       | —                                                      | edit again                                                              | toast / red strip                                                        | Enter or blur saves, Esc cancels |
| Edit the date by typing  | date field                    | `dd/mm/aaaa`, a real date (an emptied field is sent as an empty correction; unverified how MedLens stores it)           | converted to ISO, then as above                                                                                                                | —                                                      | edit again                                                              | toast; impossible date → red toast, input kept                          | Enter / blur / Esc           |
| Pick a date              | calendar icon                 | —                                                                                                                       | the picked day is saved at once (an open typing edit is dropped)                                                                               | —                                                      | pick again                                                              | toast                                                                    | Esc closes                   |
| Match a convênio/médico  | the search field              | a text was read; the mirror in memory                                                                                   | the pick is remembered for this read text (every bill, every file) and the fields re-match                                                     | —                                                      | pencil + pick another (replaces)                                        | the field turns into code + name + check; a failure goes to the red strip | Esc closes and blurs         |
| Redo a match             | pencil on a matched field     | —                                                                                                                       | reopens the search with the cursor in it                                                                                                       | —                                                      | Esc or a click outside keeps the standing match                          | —                                                                        | —                            |
| Edit a cell              | Ref. / Material / Qtde / Lote | row not struck                                                                                                          | correction + re-read                                                                                                                           | —                                                      | edit again                                                              | toast / red strip                                                        | Enter / blur / Esc           |
| Strike / restore an item | × / ↺ at the row's end        | —                                                                                                                       | a mark, never a deletion: the row stays, struck                                                                                                | —                                                      | yes                                                                     | none on success; red strip on failure                                    | —                            |
| Add an item              | "+ Acrescentar item" → "Gravar item" | Material filled; no save in flight                                                                                | new item with whatever of Ref./Qtde/Lote was filled, appended                                                                                  | —                                                      | strike it                                                               | toast "Item acrescentado."                                               | none (Enter does nothing)    |
| Cancel adding            | "Cancelar"                    | —                                                                                                                       | closes the row; the typed draft is kept for the next "Acrescentar item" until the screen is left                                               | —                                                      | —                                                                       | —                                                                        | —                            |
| Aprovar                  | action row                    | gate clear, bill not discarded, no approval in flight (tooltip only for the gate)                                       | Rust re-reads MedLens, re-matches, re-reads the colinha, re-checks the gate; writes the queue file; creates the Faturamento record; tries the hand-off | **inline**: "Enviar esta conta para a fila do Fantasma?" → "Enviar" / "Cancelar" | only from the Faturamento ("Retirar da fila"), until the robot closes the conferência | "…" → toast (queued / refused / error); the bill leaves                  | —                            |
| Copiar                   | action row                    | —                                                                                                                       | this bill to the clipboard                                                                                                                     | —                                                      | —                                                                       | toast "Conta copiada."                                                   | —                            |
| Copiar tudo              | tools bar                     | —                                                                                                                       | every bill of the file to the clipboard                                                                                                        | —                                                      | —                                                                       | the label reads "Copiado" for about 1.6 s (no toast)                     | —                            |
| Não enviar               | action row                    | —                                                                                                                       | marks the bill (this app's record), then withdraws its queue file if one is there                                                              | none                                                   | "Voltar a conferir"                                                     | amber toast only when a file was withdrawn                               | —                            |
| Voltar a conferir        | action row of a discarded bill | —                                                                                                                      | removes the mark; the bill returns to "A conferir"                                                                                             | none                                                   | "Não enviar"                                                            | none                                                                     | —                            |
| "{n} no Faturamento →"   | tools bar                     | n > 0                                                                                                                   | opens the Faturamento (unfiltered)                                                                                                             | —                                                      | —                                                                       | —                                                                        | —                            |
| Resize the split         | divider                       | —                                                                                                                       | work column between 40% and 50%                                                                                                                | —                                                      | —                                                                       | —                                                                        | —                            |
| Recorte / Original       | strip toggle (crop only)      | —                                                                                                                       | swaps what the stage shows; originals load once per file                                                                                       | —                                                      | yes                                                                     | note under the toggle changes                                            | —                            |
| Page originals           | pager                         | more than one original                                                                                                  | previous / next original                                                                                                                       | —                                                      | yes                                                                     | "{i}/{n}"                                                                | —                            |
| Zoom, fit, rotate, pan   | zoom row / wheel / drag       | a photo on screen; pan only past fit                                                                                    | view only — nothing sent changes                                                                                                               | —                                                      | yes                                                                     | level %                                                                  | wheel                        |
| Tentar de novo           | inside the failure box        | —                                                                                                                       | asks for the document once more                                                                                                               | —                                                      | —                                                                       | loading line                                                             | —                            |

No gesture acts on several bills at once, and there is no keyboard shortcut
beyond Enter/Esc in editors, Esc in the search list and the calendar, and the
wheel in the viewer.

### Business rules

1. **MedLens keeps the truth; this side never holds a diverging draft.** Every
   committed edit — header field, cell, strike, added item — is a correction
   sent to MedLens followed by a re-read; what the screen shows is what MedLens
   would answer anyone (decision 018 § 1).
2. **The list never shrinks.** A struck item stays, marked; deleting would
   silently re-aim every later item at another product (018 § 1).
3. **Bill by bill, never file by file** — one informe carries several bills,
   and every gesture here acts on one.
4. **Text → code matching is remembered, and has four answers.** The human's
   memory wins over the catalog; from the catalog only an **exact** (folded)
   name or alias counts, never "contains"; an automatic match is not stored — a
   machine's guess must not wear a human's decision (018 § 2,
   `resolution.rs`). The memory is keyed by the normalized read text, so one
   pick answers every bill that reads the same.
5. ⚠️ **Two registries answering one name is a question, not a coin toss** —
   nothing is resolved, the field is flagged, the gate holds (AGENTS.md [013]).
6. **The insurer and the doctor need a code whenever the report brought a
   text**; the doctor is not demanded when none was read — inventing a doctor
   is worse than not having one (`queue.rs`).
7. ⚠️ **The lot blocks; the reference does not.** The ERP resolves the item
   from the lot; the reference we send is human conferência only (AGENTS.md
   [013]).
8. **The colinha is the file's adapter's**, seven required fields, `client`
   deliberately not among them: absent means "do not touch the field", and
   only one client uses it (decision 021; `queue.rs`).
9. ⚠️ **Whoever has not asked cannot answer "no"**: the gate draws no colinha
   line before the colinhas were read — a read error must not be drawn as
   "sem colinha".
10. ⚠️ **Nothing enters the queue unfinished, and the gate is re-checked where
    the file is written**, from MedLens's own answer, the matching memory and
    the colinha on disk — never trusting the screen; a refusal crosses as data
    and is worded with the screen's own sentences (decision 022 § 2).
11. ⚠️ **One bill, one click, with confirmation** — no gesture approves several
    (AGENTS.md [016]).
12. ⚠️ **A long command shows its sign of life before it is invoked** —
    Aprovar's "…" is set before the call.
13. **The bill is a token**: approved, it leaves the Revisão (no tab, no pill);
    queued, working, finished and failed all live on the Faturamento; only a
    quiet pointer stays (023 § 6).
14. **"Não enviar" is ours and reversible**, recorded before the screen adopts
    it, and **it reaches the queue**: the decision is written first, then the
    bill's queue file is withdrawn, and the screen says when one actually left
    (022 § 5; `review.rs`).
15. **Dates are `dd/mm/aaaa` on screen and ISO on the wire, converted in one
    place; an impossible date is refused before writing** — 31/02 defeats the
    duplicate check downstream and becomes a duplicated invoice (018 § 7). No
    native date input: the house calendar only (style manual).
16. **The source document is asked once per file; a failure is an answer, and
    trying again is a gesture** — the loop once burned MedLens's rate limit for
    every caller (018 § 8).
17. **The document is looked up locally first** (a crop, then this app's
    spool), MedLens last; an e-mail shows its HTML body, never the raw message
    (018 § 7).
18. ⚠️ **A crop is said, never drawn as the source document**: conferring a
    reading against a page the machine never saw is conferring it wrong. The
    original is the default; originals come only from this disk, never from
    MedLens, and a watched file that changed is refused (decision 019 § 6;
    project memory).
19. ⚠️ **The e-mail body stays fully sandboxed** — it is another process's
    HTML; the spotlight is written into it before it enters the frame, and no
    script ever runs there. The spotlight is HTML-only, cell-exact on the
    atendimento, and waits for a click (006 § 2).
20. **Nothing is open on arrival** — opening a file already showing a bill is
    the screen choosing for the operator (operator's call, 17/08/2026).
21. **The mirror suggests**: the search runs over the locally cached mirror,
    and a never-collected catalog leaves the field searchable with a note —
    but the gate still demands a code (rule 6).
22. **The journal never carries patient data**: corrections are logged by
    count, approvals and discards by bill position only (022 § 4).

### Must survive the redesign

- The document beside the bills, getting at least half the width.
- One open bill at a time; none open on arrival.
- The four matching outcomes, visibly distinct — automatic vs remembered
  (tooltip-level at least), not found vs found two (the homonym note) — and
  the read text always visible beside the code.
- The redo gesture on a standing match, and "redo" never drawn as an alert.
- The gate on the bill, above its actions, listing **what** is missing in the
  sentences above; Aprovar disabled while it holds, saying why.
- Inline confirmation, sign of life, one bill per gesture, no bulk approve.
- A refusal from the re-check shown in words, never raw keys.
- Struck items visible, in place, struck; never removed.
- Não enviar reachable and reversible ("Não enviar" view + "Voltar a
  conferir").
- Departed bills absent from this screen; the pointer to the Faturamento quiet.
- The crop note ("é isto que o MedLens leu" / "não é o que o MedLens leu")
  stated next to the toggle, and the original as the default.
- Sandbox, spotlight only for e-mail bodies and only after a click.
- The failure line carrying its own retry; no automatic retry.
- `dd/mm/aaaa` on screen; no native date input.
- Hover and editing look different on cells.

### Free to redesign

Card and row styling; where Copiar/Copiar tudo live; the segmented control's
look; the matched field's layout (keeping its content); the gate box's look
(keeping place and wording); the calendar's look; the viewer strip's
arrangement (keeping the note under the toggle); icons; the divider's look.

### Sample content for mockups

File: Hospital Vale Verde · `informe_vale_verde_2026-09-02.pdf` · 5 contas.
Tools bar: "1 no Faturamento →" · "A conferir (3)" · "Não enviar (1)" ·
"Todas (5)" (today "Todas" lists 4: the departed bill is not drawn).

| Bill                    | Insurer pill (as read) | Meta                               | Dot   | Fate            |
| ----------------------- | ---------------------- | ---------------------------------- | ----- | --------------- |
| MARTA LEMOS FERREIRA    | Vida Plena Saúde       | 01/09/2026 · 2026004517 · 3 itens  | green | a conferir      |
| JOÃO CARLOS PRADO       | AMPARO SAÚDE           | 03/09/2026 · 2026004602 · 2 itens  | amber | a conferir      |
| LUÍSA ANDRADE REIS      | PARTICULAR             | 04/09/2026 · 2026004633 · 2 itens  | amber | a conferir      |
| PEDRO HENRIQUE SALLES   | SUS                    | —                                  | —     | departed (not drawn) |
| BEATRIZ MONTEIRO        | Vida Plena Saúde       | 02/09/2026 · 2026004611 · 1 item   | grey  | não enviar      |

Open card — JOÃO CARLOS PRADO (gate holding):

- Paciente "JOÃO CARLOS PRADO" · Data da cirurgia "03/09/2026" · Atendimento
  "2026004602".
- CONVÊNIO · AMPARO SAÚDE → search field + warning + homonym note; typing
  "amparo" lists "000388  AMPARO SAÚDE" and "000391  AMPARO SAÚDE".
- MÉDICO · Dra. Helena Castro → "004533 DRA. HELENA CASTRO" + check (tooltip
  "Casamento seu, lembrado — o app não pergunta de novo.") + pencil.
- Items: `PB35-06` · PLACA BLOQUEADA 3,5 MM 6 FUROS · 1 · A7K2219 ·
  10000000002 — `FK16` · FIO DE KIRSCHNER 1,6 MM · 2 · sem lote · 10000000004.
- Gate: "Convênio → código — mais de um cadastro atende; escolha um" · "Lote —
  1 item(ns) não riscado(s) sem lote: linhas 2".
- Actions: "Aprovar" (disabled) · "Copiar" · "Não enviar".

Open card — MARTA LEMOS FERREIRA (gate clear): CONVÊNIO · Vida Plena Saúde →
"000412 VIDA PLENA SAÚDE" + check (automatic); MÉDICO · DR. RAFAEL NOGUEIRA →
"004512 DR. RAFAEL NOGUEIRA" + check (automatic); items `PC35-24T` · PARAFUSO
CORTICAL 3,5 MM X 24 MM TITÂNIO · 4 · 451288; `PB35-06` · PLACA BLOQUEADA 3,5
MM 6 FUROS · 1 · A7K2219; `CO-40` · CIMENTO ÓSSEO 40 G · 1 · C24B0917
(struck). No gate box; "Aprovar" → "Enviar esta conta para a fila do
Fantasma?" "Enviar" "Cancelar".

Open card — LUÍSA ANDRADE REIS: MÉDICO · DR. OTÁVIO PIRES (not in the mirror)
→ search field + warning; typing "pires" shows an empty list box. Gate:
"Médico → código — case o texto lido com o cadastro".

Another client's colinha line: "Colinha do adapter — falta Tabela, Cond. de
Venda (Configurações → Adapters)".

Document pane variants: the PDF informe in the native viewer ("Documento de
origem"); the e-mail body with the rows of 2026004602 lit under the veil; a
crop of `etiquetas_sala3.jpg` with "Recorte · **Original**", "‹ 1/2 ›" and
"O documento como chegou — não é o que o MedLens leu" (the crop reading's own
file name is unverified).

Toasts: "Correção gravada no MedLens." · "Conta enfileirada — ela agora vive no
Faturamento." · "A checagem onde o arquivo é escrito recusou — Médico → código
— case o texto lido com o cadastro · Lote — 1" · "“Não enviar” também retirou a
conta da fila." · "Data impossível — use dd/mm/aaaa."

### Design opportunities

1. **A pending mark on the edited field or cell.** Today a correction shows no
   sign of life (the old value stays until the re-read), a second edit made
   during a save is lost silently, and a failed save discards what was typed.
   A per-cell "saving / not saved — retry" state would honour the house's
   sign-of-life rule where the eye already is.
2. **Say the scope of a match.** One pick answers every bill with the same read
   text, in every file; a line like "vale para todas as contas que leem
   'AMPARO SAÚDE'" at pick time would make that visible.
3. **List the homonyms for "found two".** The operator must search to find the
   two entries; showing them directly would turn the question into a choice.
   Needs data: the match answer carries no candidates today.
4. **A sentence for "no result"** in the search list (today an empty box), and
   keyboard navigation (arrows + Enter) like the Colinha dialog's code field —
   the two "pick a registry code" patterns could become one.
5. **Tell atendimento from prontuário** when the fallback is showing (today the
   label says "Atendimento" either way).
6. **An "unknown" state for the gate** while the colinhas (or the matches) are
   not yet known, instead of a green dot and an enabled Aprovar.
7. **Gate lines that point.** "linhas 2, 5" could jump to those lot cells;
   "Convênio → código" could focus the field.
8. **A collapse gesture** on the open card, and a clear state after Aprovar or
   Não enviar (today the spotlight keeps lighting the bill that left, and
   nothing says where the next work is). An automatic "open the next bill"
   would contradict "nothing opens on its own" — offer it as a gesture, not a
   default.
9. **Make "Todas (n)" honest**: either count only what the view lists, or list
   departed bills there as quiet, read-only lines.
10. **Full text on demand** for ellipsized cells (material names are long),
    and a visible mark on added items and on corrected values (today both look
    like MedLens's reading).
11. **Consistent copy feedback** (one toast for both) and `dd/mm/aaaa` in the
    copied text, if the clipboard counts as "on screen".
12. **A "não enviar" banner inside an open discarded bill**, and a quiet undo
    toast after "Não enviar" (today the bill simply vanishes from "A
    conferir").
13. **An empty state for "Abrindo a extração…" that fails**: today the loading
    line stays forever under the red strip.
14. **Remember the divider position** within the 40–50% range the operator set.

### Open questions

- A doctor the ERP registry does not know (sample: DR. OTÁVIO PIRES), or a
  catalog never collected, leaves the bill with no way to be approved: the
  read text cannot be corrected or cleared here and codes only come from the
  mirror. Is there an intended path besides "Não enviar"?
- docs/telas.md and AGENTS.md demand "lot + quantity" on every non-struck item;
  the gate checks only the lot (a quantity is always a number, so 0 passes).
  Is a zero quantity acceptable?
- Should the insurer and doctor **texts** be correctable on this screen, as the
  other header fields are?
- What should the disabled CSV export contain (all bills? the view? which
  columns)? Its tooltip still says it "chega com a fila", and the queue exists.
- Should the Faturamento pointer open the Faturamento filtered to this file?
