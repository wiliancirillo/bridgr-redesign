# Relatórios — `/sheets` and `/sheets/[id]`

> Where: sidebar › **Fora da esteira** › Consulta › **Relatórios** (indented sub-item, icon `ph-chart-bar`); `/sheets/[id]` opens from a line of the list and keeps "Relatórios" lit ([04-shell](../04-shell.md)) · Status: **built** (both) · Code: `src/routes/sheets/+page.svelte`, `src/routes/sheets/[id]/+page.svelte`, `src/lib/components/SavedList.svelte`, `src/lib/saved-list.ts`, `src/lib/components/GridSurface.svelte`, `src/lib/grid-view.svelte.ts`, `src/lib/csv.ts`, `src/lib/sheet.ts`, `src/lib/query.svelte.ts` (Rodar de novo), `src-tauri/src/sheets.rs`, `src-tauri/src/seal.rs`, `src-tauri/src/vault.rs`, `src-tauri/src/lib.rs` (`save_sheet`, `list_sheets`, `read_sheet`, `set_sheet_pinned`, `delete_sheet`, `export_csv`)

## The question

- The list: **"O que você guardou do Everest — recortes salvos, cifrados nesta máquina."** (the i18n `question.sheets`, drawn as the list's subtitle).
- One sheet has no sentence on screen. In prose: _is this the cut I kept — and do I read it, trim it, export it, or ask its question again?_

## Where it sits in the work

- A **Relatório** ("sheet" in code) is a [Consulta](06-query.md) answer the operator edited and chose to keep, so it can be reread without paying the backdoor's tens of seconds again. It is born only from the Consulta's "Salvar como relatório".
- ⚠️ **Vocabulary.** On screen, "relatório" means only this. The hospital's consolidated spreadsheet is the **informe** — never label it "relatório" (glossary; decisions 002 and 014).
- **A photograph of the data, inside a live document.** The rows are the Everest's answer on the day of the query — never re-queried, never refreshed (the list's footer: "Relatório é foto, não consulta viva"). The document around them is editable: rename, hide/reorder/resize columns, sort, remove rows — and **Salvar** rewrites the same file. Listas, the cuts of Catálogos, are the inverse: live data ([09-shortlists](09-shortlists.md)).
- **Patient data at rest.** Each sheet is one encrypted file on this machine; the key lives in the system wallet.
- **Exits**: CSV in the clear (per line on the list; whole sheet or ticked rows on the sheet); "Rodar de novo" → the Consulta carrying the sheet's (redacted) SQL.

## Anatomy

### A. The list — `/sheets`

Drawn by the shared **SavedList** ([05-components](../05-components.md), _SavedList_); this screen supplies the nouns, the second line, the counts and the footer.

**A1. Header** — title **"Relatórios"**, subtitle (the question above); right: the screen's one primary button **"+ Novo recorte"**, which opens the Consulta. **[functional]** It leads out: a cut is made where the data is, never here.

**A2. Tools row** (only when the list loaded with ≥ 1 sheet): filter **"Filtrar por nome ou trecho do SQL…"** with **"Ctrl ⇧ K"**; a segmented pair **"Todos 3"** | **"Fixados 1"**; spacer; summary **"3 recortes · 41 linhas guardadas"**; the order button (sort icon + current order + caret; accessible name "Ordenar relatórios") opening a small menu **"Mais recentes"** · **"Nome (A–Z)"** · **"Mais linhas"**, the current one checked.

**A3. The list — one line per sheet, not a card grid.** **[functional]** A column of names read top to bottom, with the numbers in their own right-hand column so they compare down the list. Each line:

- a table icon (accent when pinned);
- the title (13.5 px, 500) and, when pinned, the tag **"FIXADO"** (the word is "fixado", uppercased by style);
- under it, the source SQL, **redacted**, on one mono line with an ellipsis;
- the right column (150 px): **"21 linhas · 9 col."** over **"alterado em 09/09/2026 16:22:05"**;
- three icon buttons: pin (**"Fixar no topo"** / **"Desafixar"**; filled pin when pinned), export (**"Exportar CSV"**), trash (**"Excluir relatório"**, danger hover).
- A pinned line also carries an accent-tinted border — pinned is stated twice, because a tag alone disappears in a list of ten. Hovering a line tints its border in the accent. Clicking anywhere on the line except the three buttons opens the sheet.

**A4. Inline delete confirmation** — directly under the line whose trash was clicked, an amber-bordered strip: **"Excluir este relatório apaga o arquivo cifrado — não há lixeira."** [**Manter**] [**Excluir**] (Excluir turns red on hover). One strip at a time.

**A5. Footer** (when ≥ 1 sheet and no failure), under a divider: a lock icon + **"Relatório é foto, não consulta viva: o que está aqui é o resultado do dia em que você salvou. Reabra na Consulta para rodar de novo."**

### B. One sheet — `/sheets/[id]`

**B1. Header** (one row):

- a back arrow to the list (accessible name **"Voltar aos relatórios"**);
- the title block: the **name as an editable field** drawn like a heading (16 px, 500; transparent border, a divider border on hover, the accent border and the surface fill on focus; placeholder **"Nome do relatório…"**); a pencil (**"Renomear o relatório"**) that puts the caret in the field and selects its text; the badge **"🔒 cifrado"** (quiet fill); under them one quiet line **"salvo em 09/09/2026 16:22:05 · 21 linhas · 9 colunas"**;
- right: **"Copiar tudo"** (turns into "Copiado" for 1.6 s) and **"Exportar CSV"** (outlined, quiet); **"▶ Rodar de novo"** (accent-tinted — the screen's primary); **"💾 Salvar"** (accent-tinted when enabled, plain when disabled; turns into **"Salvando…"**); **"⋮"** (**"Mais ações"**) opening a small menu whose one item is **"🗑 Excluir relatório"** (danger hover). **[functional]** Excluir lives behind the ⋮, never beside Salvar — one slip from being the button clicked.

**B2. Delete strip** — after choosing "Excluir relatório", at the top of the body, amber: **"Excluir este relatório apaga o arquivo cifrado — não há lixeira."** [**Manter**] [**Excluir**].

**B3. Failure line** (red) when a save or a delete failed.

**B4. The grid surface**, the same shell as the Consulta ([05-components](../05-components.md), _GridSurface_): filter **"Filtrar em todas as colunas…"** with "Ctrl ⇧ K"; count "18 linhas"; chip **"3 linhas removidas"**; "Colunas" with "7 de 9"; density pair; page size; in the box, the selection bar — "3 linhas selecionadas", ["Marcar as 21 que o filtro achou"], **"Excluir selecionadas (3)"** (danger), **"Exportar seleção"**, "Limpar"; the grid; the pager with "ordenado por …" at its right end.

**B5. Sealed note** under the box: **"Este relatório mora cifrado nesta máquina; a chave fica no chaveiro do sistema. O SQL de origem guarda só o formato — os valores entre aspas viram '…'."**

## Data shown

### List

| Field                   | Meaning                                                                                                                          | Presentation rule              | Example                               |
| ----------------------- | -------------------------------------------------------------------------------------------------------------------------------- | ------------------------------ | ------------------------------------- |
| Title                   | Typed by the operator when saving or renaming. It may name anything, so it lives inside the encryption                           | Trimmed; ellipsis              | "Consumo OPME Vale Verde — semana 36" |
| Second line             | The source SQL, redacted (`'…'` for every literal)                                                                               | Mono, one line, ellipsis       | `SELECT … WHERE a.cd_local = '…'`     |
| "N linhas · M col."     | Rows stored in the file · **all** columns in the file, hidden ones included                                                      | Plural-only ("1 linhas")       | "21 linhas · 9 col."                  |
| "alterado em …"         | The file's last save. Pinning does not move it                                                                                   | `dd/mm/aaaa hh:mm:ss`          | "alterado em 09/09/2026 16:22:05"     |
| "Todos N" / "Fixados N" | Sheets listed / pinned among them                                                                                                | In the segmented pair          | "Todos 3", "Fixados 1"                |
| Summary                 | Sheets listed · the sum of their stored rows — the whole list, regardless of filter and tab                                      | Plain digits, no thousands dot | "3 recortes · 41 linhas guardadas"    |
| Order                   | Pinned always first; then Mais recentes (last save, newest first), Nome (A–Z) (pt-BR collation) or Mais linhas (most rows first) | —                              | —                                     |

A file that cannot be opened (corrupt) is skipped from the list and from every count, without a word (rule 5).

### One sheet

| Field                  | Meaning                                                                             | Presentation rule                                                                                       | Example                               |
| ---------------------- | ----------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- | ------------------------------------- |
| Name field             | The title, editable in place                                                        | Dirty when the trimmed text differs from the file                                                       | "Consumo OPME Vale Verde — semana 36" |
| "salvo em …"           | The file's last save                                                                | The **file's** fact, not the draft's — it stops agreeing with the screen the moment something is edited | "salvo em 09/09/2026 16:22:05"        |
| "N linhas · M colunas" | Rows and columns in the file (hidden columns counted)                               | Plural-only                                                                                             | "21 linhas · 9 colunas"               |
| Tools count            | Rows the filter found among the surviving rows                                      | —                                                                                                       | "18 linhas"                           |
| Removed chip           | Rows removed since the file was opened or last saved                                | Only when > 0                                                                                           | "3 linhas removidas"                  |
| Colunas badge          | Visible of all columns                                                              | "N de M"                                                                                                | "7 de 9"                              |
| Grid                   | Stored rows, drawn through the stored column order, hidden columns, widths and sort | The Consulta grid's voices ([06-query](06-query.md))                                                    | —                                     |

Not shown anywhere on this screen: the source SQL (used only by Rodar de novo), the moment the Everest answered (stored with the sheet, never displayed), and the creation date.

## States

### List

| State              | Trigger                                        | What is drawn                                                                                                                                                                                                                                                                                                                             |
| ------------------ | ---------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Loading            | First read — every file is decrypted to answer | Header only; the body is blank (no sign of life)                                                                                                                                                                                                                                                                                          |
| Nothing saved      | No sheet on this machine                       | Empty state, table icon: **"Nenhum relatório salvo"** / **"Na Consulta, edite o resultado e use “Salvar como relatório” — ele fica guardado aqui, cifrado nesta máquina."** No tools row, no footer                                                                                                                                       |
| Filter miss        | The filter matches no title and no SQL         | Magnifying glass: **"Nada com esse filtro"** / **"O filtro olha o nome e o SQL de origem. Talvez o relatório tenha outro nome — ou ainda não exista."**                                                                                                                                                                                   |
| Nothing pinned     | "Fixados" tab, empty filter, no pinned sheet   | Magnifying glass: **"Nenhum relatório fixado"** / **"Fixe um relatório pelo alfinete da lista para ele subir ao topo e ficar aqui."**                                                                                                                                                                                                     |
| Lost key           | Files present, key gone from the wallet        | The body is one red sentence: **"selado: a chave do relatório sumiu do chaveiro — os arquivos continuam, ilegíveis"** — no tools, no list, no footer; header and "Novo recorte" remain. Siblings: "selado: a chave do relatório no chaveiro está corrompida"; a wallet refusal arrives as "the system wallet refused: …" (English prefix) |
| Exporting one line | A line's export clicked                        | Every line's export button disabled until the save dialog answers (the file is decrypted first)                                                                                                                                                                                                                                           |
| Delete confirming  | Trash clicked                                  | The amber strip under that line                                                                                                                                                                                                                                                                                                           |
| Pin failed         | The pin toggle was refused                     | The red sentence replaces the whole list until the screen is reopened (current behaviour; Design opportunity 6)                                                                                                                                                                                                                           |
| Delete failed      | The delete was refused                         | The red sentence appears, then the list refresh that follows clears it (current behaviour; Design opportunity 6)                                                                                                                                                                                                                          |
| Narrow window      | Minimum window 1024 × 768                      | The tools row does not wrap — unverified visually                                                                                                                                                                                                                                                                                         |

### One sheet

| State                                      | Trigger                                                                                | What is drawn                                                                                                                                                                                                                                                                                                                                                                                                  |
| ------------------------------------------ | -------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Loading                                    | Opening                                                                                | The header shows only the back arrow; the body is blank                                                                                                                                                                                                                                                                                                                                                        |
| Load failure                               | Missing file, unreadable file, key gone, or an id this side never minted               | Body: the red sentence + a link **"Voltar aos relatórios"**; header: only the back arrow. Sentences: **"selado: este relatório não existe mais"**; **"selado: o arquivo do relatório está ilegível — o selo não confere"**; **"selado: a chave do relatório sumiu do chaveiro — os arquivos continuam, ilegíveis"**; for a malformed id, "the sheet was refused: the id is not one this side minted" (English) |
| Clean                                      | Just opened or just saved                                                              | Salvar disabled                                                                                                                                                                                                                                                                                                                                                                                                |
| Dirty                                      | Title, column order, hidden columns, widths, sort or removed rows differ from the file | Salvar enabled — nothing else marks it (Design opportunity 1)                                                                                                                                                                                                                                                                                                                                                  |
| Blank title                                | Name field emptied                                                                     | Salvar disabled; no reason shown                                                                                                                                                                                                                                                                                                                                                                               |
| Saving                                     | Salvar                                                                                 | "Salvando…", disabled. Success: toast **"Relatório salvo."**, the chip goes, Salvar disables — but the "salvo em · linhas · colunas" line keeps the values it loaded until the sheet is reopened (current behaviour; Design opportunity 3)                                                                                                                                                                     |
| Save failed                                | Refused                                                                                | The red line above the grid                                                                                                                                                                                                                                                                                                                                                                                    |
| No rows to show                            | The filter finds nothing, every row was removed, or the file holds no row              | Inside the grid box: the header row + **"Nenhuma linha com o que você digitou"** (no second line)                                                                                                                                                                                                                                                                                                              |
| Rows ticked / removal question / exporting | As on the Consulta                                                                     | Removal question: **"Excluir 3 deste relatório? Só some do arquivo ao salvar."** [**Manter**] [**Excluir**]                                                                                                                                                                                                                                                                                                    |
| Menu open                                  | ⋮                                                                                      | Closes on a click outside or Escape                                                                                                                                                                                                                                                                                                                                                                            |
| Delete confirming / failed                 | "Excluir relatório"                                                                    | The amber strip; on failure the red line, and the strip closes                                                                                                                                                                                                                                                                                                                                                 |
| Leaving with unsaved edits                 | Back arrow, sidebar, Rodar de novo                                                     | The edits are discarded silently — no question (Design opportunity 1)                                                                                                                                                                                                                                                                                                                                          |
| Narrow window                              | —                                                                                      | The header row does not wrap; the name field shrinks first — unverified visually                                                                                                                                                                                                                                                                                                                               |

## Actions

### List

| Gesture           | Where          | Precondition              | Effect                                                                                                                                                                                                                                                                                                   | Confirmation | Reversible                  | Feedback                                                                                                            | Shortcut                               |
| ----------------- | -------------- | ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------------------- | ------------------------------------------------------------------------------------------------------------------- | -------------------------------------- |
| Novo recorte      | Header         | —                         | Opens the Consulta                                                                                                                                                                                                                                                                                       | —            | —                           | —                                                                                                                   | —                                      |
| Filter            | Tools row      | —                         | Title or redacted SQL, case-insensitive                                                                                                                                                                                                                                                                  | —            | Clear ×                     | List shrinks                                                                                                        | **Ctrl ⇧ K**                           |
| Todos / Fixados   | Tools row      | —                         | All sheets / pinned only                                                                                                                                                                                                                                                                                 | —            | Yes                         | —                                                                                                                   | —                                      |
| Order             | Tools row menu | —                         | Mais recentes / Nome (A–Z) / Mais linhas; pinned stay first                                                                                                                                                                                                                                              | —            | Yes                         | Check on the current                                                                                                | Menu closes on outside click or Escape |
| Open              | The line       | —                         | Opens `/sheets/[id]`                                                                                                                                                                                                                                                                                     | —            | —                           | —                                                                                                                   | —                                      |
| Pin / unpin       | Line           | —                         | Flips the flag inside the sealed file; the line updates in place and pinned lines rise; "alterado em" does not change                                                                                                                                                                                    | None         | Yes (the same button)       | No toast                                                                                                            | —                                      |
| Exportar CSV      | Line           | No other export in flight | Decrypts the file and exports **what is saved**: stored column order, hidden columns left out, stored sort. Save dialog **"Exportar o recorte como CSV"**, filter **"Planilha CSV"**, name suggested from the title (e.g. "Consumo-OPME-Vale-Verde-semana-36.csv"; an empty title gives "relatorio.csv") | The dialog   | The file is outside the app | **"CSV exportado — o arquivo sai do app sem cifra."**; cancel = nothing; failure **"Não consegui gravar o CSV: …"** | —                                      |
| Excluir relatório | Trash          | —                         | Deletes the file                                                                                                                                                                                                                                                                                         | Inline strip | **No** — "não há lixeira"   | The line disappears; no toast                                                                                       | —                                      |

### One sheet

| Gesture                  | Where              | Precondition                       | Effect                                                                                                                                                                                    | Confirmation | Reversible                                    | Feedback                                                                                                                                                             | Shortcut                     |
| ------------------------ | ------------------ | ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------- |
| Back                     | Arrow              | —                                  | To the list; unsaved edits are lost                                                                                                                                                       | None         | —                                             | —                                                                                                                                                                    | —                            |
| Rename                   | Name field; pencil | —                                  | Edits the title; kept only by Salvar                                                                                                                                                      | —            | Until saved                                   | Salvar lights up                                                                                                                                                     | —                            |
| Copiar tudo              | Header             | —                                  | Tab-separated text of the rows the filter found, visible columns, their order and sort, unsaved edits included                                                                            | —            | —                                             | "Copiado" for 1.6 s                                                                                                                                                  | —                            |
| Exportar CSV             | Header             | No export in flight                | CSV of **what is on screen**: every row the filter found (not only the page), visible columns, their order and sort, unsaved edits included, ticks ignored; name suggested from the title | The dialog   | Outside the app                               | Same toasts as the list                                                                                                                                              | —                            |
| Rodar de novo            | Header             | —                                  | Puts the stored SQL into the Consulta's editor (replacing whatever was there), then opens the Consulta; **never runs it**. When the SQL holds `'…'`, a warning toast                      | None         | —                                             | Warning toast **"O SQL salvo tem literais apagados pelo selo (aparecem como '…'). Redigite-os antes de rodar — como está, a consulta roda e responde outra coisa."** | —                            |
| Salvar                   | Header             | Dirty, title not blank, not saving | Rewrites the same file: same id and creation date, a new "salvo em", removed rows pruned for good, the current column order, hidden columns, sort and widths, the pin kept                | None         | Pruned rows are gone                          | Toast **"Relatório salvo."**                                                                                                                                         | —                            |
| ⋮ → Excluir relatório    | Header menu        | —                                  | Deletes the file and returns to the list                                                                                                                                                  | Amber strip  | **No**                                        | No toast                                                                                                                                                             | Escape closes menu and strip |
| Grid gestures            | Surface            | —                                  | Filter, sort, resize, Colunas (its "Desfazer edições" also restores removed rows), density, page size, ticks, "Marcar as N que o filtro achou"                                            | —            | Yes                                           | —                                                                                                                                                                    | Ctrl ⇧ K, Shift, Ctrl        |
| Excluir selecionadas (N) | Selection bar      | ≥ 1 tick                           | Removes the ticked rows from the draft                                                                                                                                                    | In place     | "Desfazer edições", or leaving without saving | Chip "N linhas removidas"                                                                                                                                            | —                            |
| Exportar seleção         | Selection bar      | ≥ 1 tick                           | CSV of the ticked rows (visible columns, order, sort); name suggested "<título>-selecao.csv"                                                                                              | The dialog   | Outside the app                               | Same toasts                                                                                                                                                          | —                            |

## Business rules

1. **"sheet" in code, "Relatórios" on screen**; "report" is the glossary's word for the informe, and a second synonym for a glossary word is a defect. [014 § 1; 002]
2. **Nothing persists without the gesture.** Query results live in memory; a sheet is the deliberate exception, created only by "Salvar como relatório". [014 § 2]
3. **One sealed file per sheet**, ChaCha20-Poly1305, key minted at the first save and kept in the system wallet; the title lives inside the ciphertext; the file name is opaque; the file is readable by its owner only. [014 § 3, § 5; `sheets.rs`, `vault.rs`]
4. **The stored SQL is redacted even under the seal** — the `WHERE` may name the very patient the operator removed from the rows. [014 § 4]
5. **A lost key fails out loud**, never as an empty list; a machine that never saved a sheet never asks the wallet; one unreadable file is skipped in the list (one rotten file must not hide the healthy ones) but fails loudly when opened by name. [014 § 7]
6. **Pinned always rises**, whatever the order; pinning does not move "alterado em" (the list would reorder under the operator's finger); the answered line is replaced in place (re-listing decrypts every file). [014 § 6b]
7. **Three empties are told apart**: nothing saved, nothing pinned, nothing matching. [`docs/telas.md`]
8. **Photograph of the data, live document.** The rows are never refreshed from the ERP; the document is edited and re-saved in place, pruning removed rows. [014 § 6]
9. **The title block's counts are the file's**, not the draft's. [`docs/telas.md`]
10. **Excluir sits behind the ⋮**, never beside Salvar. [`docs/telas.md`]
11. **Rodar de novo loads and never runs.** The redacted SQL is still valid SQL: running it would answer a _different_ result wearing this sheet's name; the toast names what the seal erased. [014; `docs/telas.md`]
12. **Two exports, two truths.** The list exports what is **saved** (from a list there is no screen to trust); the sheet exports what is **on screen** (unsaved edits, filter, visible columns, order, sort). Neither exports hidden columns. [`csv.ts`]
13. **Out in the clear only through the save dialog**, with the Consulta's CSV rules (`;`, CRLF, BOM, formula guard). [`csv.ts`; `lib.rs` `export_csv`]
14. **Removing rows asks in place and is undoable until Salvar**; Salvar prunes for good. [014 § 6, § 6a]
15. **Dirtiness is a comparison with the file**, not a flag, so no gesture can forget to raise it. Filter, page, ticks and density are not edits. [page]
16. **A save is validated at the boundary**: a non-blank title, rows as wide as the header, a complete column order, hidden/sorted/sized columns that exist, and an id this side minted. [`sheets.rs`]
17. **Order, hidden columns, widths and sort are stored by the column's original position** — the header is frozen at save, so positions stay stable. [026, decision 2]
18. **There is no bin.** Delete asks inline and says so. [014]

## Must survive the redesign

- One line per sheet: name, redacted SQL, counts in their own column, "alterado em", and pin / export / delete.
- Pinned first, and stated twice (not by a tag alone).
- The three empties; the lost key as a loud sentence, never an empty list.
- The footer "Relatório é foto, não consulta viva…" on the list; the sealed note on the sheet.
- Delete asks inline and says there is no bin.
- On the sheet: the name editable in place with a visible affordance; the "cifrado" badge (true of every sheet); the file-facts line; Excluir behind a menu; Rodar de novo never runs and warns about `'…'`; Salvar enabled only when there is something to save.
- The two exports with their different scopes, and the toast saying the file leaves without cipher.
- The grid surface's behaviours ([05-components](../05-components.md)).

## Free to redesign

- The line's internal layout, icons, the tag and badge styling, the segmented tabs, the order control's shape.
- Where the footer note and the sealed note sit (keep them present).
- The header's button arrangement — with Excluir kept away from Salvar.
- Loading and failure drawings (today there are none for loading).

## Sample content for mockups

Fictitious ([06-sample-data](../06-sample-data.md)); table and column names illustrative.

List — "Todos 3" · "Fixados 1" · "3 recortes · 41 linhas guardadas":

| Title                               | Second line (redacted)                                                                                                                                       | Counts             | alterado em         | Pinned |
| ----------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------ | ------------------- | ------ |
| Consumo OPME Vale Verde — semana 36 | `SELECT a.cd_atendimento, a.nm_paciente, i.lote, i.qtde FROM Atendimento a JOIN ItemConsumo i ON i.cd_atendimento = a.cd_atendimento WHERE a.cd_local = '…'` | 21 linhas · 9 col. | 09/09/2026 16:22:05 | yes    |
| Pedidos AMPARO SAÚDE — setembro     | `SELECT p.cd_pedido, p.cd_convenio, p.dt_emissao FROM Pedido p WHERE p.cd_convenio IN ('…', '…')`                                                            | 14 linhas · 7 col. | 08/09/2026 11:03:40 | no     |
| Lotes do parafuso cortical PC35-24T | `SELECT lote, qtde, cd_atendimento FROM ItemConsumo WHERE referencia = '…'`                                                                                  | 6 linhas · 4 col.  | 05/09/2026 09:47:12 | no     |

Sheet — name "Consumo OPME Vale Verde — semana 36"; "salvo em 09/09/2026 16:22:05 · 21 linhas · 9 colunas"; "Colunas 7 de 9"; after two removals the chip "2 linhas removidas" and Salvar lit. Rows as in the Consulta sample (MARTA LEMOS FERREIRA, JOÃO CARLOS PRADO, LUÍSA ANDRADE REIS, PEDRO HENRIQUE SALLES; lots 0000451288, A7K2219, C24B0917, 0000902211).

Rodar de novo toast, when the stored SQL carries `'…'`: the sentence quoted in Actions.

## Design opportunities

1. **Make dirtiness visible, and ask before discarding it.** Only the enabled Salvar says a sheet has unsaved edits, and the back arrow, the sidebar and Rodar de novo all drop them without a word. A chip ("alterações não salvas") and a question on leaving would protect the work.
2. **Show the photograph's date.** The moment the Everest answered is stored with the sheet but never drawn; the only date is "salvo em", which moves with every re-save. The footer promises "o resultado do dia em que você salvou" — true only for a sheet never re-saved. "Dados de 02/09/2026" beside "salvo em" would make the photograph honest.
3. **Refresh the title block after Salvar.** The line keeps the stamp and the row count it loaded until the sheet is reopened, contradicting "the counts are the file's".
4. **Show the source question on the sheet.** Only the list shows the redacted SQL; the sheet's own note talks about it, but the screen never draws it.
5. **Loading signs.** Both screens draw nothing while the files are decrypted.
6. **Errors that stay where they happened.** A failed pin replaces the whole list with a red sentence; a failed delete flashes and vanishes. A line-level error would keep the list and the reason.
7. **One confirmation idiom.** Deleting a sheet uses an amber strip; removing rows uses the in-place swap of the selection bar. Two shapes for one idea ([05-components](../05-components.md), _ConfirmButton_).
8. **Say what "9 col." counts.** It includes hidden columns; "9 col. (7 à vista)" would match what opening the sheet shows.
9. **One number style and singular forms.** "41 linhas guardadas" becomes "1204", not "1.204", at scale; "1 linhas".
10. **The filter placeholder over-promises.** "Filtrar em todas as colunas…" skips hidden columns by rule; "nas colunas à vista" would be true.
11. **Rodar de novo lands on a mismatched Consulta.** The editor gets this sheet's SQL while the answer pane still shows whatever answer was there; and any unsaved text in the Consulta's editor is replaced silently. Highlighting the `'…'` to retype would turn the warning into a guide.
12. **A sign of life on exports** — only the disabled state marks the time the dialog is open.
13. **"Novo recorte" names its destination poorly** — it opens the Consulta, where a cut starts as a query.

## Open questions

1. Which date identifies a sheet on screen: when the Everest answered, the first save, or the last save?
2. Should "salvo em" and the counts update right after a save on the sheet?
3. Should leaving a dirty sheet ask?
4. Should hidden columns ever be exportable (they are kept in the file)?
5. Should deleting a sheet give feedback, or an undo, given there is no bin?
6. Is a sheet with no rows a valid thing to keep (the save accepts it)?
