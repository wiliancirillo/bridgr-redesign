# Consulta ao Everest — `/query`

> Where: sidebar › **Fora da esteira** › **Consulta** (icon `ph-brackets-curly`; **Relatórios** hangs under it as an indented sub-item — see [04-shell](../04-shell.md)) · Status: **built** · Code: `src/routes/query/+page.svelte`, `src/lib/query.svelte.ts`, `src/lib/components/SqlEditor.svelte`, `src/lib/everest-completion.ts`, `src/lib/everest.ts`, `src/lib/tsv.ts`, `src/lib/csv.ts`, `src/lib/sheet.ts`, `src/lib/grid-view.svelte.ts`, `src/lib/catalog-columns.ts` (`layoutKeyOf`), `src-tauri/src/lib.rs` (`query_fantasma`, `export_csv`, `save_sheet`, `grid_layout`), `src-tauri/src/fantasma.rs`, `src-tauri/src/queries.rs`

## The question

The i18n inventory gives this screen the question **"O que o ERP sabe, que a esteira ainda não responde?"** — it is not drawn anywhere today; the header carries the subtitle "Pelo backdoor do Fantasma — somente leitura" instead.

What the screen answers: the operator writes one read-only `SELECT` against the Everest ERP, waits tens of seconds, reads the answer in a grid, trims it into a draft, and then copies it, exports the ticked rows as CSV, or keeps it as a sealed **Relatório** ([07-sheets](07-sheets.md)).

## Where it sits in the work

- **Outside the conveyor.** Nothing here moves a bill and nothing is written to the ERP. It exists for the questions the conveyor cannot answer yet ("was this lot already invoiced?", "which orders does this insurer have this week?").
- **The channel is not a database connection.** The Fantasma (the robot that drives the Everest) opens the Everest's own SQL screen, types the query, runs it and exports the grid through the clipboard. Each query costs **tens of seconds** and holds the Fantasma's **single-flight lock**: while it runs, no catalog collection, no billing gesture and no other query can start — and a query cannot start while one of those holds the lock.
- **Entry points**: the sidebar; "Novo recorte" on Relatórios; "Rodar de novo" on a saved sheet (arrives with that sheet's SQL in the editor, and never runs it).
- **Exit points**: "Salvar como relatório" (a new sealed sheet in Relatórios), "Exportar seleção" (a CSV in the clear, where the operator points), "Copiar tudo" (clipboard).
- **The data is patient data** (LGPD). The answer lives in the app's memory only, for as long as the app is open; see rule 6.

## Anatomy

Reading order: a header across the top; under it two columns — the **asking rail** (fixed 452 px, left, divider on its right) and the **answer pane** (the rest). **[functional]** Both are visible at once: the trail is reused while an answer is read, and a refusal is read beside the editor that caused it.

### 1. Header

- Title **"Consulta ao Everest"**; under it **"Pelo backdoor do Fantasma — somente leitura"** (muted).
- Right end: the **clock tag** — a small accent-tinted pill with a clock icon. Three faces:
  - while a query runs: a live count "0,0s", "0,5s", "1,0s"… (every 0.5 s), the icon spinning — **[functional]** part of the sign of life;
  - otherwise: the duration of the newest *finished* line of the trail (answered or refused), e.g. "23,9s" — possibly a query from an earlier session;
  - absent while the trail holds no finished line.

### 2. Asking rail (left, top to bottom)

**2.1 SQL card** — a framed box.
- Head strip: kicker **"SQL"** (uppercase, quiet) left; **"somente SELECT · sem acento"** right.
- The **editor** (CodeMirror): monospace 11.5 px, lines wrap, placeholder **"SELECT TOP 20 …"**. 208 px tall by default; the operator drags its bottom-right corner to make it taller or shorter (minimum 120 px). What it paints and completes is in *Data shown → Editor*.

**2.2 Run row** — the primary (outlined) button **"▶ Consultar"**, and beside it the hint **"Ctrl+Enter"**. While running: **"◌ Consultando…"** with a spinner, disabled.

**2.3 Rules box** — a bordered paragraph that quotes the Fantasma's gate. **[functional]** It sits next to the editor.
- idle: **"Só SELECT (ou WITH), uma instrução por vez, sem comentário e sem acento — acento quebra a consulta no caminho até o editor do Everest. Quem recusa é o Fantasma, antes de tocar a tela."**
- while running, replaced by: **"O caminho é a tela do Everest: abrir o backdoor, digitar, executar e exportar pela área de transferência. Leva dezenas de segundos, e o Fantasma fica ocupado até acabar."**

**2.4 Map note** — one quiet line about the autocomplete's map of the ERP:
- loaded: **"Autocomplete sobre 3.212 tabelas do Everest, mapeadas em 05/08/2026"** (the count and the survey date come from the map file);
- loading: **"Carregando o mapa do Everest…"**;
- failed: **"Sem o mapa do Everest — o editor completa só palavras-chave do SQL."**

**2.5 Failure box** — only after a refused run: a danger-tinted block (`--st-danger-bg`, `--st-danger-fg`) with the refusal **verbatim**, line breaks kept. **[functional]** It lives in the rail, next to the question — never in place of the answer. It stays until the next run starts, also across navigation.

**2.6 Trail — "Consultas"**
- Kicker **"CONSULTAS"**.
- A filter field (funnel icon, placeholder **"Filtrar por trecho do SQL…"**, a clear × while it has text). It draws **no** "Ctrl ⇧ K" hint — that shortcut belongs to the result filter (rule 18).
- A scrolling list of cards filling the rest of the rail, newest first. Each card: the SQL on one monospace line (ellipsis), a meta line under it, a status dot, and an **×** (accessible name and tooltip **"Esquecer"**, danger hover). The card whose answer is on screen is outlined and tinted in the accent.
- Empty: **"Nenhuma consulta ainda."**

### 3. Answer pane (right)

**3.1 Empty frames** — before any answer, and for a proven-empty answer, the pane is one framed box with a centred empty state (see States). **[functional]** The frame matches the grid's box, so the pane does not jump when an answer arrives.

**3.2 Answer head** (one row, only when the answer has rows):
- the **count** — "42 linhas", or "12 de 42 linhas" while the result filter has text;
- the pill **"2 colunas vazias ocultas"** when columns came back empty end to end — **[functional]** never silent;
- the quiet note **"célula editável aqui dentro — nada volta para o Everest"** (ellipsis when narrow; no cell is editable today — Design opportunity 4);
- right end: **"Copiar tudo"** (turns into **"Copiado"** for 1.6 s) and **"Salvar como relatório"** (toggles 3.3). Both are small outlined buttons with an icon.

**3.3 Save box** (inline, amber-bordered and amber-tinted like everything awaiting a human, hugging the left edge — never a dialog): a title field **"Nome do relatório…"**, a scope note (**"todas as linhas à vista vão para o relatório"**, or **"3 linhas selecionadas vão para o relatório"** when rows are ticked), **"Cancelar"**, **"Salvar"** (turns into **"Salvando…"**). A red line under the box carries a save failure.

**3.4 The grid surface** — the house's shared grid shell ([05-components](../05-components.md), *GridSurface*), configured as:
- tools row: result filter **"Filtrar resultado…"** with the **"Ctrl ⇧ K"** hint; the count "42 linhas"; the chip **"3 linhas removidas"** (when rows were removed); **"Colunas"** with its badge "7 de 7"; the density pair; the page size **"50 ▾ por página"**;
- inside one framed box: the accent **selection bar** when rows are ticked — **"3 linhas selecionadas"**, [**"Marcar as 212 que o filtro achou"**], **"Excluir selecionadas (3)"** (danger tone), **"Exportar seleção"**, … **"Limpar"**; the grid; the pager **"← Anterior · 1–50 de 212 · Próxima →"**, with **"ordenado por NM_PACIENTE ↑"** at its right end while something sorts.

**3.5 Memory note** under the box: **"O resultado mora na memória desta sessão — em disco só quando você salva um relatório, e aí vai cifrado, por gesto seu. O histórico guarda só o metadado do que foi perguntado."** **[functional]** It is the screen's statement of where the data lives.

## Data shown

| Field | Meaning | Presentation rule | Example |
| --- | --- | --- | --- |
| Editor text | The SQL as typed. Lives in the app's memory; survives navigation | Mono, wraps | `SELECT TOP 20 …` |
| Syntax colours | Keywords accent-400 bold; strings `--st-ok`; numbers, true/false/NULL `--st-warn`; **comments `--st-danger`** (a comment is refused by the gate, so it is painted as an error, never as a grey annotation); operators and punctuation 55 % ink | Tokens only, so the editor follows every theme | — |
| Non-ASCII mark | Every character outside ASCII (accent, ç, curly quotes pasted from elsewhere, a non-breaking space) gets a red-tinted background | The criterion is "outside ASCII", not a list of accents | In `nm_paciente = 'JOÃO'` the `Ã` is marked |
| Completion popup | Tables: detail = approximate row count when the map was made ("959 linhas", "1 linha"). Columns: detail = type, plus "· pk" for a primary key ("char(13) · pk"). The typed stretch is in accent bold. Names with a space or a dot are inserted in brackets | Surface box, divider border, mono 11.5 px, at most ~16 lines, selected line on the quiet fill | — |
| Clock | Seconds, one decimal, comma | Pill in the header | "23,9s" |
| Map note | Tables in the map (pt-BR thousands dot) and the survey date | Quiet line | "3.212 tabelas … 05/08/2026" |
| Trail card — SQL | The question **redacted**: every single-quoted literal became `'…'` before it was written | One mono line, ellipsis | `SELECT … WHERE a.dt_uso >= '…'` |
| Trail card — meta | When it left (`dd/mm/aaaa hh:mm:ss`) and the outcome | Answered: "· 42 linhas · 23,9s"; refused: "· falhou · 11,2s"; no answer: "· sem resposta" | "10/09/2026 09:14:03 · 42 linhas · 23,9s" |
| Trail dot | Green = answered; red = refused; grey = no answer (still running, or the app closed mid-run) | Three facts, three tones | — |
| Trail length | Newest first; the file keeps the last 200 lines | — | — |
| Head count | Data rows the Everest answered (header excluded) — the **whole** answer: rows removed from the draft still count | "N linhas"; while filtering "{found} de {total} linhas", found = surviving rows the filter matches | "12 de 42 linhas" |
| Empty-columns pill | Columns empty in every row of the answer, dropped before the grid (they are not in Colunas and are not saved into a sheet) | Only when > 0 | "2 colunas vazias ocultas" |
| Tools count | Rows in the grid now: surviving rows the filter found | — | "39 linhas" |
| Removed chip | Rows removed from the draft since this answer arrived | Only when > 0 | "3 linhas removidas" |
| Colunas badge | Visible columns of the grid's columns (after the empty-column drop) | "N de M" | "7 de 9" |
| Pager range | First–last row of this page, of the rows the filter found | Plain digits, no thousands dot | "1–50 de 212" |
| Sorted label | The sorted column's name and direction | Mono; empty while unsorted | "ordenado por DT_USO ↓" |
| Selection label | Ticked rows — across all rows, not only the filtered ones or the page | Has a singular form | "1 linha selecionada" / "3 linhas selecionadas" |
| Select-all offer | Rows the filter found | pt-BR thousands dot | "Marcar as 3.653 que o filtro achou" |
| Cells | The TSV cells exactly as the Everest exported them | Grid voices: numeric columns right-aligned mono accent-300; first column accent and frozen; empty cell "—"; at most 280 px wide with the full value in the tooltip ([05-components](../05-components.md), *Grid*) | — |

Pluralization: the head count, the pill, the removed chip, the tools count and the save-scope note have **no singular form** in the code — one row reads "1 linhas".

## States

| State | Trigger | What is drawn |
| --- | --- | --- |
| Nothing asked yet | No answer in this app session | Answer pane: framed empty state, icon `ph-brackets-curly`, **"Nada consultado ainda"** / **"Escreva um SELECT e mande. Quem valida o SQL é o Fantasma: o que ele recusar aparece aqui com as palavras dele."** The rail is complete; the clock may show an older duration. No result filter exists yet, so Ctrl ⇧ K does nothing. |
| Blank editor | Editor empty or whitespace | Consultar disabled (45 % opacity), no reason given; Ctrl+Enter ignored |
| Map loading / loaded / missing | The ~1,6 MB map loads on demand, after the screen is already usable | The three map-note sentences; without the map, completion offers SQL keywords only |
| Running — the sign of life | Consultar or Ctrl+Enter | Button "Consultando…" + spinner (disabled); clock counting, icon spinning; rules box → running note; the previous failure box cleared; the trail re-read every 2 s so the in-flight query appears at the top (grey dot, "sem resposta"); the previous answer stays visible and usable |
| Answered with rows | The Fantasma returned a TSV with ≥ 1 data row | Answer head + grid. The whole view resets (filter, page, sort, columns, widths, ticks, removed rows, save box); then the column layout last used for **the same header shape** is re-applied. The query's trail card is highlighted |
| Proven empty | A TSV with a header and no data rows | Framed empty state, icon `ph-check-circle` (muted, not green), **"Nenhuma linha."** / **"O Fantasma provou o vazio com a query-canário — não é falha de leitura."** No answer head, no copy, no save |
| Refused after leaving | The query reached the robot and failed: the gate refused it, the server refused the SQL, or emptiness could not be proven | Failure box with "Fantasma: " followed by the robot's own words (its exact wording is the Fantasma's — unverified here); trail card with a red dot and "falhou"; **the previous answer stays on screen** |
| Refused before leaving | The lock was held, the session was closed, or the Fantasma is not configured | Failure box, verbatim: **"Fantasma: o Fantasma está ocupado — aguarde o gesto atual terminar"** (immediate); **"Fantasma: o Fantasma precisa estar com a sessão aberta para consultar — ligue em Configurações → Fantasma"** (after a few seconds of checking; also when the Fantasma is off, up without a session, or started by another program without a session); **"Fantasma: the Fantasma is not fully configured"** and **"Fantasma: <pasta> does not look like the Fantasma project (no compose.yaml)"** (English today). **No trail line** is written for any of these |
| Filter matched nothing | The result filter matches no visible cell | Inside the grid box: the header row alone, then icon `ph-magnifying-glass`, **"Nada com o que você digitou"** / **"O filtro olha as colunas à vista. A consulta em si respondeu normalmente — limpe o filtro para ver o resultado inteiro."**; head count "0 de 42 linhas"; pager "0–0 de 0" |
| Every row removed | All rows removed from the draft | Today the same no-match drawing as above, even with an empty filter (Design opportunity 11) |
| Rows ticked | ≥ 1 tick | Selection bar inside the box |
| Removal question | "Excluir selecionadas (N)" clicked | The button swaps in place into **"Excluir 3 do resultado? Nada muda no Everest."** [**Manter**] [**Excluir**]. If the ticked count changes while it is up, the question collapses back into the button |
| Save box open / saving / failed | "Salvar como relatório" | Amber box; Salvar disabled while the title is blank; "Salvando…"; a red failure line (e.g. a wallet refusal "the system wallet refused: …", a seal failure "selado: o selo não pôde ser aplicado", a disk error naming the path) |
| Exporting | "Exportar seleção" clicked | That button stays disabled until the system save dialog answers (no spinner) |
| Back on the screen | Navigated away and returned | Kept: editor text, the answer, the failure box, the running state, the trail. Lost: removed rows, ticks, the filter, the page, the page size, an open save box. Re-applied: the saved column layout for that header shape |
| Back mid-run | Returned while a query is still running | "Consultando…" still drawn, but the clock restarts from 0,0s (Design opportunity 18) |
| Fantasma off, busy or unconfigured, before asking | — | This screen does not show the Fantasma's state; the sidebar footer and the status bar do ([04-shell](../04-shell.md)). The screen learns it only by the refusal |
| Wide result | e.g. a `SELECT *` of 37 columns | The grid scrolls inside its box in both directions; header row and first column stay put; each column capped at 280 px (ellipsis + tooltip) unless dragged wider |
| Narrow window | Minimum window 1024 × 768 | The rail keeps its 452 px; with the sidebar expanded the answer pane is roughly 300 px wide; the tools row wraps onto several lines; the answer head's note truncates — unverified visually |

## Actions

| Gesture | Where | Precondition / disabled | Effect | Confirmation | Reversible | Feedback | Shortcut |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Type and complete SQL | Editor | — | Completion while a word is typed: after `alias.` or `Tabela.` → that table's columns; after `dbo.` → tables; otherwise the columns of the tables already named in `FROM`/`JOIN` first, then every table, plus SQL keywords. With nothing typed, only on demand | — | Editor undo | Popup | Ctrl+Space (on demand) |
| Consultar | Run row; editor | Editor not blank; not already running | Sends the text as typed | None | Read-only | Sign of life, then answer / failure box / trail line | **Ctrl+Enter** (editor focused) |
| Resize the editor | Card's bottom-right corner | — | Taller or shorter (min 120 px) | — | — | — | — |
| Reuse a trail line | Click a card | — | Replaces the editor text with the line's **redacted** SQL; does not run | None | Editor undo | Tooltip **"Volta o formato da consulta; os valores entre aspas você redigita — o rastro não os guarda."** | — |
| Esquecer | × on a card | — | Deletes the line from the trail file | None | **No** | The card disappears | — |
| Filter the trail | Trail filter | — | Case-insensitive substring over the redacted SQL | — | Clear × | List shrinks | — |
| Copiar tudo | Answer head | — | Tab-separated text: the visible header + every row the filter found (not only the page), in display order and current sort, removed rows excluded | None | — | Label "Copiado" for 1.6 s; no toast; a clipboard failure says nothing | — |
| Salvar como relatório | Answer head → save box | Title not blank | A new sealed sheet: the ticked rows if any, else every surviving row (**the filter is ignored**), full width, plus column order, hidden columns, sort and widths; unpinned; source SQL = the editor's text at that moment, redacted on the other side | The box itself | The sheet can be deleted in Relatórios | Toast **"Relatório salvo."**; the box closes; the operator stays here | Enter in the title field; Escape closes the box |
| Filter the result | Tools row | — | Case-insensitive over visible cells; page back to 1 | — | Clear × (**"Limpar filtro"**) | Counts update | **Ctrl ⇧ K** focuses it |
| Sort | Header click | — | Ascending → descending → unsorted — the grid's cycle, with its first-click exception ([05-components](../05-components.md), *Grid*) | — | Yes | Arrow, accent header, "ordenado por …" | — |
| Resize a column | Header's right edge | — | Drag; double-click = back to automatic | — | Desfazer edições | Live | — |
| Colunas | Tools row | The last visible column cannot be hidden | Show, hide, reorder; "Selecionar tudo"; "Desfazer edições" | — | Yes | Badge "N de M" | ArrowUp/ArrowDown on a grip |
| Density | Tools row | — | Comfortable or compact rows — **every grid in the app**, remembered | — | Yes | — | — |
| Page size | Tools row | — | 25 / 50 / 100 / 250 (default 50); page back to 1 | — | Yes | — | — |
| Page | Pager | Disabled at the ends | Previous / next page | — | — | Range | — |
| Tick rows | Tick column; rows | — | One box = one row; header box = this page; Shift = range from the last touched row, within the page; Ctrl/⌘ + click on a row toggles it | — | "Limpar" | Bar | Shift / Ctrl |
| Marcar as N que o filtro achou | Bar | Only when the page is fully ticked and the filter holds more | Ticks every row the filter found (adds, never replaces) | — | "Limpar" | Label count | — |
| Excluir selecionadas (N) | Bar | ≥ 1 tick | Removes the ticked rows from the draft — not from the ERP | In place (above) | **"Desfazer edições"**, until a new answer arrives or the screen is left | Chip "N linhas removidas" | — |
| Exportar seleção | Bar | ≥ 1 tick; no export in flight | CSV of the ticked rows (visible columns, their order, the current sort) to a path the operator picks in the system save dialog — title **"Exportar o recorte como CSV"**, filter **"Planilha CSV"**, suggested name **"consulta-selecao.csv"** | The dialog | The file is outside the app | Toast **"CSV exportado — o arquivo sai do app sem cifra."**; cancel = nothing; failure toast **"Não consegui gravar o CSV: …"** | — |
| Limpar | Bar | — | Unticks everything | — | — | The bar disappears | — |
| Desfazer edições | Colunas popover, footer | Only when there is something to undo | Columns back to the answer's order, all shown, automatic widths; removed rows back; ticks cleared (the sort stays) | None | — | Chip and badge update | — |
| Escape | Anywhere | — | Closes the save box and the grid's popovers | — | — | — | Esc |

## Business rules

1. **This side never validates the SQL.** The Fantasma's gate (`validar_select`) decides: `SELECT` or `WITH`, one statement, no comment, ASCII only. The rules box quotes it; the editor only *shows* the two mistakes whose error message never names its cause — a comment (painted red) and a non-ASCII character (marked red; it breaks on the clipboard path to the Everest). A second gate here would be a second source of truth, and the stale one would be this. [013 § 1; `SqlEditor.svelte`]
2. **A refusal is never dressed as an empty result.** "Empty" exists only when the Fantasma proved it with its canary query; when it cannot prove it, it refuses. Downstream, "nothing found" reads as "safe to invoice" — the predecessor sent eleven already-invoiced bills to the queue behind a fake "sem registro". [013 § 1; `AGENTS.md`]
3. **A refusal keeps the previous answer on screen** — taking it away would punish the operator twice for one refusal. [`query.svelte.ts`]
4. **The Fantasma's words are shown verbatim**, never rewritten, never interpreted. [013; `AGENTS.md`]
5. **Single-flight.** The lock is taken on the Rust side, before the session is checked: two queries would type over each other in the same Everest editor. "Busy" is refused at once, before seconds are spent on the session check. [`lib.rs` `query_fantasma`; 012]
6. **The answer lives in memory**, for the app's life, and leaves only by the operator's gesture: sealed (Salvar como relatório) or in the clear (Exportar seleção — always through a save dialog, to a path the operator picks, never one this side chose). [013 § 2; 014 § 2; `csv.ts`]
7. **The trail keeps metadata and a redacted question.** Every quoted literal becomes `'…'` before it is written (a patient's name in a `WHERE` is patient data); the refusal text is redacted too. The line is written **before** the query leaves, so a query survives the app being closed mid-run — it then reads "sem resposta", never "falhou". [013 § 2–3; `queries.rs`]
8. **Reusing a trail line brings back a shape, not the question** — the values must be retyped, and the tooltip says so. [013 § 3]
9. **Columns empty end to end are hidden and said out loud** (the pill) — a column the ERP answered and the app hid without a word would be the screen deciding what the operator may know. With no data rows nothing is hidden. [013 § 6; `tsv.ts`]
10. **A new answer resets the whole view**: a filter typed for one result silently emptying the next reads as a query that answered nothing, and a sort carried into another table sorts a different column. The layout saved for the **same header shape** is applied on top — never for another shape. [`grid-view.svelte.ts`; 026 § 4]
11. **Numeric alignment is decided over the whole result**, never the page — or a column flips alignment when the page turns. [`sorting.ts`]
12. **The head count is what the Everest answered**; while filtering it reads "found de total". The proven-empty drawing depends on the whole answer, never on the filter. [page]
13. **The grid is the draft.** On this screen removals, hiding and reordering change the screen, the copy and what a save freezes. [014 § 6]
14. **Removing rows asks, in place, and can be undone.** The question carries the count and dies when the count changes; "Desfazer edições" restores rows and columns in one gesture. [014 § 6a, § 8]
15. **Ticks span all rows, never just the filter**: the bar's number is the number exported. [006 § 2]
16. **Copiar tudo copies exactly what is on screen**: all filtered rows, visible columns, their order and sort, removals applied. [page]
17. **A save from here is always a new sheet.** Ticked rows are the finer knife; the transient filter is never captured. [014 § 6]
18. **One `Ctrl ⇧ K` per screen**: the result filter owns it; the trail filter draws no hint. Two fields claiming one shortcut would make one hint a lie. [`FilterField.svelte`]
19. **The map is generated, loaded on demand and dated**; a failed load is not fatal; names outside ASCII are pruned from it, because suggesting what the gate refuses is worse than suggesting nothing. [013 § 5; `everest.ts`]
20. **CSV rules**: `;` separator (Excel in pt-BR reads `,` as a decimal mark), CRLF line ends, UTF-8 with a BOM (or accented names open garbled), RFC 4180 quoting, and a tab before any cell starting with `=`, `+`, `-` or `@` so a spreadsheet does not execute it. [`csv.ts`; `lib.rs` `export_csv`]

## Must survive the redesign

- The refusal, verbatim, beside the question — and the previous answer kept under it.
- Three distinct empties: never asked; proven empty (with the canary sentence); filter miss inside the grid box with the header row kept.
- The sign of life before the answer: busy Run button, running clock, running note; Run unavailable while running.
- The gate's rules near the editor; comments painted as errors; non-ASCII marked in place; the map's date and the "sem o mapa" sentence.
- The empty-columns pill; the "12 de 42 linhas" count while filtering.
- The trail: redacted text visibly redacted (`'…'`), the retype warning on reuse, three outcome tones (answered, refused, no answer), "Esquecer".
- The memory note saying where the data lives.
- Exportar seleção only in the selection bar, always through the save dialog, and its toast saying the file leaves without cipher.
- The destructive selection gesture keeps its danger tone inside the accent bar; its confirmation is in place (never modal) and names the count.
- "Desfazer edições" restores rows and columns together.
- One `Ctrl ⇧ K` on the screen.
- The answer survives navigation.

## Free to redesign

- The 452 px rail, the two-column split, the card styling, and where the trail lives (drawer, tab, collapsible rail) — as long as it stays reachable while asking.
- The clock's shape and place.
- The answer head's arrangement (Design opportunity 3); the save box's shape — keep it non-modal, the house idiom; the density icons; the pager wording layout.
- Cell typography, except that codes and numbers stay monospace ([02-invariants](../02-invariants.md)).
- Icons (the Phosphor names are today's choice, not a rule).

## Sample content for mockups

Fictitious throughout ([06-sample-data](../06-sample-data.md)); table and column names are illustrative, not the ERP's real schema. The SQL itself must be ASCII (rule 1); the data can carry accents.

Editor:

```sql
SELECT TOP 20 a.cd_atendimento, a.nm_paciente, a.dt_uso,
       c.nm_convenio, c.cd_convenio, i.lote, i.qtde
FROM Atendimento a
JOIN Convenio c ON c.cd_convenio = a.cd_convenio
JOIN ItemConsumo i ON i.cd_atendimento = a.cd_atendimento
WHERE a.dt_uso >= '2026-09-01'
```

Trail, newest first:

| SQL as stored | Meta | Dot |
| --- | --- | --- |
| `SELECT TOP 20 a.cd_atendimento, a.nm_paciente, … WHERE a.dt_uso >= '…'` | 10/09/2026 09:14:03 · 42 linhas · 23,9s | green (current card) |
| `SELECT lote, qtde FROM ItemConsumo WHERE referencia = '…'` | 10/09/2026 09:02:47 · falhou · 11,2s | red |
| `SELECT COUNT(*) FROM Pedido WHERE cd_convenio IN ('…', '…')` | 09/09/2026 17:40:12 · sem resposta | grey |

Failure box: "Fantasma: o Fantasma está ocupado — aguarde o gesto atual terminar".

Result — the Everest answered 9 columns, 2 of them empty in every row: head "42 linhas" · "2 colunas vazias ocultas"; tools "42 linhas" · "Colunas 7 de 7"; pager "1–42 de 42" (default 50 per page).

| CD_ATENDIMENTO | NM_PACIENTE | DT_USO | NM_CONVENIO | CD_CONVENIO | LOTE | QTDE |
| --- | --- | --- | --- | --- | --- | --- |
| 2026004517 | MARTA LEMOS FERREIRA | 02/09/2026 | VIDA PLENA SAÚDE | 000412 | 0000451288 | 2 |
| 2026004602 | JOÃO CARLOS PRADO | 02/09/2026 | AMPARO SAÚDE | 000391 | A7K2219 | 1 |
| 2026004633 | LUÍSA ANDRADE REIS | 03/09/2026 | SUS | 000001 | C24B0917 | 1 |
| 2026004651 | PEDRO HENRIQUE SALLES | 03/09/2026 | PARTICULAR | 000002 | 0000902211 | — |

How it draws: CD_ATENDIMENTO, CD_CONVENIO and QTDE are numbers all the way down (QTDE's blank is ignored), so they are right-aligned in accent-300 mono; the first column is also the frozen identity column in the accent; LOTE stays left-aligned text because one value (A7K2219) is not a number; DT_USO stays text (dates are never numeric). The date format is whatever the Everest exports — the sorter understands `dd/mm/aaaa`, optionally followed by a time.

Selection: three rows ticked → bar "3 linhas selecionadas · Excluir selecionadas (3) · Exportar seleção · Limpar"; question "Excluir 3 do resultado? Nada muda no Everest." Save box: title "Consumo OPME Vale Verde — semana 36", note "3 linhas selecionadas vão para o relatório".

## Design opportunities

1. **Say the Fantasma's state before the click.** Today the screen learns "off", "busy" or "unconfigured" only from a refusal, sometimes after seconds of checking. A quiet line beside Consultar mirroring the live state the status bar already shows would save a wasted gesture. The refusal must stay the truth: show the state with its age, never disable the button on a stale reading.
2. **Show which question produced the answer.** The answer pane never shows its SQL; the editor may have changed since, and "Salvar como relatório" stores the editor's current text as the sheet's source. A caption over the answer (the highlighted trail card's text) would make provenance visible.
3. **One count line instead of two.** The head says "42 linhas" and the tools row says "42 linhas"; after removals they diverge ("42" vs "39") without saying why. One line such as "39 de 42 linhas · 3 removidas" answers both.
4. **The "célula editável" note promises what does not exist.** No cell can be edited on this screen; what the draft allows is removing rows and hiding, reordering and resizing columns. Reword or drop.
5. **Tell "running" from "no answer", and keep the refusal reachable.** The in-flight query reads "sem resposta", the same words as a query orphaned by an app closed mid-run; and a failed card shows only "falhou" although the (redacted) refusal is stored with it.
6. **Tell the trail's two empties apart.** "Nenhuma consulta ainda." is drawn both for an empty trail and for a filter that matched nothing — the house rule is one sentence per kind of absence.
7. **Bring the four SQL hints to this editor.** The app already has them (double quotes used as text, `= NULL`, a trailing `;`, `BETWEEN` two bare dates) and draws them as a lightbulb list under the Catálogos SQL dialog — warnings, never a gate. The Consulta, where queries are hand-written, does not show them.
8. **Syntax colours borrow the state roles.** Strings in `--st-ok` green and numbers in `--st-warn` amber use semantic colours decoratively, which the style manual forbids ("amber = a human is needed"). The red comment is deliberate (an error); the other two are not.
9. **The save scope note says "à vista", but the filter is ignored.** With 12 of 42 rows filtered and nothing ticked, the save takes all 42 surviving rows. Say what is taken ("todas as 42 linhas do resultado — o filtro não entra").
10. **After saving, a way to the new Relatório.** The toast says it was saved and the operator stays here; a link to open it (or to the list) would close the loop.
11. **A sentence for "every row removed".** Today it wears the filter-miss sentence even with an empty filter — an absence dressed as another.
12. **Keep the draft's removals across navigation, or ask before losing them.** Column edits come back from the saved layout; removed rows and ticks do not, silently.
13. **A visible sign of life on Exportar seleção.** Only the disabled state marks the time the dialog is open.
14. **Singular forms and one number style.** "1 linhas", "1 colunas vazias ocultas"; the pager writes "3653" while the select-all offer writes "3.653".
15. **One notation for shortcuts.** "Ctrl+Enter" here, "Ctrl ⇧ K" in the filter.
16. **The narrow window.** At the minimum window size the fixed rail leaves the answer about 300 px; a collapsible rail would give the grid the room.
17. **Name the hidden empty columns.** The pill counts them, but they cannot be seen or restored from Colunas; a hover listing their names would keep the promise "never silent" complete.
18. **The clock after returning mid-run** restarts from zero; it could count from the in-flight trail line's start stamp.

## Open questions

1. Should a saved sheet's source SQL be the question that produced the answer, rather than the editor's text at save time?
2. Should the head count subtract rows removed from the draft?
3. Should removed rows and ticks survive leaving the screen?
4. Should the Consulta show the Fantasma's live state, given that the refusal is the only verified answer at click time?
5. Where, if anywhere, should the screen's question ("O que o ERP sabe, que a esteira ainda não responde?") appear?
6. Should the all-empty columns be listed, restorable, or saved into a sheet?
7. Should a refusal's text remain readable after the next run (it is stored, redacted, in the trail)?
