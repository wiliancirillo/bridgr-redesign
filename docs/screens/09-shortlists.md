# Listas — `/shortlists` and `/shortlists/[id]`

> Where: sidebar › Fora da esteira › Catálogos › Listas (indented sub-item; the sub-list folds, see [shell](../04-shell.md)); `/shortlists/[id]` opens from a line of Listas · Status: built · Code: `src/routes/shortlists/+page.svelte`, `src/routes/shortlists/[id]/+page.svelte`, `src/lib/components/SavedList.svelte`, `src/lib/saved-list.ts`, `src/lib/components/GridSurface.svelte`, `src/lib/components/ConfirmButton.svelte`, `src/lib/catalog-columns.ts`, `src/lib/catalogs.svelte.ts`, `src/lib/sheet.ts`, `src/lib/csv.ts`, `src/lib/ipc.ts`, `src-tauri/src/shortlists.rs`, `src-tauri/src/vault.rs`, `src-tauri/src/seal.rs`, `src/lib/i18n/locales/pt-BR.ts` (`shortlists`, `sheets`, `catalogs`, `query`)

## The question

"Que recortes de catálogo eu guardei?" (the subtitle of Listas). One list
answers: *which items of this catalog did I keep, and what does the ERP say
about them today?*

## Where it sits in the work

- A **list** is a cut of one catalog: the codes of the rows the operator
  ticked in [Catálogos](08-catalogs.md), plus how the grid was arranged. It is
  to Catálogos what [Relatórios](07-sheets.md) is to the Consulta — with the
  opposite nature: a report is a **photograph** of a query result; a list is
  **live**. It stores codes, never rows, and every time it opens it looks the
  codes up in the catalog's mirror **as the mirror is now**, so a description
  the ERP changed shows through.
- It is also where "editing a catalog" lives: the catalog grid has no row
  removal, and a list is the allow-list of codes the operator kept, in its own
  sealed file, which a new collection never touches.
- **Born only in Catálogos**, from a selection (Salvar como lista). The
  "Nova lista" button here leads to Catálogos; there is no other way to create
  one.
- A list needs its catalog's mirror to show anything: a list of a catalog never
  collected on this machine is intact (it holds the codes) but has nothing to
  resolve them against.

## Anatomy — Listas (`/shortlists`)

The shape is the saved-documents screen shared with Relatórios (SavedList, see
[components](../05-components.md)); this screen supplies the nouns, the second
line, the counts and the footer.

1. **Header**: title "Listas"; subtitle "Que recortes de catálogo eu
   guardei?"; at the right, the only primary button, **[+ Nova lista]** — it
   leads **out**, to Catálogos, because a list is cut where the data is.
2. **Tools row** (only when at least one list exists):
   - Filter (funnel, placeholder "Filtrar por nome ou catálogo…", clear ✕ while
     text is typed, hint "Ctrl ⇧ K").
   - Segmented pair "Todos {n}" / "Fixados {n}".
   - Spacer · summary "{n} listas · {m} itens guardados".
   - Order button (sort icon + the current order's name + caret; its accessible
     label is "Ordenar relatórios" — Relatórios' wording, see Open questions) →
     menu "Mais recentes" / "Nome (A–Z)" / "Mais itens", with a check on the
     current one. "Mais recentes" on entry.
3. **Lines** — one line per list, stacked (not cards):
   - The line body is one button that opens the list: `ph-list-checks` mark
     (accent when pinned) · **title** (ellipsis) + tag "FIXADO" (text "fixado")
     when pinned · second line: the **catalog's name** in monospace (e.g.
     "Materiais") · right column: "{n} itens" over "alterado em {dd/mm/aaaa
     hh:mm:ss}".
   - Actions at the line's end: **pin** (push-pin, filled when pinned; tooltip
     "Fixar no topo" / "Desafixar") and **trash** (tooltip "Excluir relatório" —
     Relatórios' wording). No export or other action per line.
   - A pinned line also has an accent-tinted border.
   - **Inline delete confirmation**, right under the line, amber: "Excluir esta
     lista? Os itens continuam no catálogo — some só a lista." · [Manter] ·
     [Excluir]. One line asks at a time.
4. **Footer** (when lists exist and nothing failed): lock icon + "Lista é viva,
   não é foto: ela guarda os códigos e mostra o que o espelho diz hoje.
   Recoletar o catálogo não a apaga."

## Anatomy — one list (`/shortlists/[id]`)

1. **Header row**:
   - ← back arrow to Listas (accessible label "Voltar aos relatórios" —
     Relatórios' wording).
   - **Title** as an inline text field that reads as a heading until hovered or
     focused (placeholder "Nome da lista") · pencil button (tooltip "Renomear o
     relatório" — Relatórios' wording; it focuses and selects the title) ·
     badge **"viva"** with a ↻ icon, in accent **[functional]**.
   - Meta line under the title: "salvo em {dd/mm/aaaa hh:mm:ss} · {n} itens ·
     de {Catálogo}".
   - Right: [Copiar tudo] · [Exportar CSV] · **[Salvar]** (accent, floppy icon;
     "Salvando…" in flight) · [⋮] (tooltip "Mais ações") → menu with one item,
     "Excluir relatório" (trash icon, danger on hover — Relatórios' wording).
   - Until the file answers, the header holds only the back arrow.
2. **Body**, top to bottom:
   - Load refusal: the refusal text (danger) and a link "Voltar aos relatórios".
   - Delete confirmation strip (amber), when asked: "Excluir esta lista? Os
     itens continuam no catálogo — some só a lista." · [Manter] · [Excluir].
   - Save refusal line (danger), when a save failed.
   - Then exactly one of:

     | When | Drawn |
     | ---- | ----- |
     | the list's catalog is not known to have been collected on this machine | empty state `ph-cloud-arrow-down`: "Este catálogo ainda não foi lido nesta máquina" / "A lista guarda os códigos e está intacta — colete o catálogo em Catálogos para ver as linhas." |
     | collected, mirror not yet in memory | turning `ph-circle-notch` + "Abrindo o espelho…" |
     | mirror in memory | the three blocks below + grid + live note |

   - **Newer-mirror banner** (accent tint — news, not a problem): ↻ icon + "O
     catálogo foi recoletado. Esta lista ainda mostra a leitura anterior." —
     plus " Salve ou descarte suas alterações para recarregar." while the draft
     has unsaved changes — and **[Recarregar]** (disabled while the draft has
     unsaved changes).
   - **Missing-codes banner** (amber — a human decides) **[functional]**:
     warning icon · bold "{n} item(ns) não estão mais no espelho" · " — Podem
     ter saído do ERP, ou o catálogo ainda não foi recoletado. Nada foi apagado
     da lista." · a monospace line with the codes, at most 12, joined by ", ",
     then " e mais {k}" · at the right, on the first line: **[Tirar as que
     sumiram]** (broom icon, danger tone), which asks in place "Tirar as {n} que
     o espelho não achou?" · [Manter] · [Tirar].
   - **Grid surface** ([components](../05-components.md)): filter (placeholder
     "Buscar por código, nome ou referência…", "Ctrl ⇧ K"), count "{n} itens",
     Colunas, density pair, page size; selection bar "1 linha selecionada" /
     "{n} linhas selecionadas" · "Marcar as {n} que o filtro achou" (when it
     applies) · **[Tirar da lista ({n})]** (minus-circle icon, danger tone) ·
     [Exportar seleção] · [Limpar]; the grid; the pager with, at its right end,
     "ordenado por {Coluna} ↑" or "↓" in monospace (empty when unsorted).
   - **Live note** under the grid: "Lista é viva, não é foto: ela guarda os
     códigos e mostra o que o espelho diz hoje. Recoletar o catálogo não a
     apaga."

## Data shown

### A line in Listas

| Field | Meaning | Presentation | Example |
| ----- | ------- | ------------ | ------- |
| Title | the list's name | as typed, trimmed | Parafusos e placas — Vale Verde |
| Catalog | which catalog the codes belong to | the catalog's on-screen name, monospace, second line | Materiais |
| "{n} itens" | codes stored in the list — **including** codes the mirror no longer answers | no thousands dot | 4 itens |
| "alterado em …" | the last **save** of the list; pinning does not move it | `dd/mm/aaaa hh:mm:ss` | alterado em 08/09/2026 16:05:12 |
| "FIXADO" | the operator pinned it | tag + accent mark + accent border | — |

**Counters in the tools row**: "Todos {n}" = every list; "Fixados {n}" =
pinned lists (both ignore the filter). Summary "{n} listas · {m} itens
guardados": lists, and the sum of stored codes over all lists (ignores the
filter and the tab; no thousands dot).

**Order**: pinned lines always first; then "Mais recentes" (last save, newest
first), "Nome (A–Z)" (pt-BR alphabetical) or "Mais itens" (most codes first).
Filter, tab and order are not remembered between visits.

### Inside a list

- **Rows**: one per stored code, in the stored order (the order they were on
  screen when saved), then arranged by the list's saved sort, if any.
- **Columns**: the list's catalog's columns — the same set Catálogos draws for
  that catalog ([Catálogos › What each catalog holds](08-catalogs.md#what-each-catalog-holds)),
  with the list's saved order, hidden columns, widths and sort. A column the
  catalog gained after the list was saved appears at the end, visible.
- **A resolved row** shows the catalog's values as the mirror has them now,
  with the same field rules as Catálogos (codes exactly as stored, text
  uppercase, Situação "ativo" / "inativo" / "—").
- **A code the mirror does not answer** **[functional]** still gets a row:
  Código = the stored code; Descrição = "não está no espelho"; every other cell
  empty (muted "—", tooltip "célula vazia"); the row is tinted amber with an
  amber bar on its first cell; it stays tickable (a ticked row wears the accent
  tint over the amber).

| Number | Counts |
| ------ | ------ |
| Meta "{n} itens" | codes in the file **as it was when the screen opened** — it does not change with the draft, and it does not refresh after Salvar (the next visit shows the new value); the same holds for "salvo em" |
| Grid "{n} itens" | draft rows matching the filter (no thousands dot) |
| Banner "{n} item(ns) …" | draft codes the mirror on screen does not answer; recounted as the draft changes |
| "Tirar da lista ({n})" / selection label | ticked rows |

## States

### Listas

| State | Trigger | Drawn |
| ----- | ------- | ----- |
| Not asked yet | before the first answer | header only; the body is blank — no "nenhuma lista" before anyone asked |
| No lists | the answer is empty | empty state `ph-list-checks`: "Nenhuma lista salva" / "Marque linhas em um catálogo e salve — a lista fica aqui, e continua olhando o espelho de hoje."; no tools, no footer |
| Loaded | — | tools, lines, footer |
| Filter found nothing | filter typed | empty state (magnifier): "Nada com o que você digitou" / "O filtro olha o nome e o catálogo. Talvez a lista tenha outro nome — ou ainda não exista." |
| Fixados with nothing pinned | tab Fixados, filter empty | empty state (magnifier): "Nenhum relatório fixado" / "Fixe um relatório pelo alfinete da lista para ele subir ao topo e ficar aqui." (Relatórios' wording) |
| Refused | the lists could not be read | the refusal text replaces everything under the header (no tools, no footer), e.g. "selado: a chave do lista sumiu do chaveiro — os arquivos continuam, ilegíveis" or "selado: a chave do lista no chaveiro está corrompida" (both as the current build words them). ⚠️ Files present with the key gone is **loss**, never an empty list |
| One file unreadable | a single sealed file does not open | that list is left out of the lines, and nothing says so; the healthy ones show |
| Pin failed | — | the refusal text replaces the lines until the screen re-reads |
| Delete failed | — | the refusal text shows, and the list re-reads right away, which clears it |

### One list

| State | Trigger | Drawn |
| ----- | ------- | ----- |
| Loading the file | — | header with only ←; blank body |
| File refused | — | refusal + "Voltar aos relatórios". Texts: "selado: esta lista não existe mais", "selado: o arquivo da lista está ilegível — o selo não confere", "selado: o arquivo do lista está ilegível — o selo não confere", "selado: a chave do lista sumiu do chaveiro — os arquivos continuam, ilegíveis", "the shortlist was refused: the id is not one this side minted" (English) |
| Catalog never collected | no mirror for this catalog on this machine | full header; the "Este catálogo ainda não foi lido nesta máquina" empty state. The same sentence is drawn while the app has not yet heard which catalogs exist (first list opened in an app session) and if that question fails |
| Opening | mirror on disk, not yet in memory | "Abrindo o espelho…". If the mirror then refuses to open, this sign stays: the refusal is not drawn on this screen |
| All codes resolve | — | no amber banner; plain rows |
| Some codes missing | — | amber banner with count and codes; flagged rows |
| Every code missing | — | banner lists the first 12 codes + " e mais {k}"; every row flagged |
| A newer mirror arrived | a collection of this catalog finished while the list was open | ⚠️ nothing changes under the operator: rows keep resolving against the mirror they were drawn from; the accent banner offers [Recarregar] |
| Unsaved changes | title edited, codes removed, columns/sort/widths changed | [Salvar] enabled; the newer-mirror banner (if any) adds "Salve ou descarte suas alterações para recarregar." and disables Recarregar. Nothing else marks the draft, and leaving (←, sidebar) drops it without a question |
| Salvar disabled | nothing changed, a save running, blank title, or no codes left | the button is grey; no reason is written |
| Saving / saved | — | "Salvando…"; then toast "Lista salva." and Salvar goes grey |
| Save refused | — | the refusal line at the top of the body (e.g. a wallet refusal) |
| Filter found nothing | — | inside the box, header row + "Nada com o que você digitou" |
| Copy/export with no mirror in memory | never collected or opening | Copiar tudo and Exportar CSV stay available; what they write is every stored code with "não está no espelho" in the description column |

## Actions

### Listas

| Gesture | Where | Preconditions | Effect | Confirmation | Reversible | Feedback | Shortcut |
| ------- | ----- | ------------- | ------ | ------------ | ---------- | -------- | -------- |
| Nova lista | header | — | goes to Catálogos (always on Materiais) | — | — | — | — |
| Open a list | line body | — | goes to `/shortlists/{id}` | — | — | — | — |
| Pin / unpin | pin button | — | saved at once; the line jumps to (or leaves) the top; "alterado em" does not move | no | yes | tag, mark, border | — |
| Delete | trash | — | inline question under the line → [Excluir] deletes the sealed file for good (no bin); the items stay in the catalog | inline, in place | no | the line disappears; no toast | — |
| Filter | tools row | — | title or catalog name, case-insensitive substring | — | clear ✕ | lines update | Ctrl ⇧ K |
| Tab, order | tools row | — | Todos / Fixados; Mais recentes / Nome (A–Z) / Mais itens | — | yes | — | — |

### One list

| Gesture | Where | Preconditions / disabled reason | Effect | Confirmation | Reversible | Feedback | Shortcut |
| ------- | ----- | ------------------------------- | ------ | ------------ | ---------- | -------- | -------- |
| Rename | title field, pencil | — | edits the draft title | — | until Salvar | Salvar enabled | — |
| Tick rows | grid | — | as in Catálogos: tick column, header tick for the page, Shift+click range, Ctrl/⌘+click toggle, "Marcar as {n} que o filtro achou", [Limpar] | — | yes | selection bar | — |
| Tirar da lista ({n}) | selection bar | ≥ 1 ticked | removes the ticked codes **from the draft list**, never from the catalog; clears the selection | in place: "Tirar {n} da lista? Os itens continuam no catálogo." · [Manter] · [Tirar]. The question withdraws itself if the ticked count changes while it is asked | until Salvar (leave without saving) | rows disappear; Salvar enabled | — |
| Tirar as que sumiram | missing-codes banner | ≥ 1 code missing | removes exactly the codes the mirror on screen does not answer, **recounted at the moment of the click**, from the draft | in place: "Tirar as {n} que o espelho não achou?" · [Manter] · [Tirar] | until Salvar | banner and flagged rows go; Salvar enabled | — |
| Recarregar | newer-mirror banner | no unsaved changes (else disabled, and the banner says why) | resolves the codes against the newest mirror; missing codes are recounted | no | — | rows and banner update | — |
| Salvar | header | a change exists, title not blank, ≥ 1 code, no save running | writes title, codes, column order, hidden columns, sort and widths; keeps the pin and the creation date | no | by editing again | toast "Lista salva." | — |
| Copiar tudo | header | — | header + filtered rows, tab-separated, to the clipboard | no | — | label "Copiado" ~1.6 s | — |
| Exportar CSV | header | not while an export dialog is open | save dialog "Exportar o recorte como CSV" (filter "Planilha CSV"), name from the current title, e.g. "Parafusos-e-placas-Vale-Verde.csv"; filtered rows, visible columns, screen order | the save dialog | — | "CSV exportado — o arquivo sai do app sem cifra." / "Não consegui gravar o CSV: {mensagem}" | — |
| Exportar seleção | selection bar | ≥ 1 ticked | same, ticked rows, "…-selecao.csv" | the save dialog | — | same | — |
| Columns, sort, resize, page size | grid tools | — | as in [components](../05-components.md); order, hidden, sort and widths are **part of the draft** (saved with Salvar, not automatically); density is the app-wide preference; page size is not saved | — | — | Salvar enabled | — |
| Delete the list | ⋮ → "Excluir relatório" | — | strip at the top of the body → [Excluir] deletes the file and returns to Listas | in place | no | no toast; refusal line on failure | Esc closes the menu and the strip |
| Back | ← | — | returns to Listas; an unsaved draft is dropped | no | — | — | — |

## Business rules

1. **A list is live, and that is the whole decision.** The file keeps the item
   codes and the column cut, never the rows; opening it resolves the codes
   against the mirror as it is now. A report keeps rows because a query result
   cannot be reproduced; a catalog row can, and freezing it would build a
   second truth that disagrees in silence. (decision 016 § 1, `shortlists.rs`)
2. **A code the mirror no longer answers is said out loud and stays.** It gets
   a row that says "não está no espelho" in the description's place (a blank
   row would read as an item without a description), the tint only helps find
   it, the banner counts and lists the codes (capped at 12), and nothing is
   removed behind the operator's back. (decision 016 § 3, § 3b)
3. **The reason is not guessed.** The banner names two causes; the likeliest
   third — the catalog's SELECT was narrowed in Catálogos — is not named, and
   the row states no cause: the app cannot know which happened.
   (decision 016 § 3b)
4. **A catalog never collected has its own sentence**, different from an empty
   list and from "everything missing": the list is intact, the mirror is what
   is missing. (decision 016 § 3)
5. **Nothing swaps under an open list.** The screen keeps the mirror it
   resolved against and **offers** the newer one; the offer is refused while
   the draft has unsaved changes, and says why. A silent swap would turn an
   unsaved draft into a cut of a different mirror and change "3 sumiram" with
   nobody told. (decision 016, consequences)
6. **Removing is a draft edit**: nothing is written until Salvar, and removal
   never touches the catalog. "Tirar as que sumiram" recounts at the moment it
   runs, so a collection finishing elsewhere cannot make it remove codes that
   came back. (decision 016 § 2, consequences)
7. **A list is always a selection**: it is created only from ticked rows in
   Catálogos, and it cannot be saved empty (Salvar is grey with no codes; the
   other side refuses "a shortlist needs at least one item"). A whole catalog
   saved as a list would be a second mirror with a name on it.
   (decision 016 § 7, `shortlists.rs` `validate`)
8. **Codes are stored exactly as the ERP spelled them** — trimmed and
   de-duplicated, never re-cased — in the operator's order.
   (decision 016 § 6, `shortlists.rs` `clean_ids`)
9. **The cut is stored as positions in the catalog's fixed column list**, which
   only grows by appending; a column born later arrives visible, never
   withheld. (decision 015 § 8, decision 026 open decision 2, `catalog-columns.ts`)
10. **Sealed at rest, with its own key.** A missing key with files present is
    shown as a refusal, never as an empty screen. (decision 016 § 5, `SavedList.svelte`)
11. **Pinning is not an edit**: it does not move "alterado em" (the order
    Listas sorts by), and a re-save keeps the pin. (`shortlists.rs` `set_pinned`)
12. **Deletion is for good**, and it deletes only the list; the confirmation
    says so. (`SavedList.svelte`, `/shortlists/[id]`)
13. **Export leaves the seal**: CSV through a save dialog, at a path the
    operator picks each time. (`csv.ts`)
14. **Listas and Relatórios are one list screen** (SavedList) — same filter,
    tabs, order, pin and inline confirmation. (decision 016 § 8)

## Must survive the redesign

- "Live" as a visible fact of every list (today the "viva" badge and the live
  note), distinct from Relatórios' "photograph".
- Missing codes: a row per code with the sentence in the description's place,
  the tint as a secondary signal, the banner with the count and the codes, and
  a gesture that removes exactly those — asked in place, applied to the draft.
- The never-collected sentence, distinct from "empty" and from "all missing".
- No silent swap of the mirror under an open list; the offer to reload, and its
  refusal with a reason while there are unsaved changes.
- Removal takes codes out of the list only, and every confirmation says the
  items stay in the catalog.
- Removals and deletion ask **in place**, never in a modal over the rows; the
  question counts what is at stake and withdraws if the selection changes.
- Lists are created only from a selection in Catálogos.
- Codes shown exactly as stored.
- A key or file refusal is shown as a refusal, never as "Nenhuma lista salva".
- Before the first answer, Listas asserts nothing.
- Pinned lists rise to the top in every order.

## Free to redesign

- Lines versus another list form; where pin, trash and counts sit; the order
  menu's shape; the tabs' look.
- The badge, the banners and the empty states' look — keeping amber for "a
  human decides here" (missing codes, delete confirmation) and accent for news
  (newer mirror, "viva", selection).
- How the title is edited (inline field today).
- The footer's icon and placement.

## Sample content for mockups

**Listas** — summary "4 listas · 11 itens guardados":

| Title | Catalog | Count | Stamp | Pinned |
| ----- | ------- | ----- | ----- | ------ |
| Parafusos e placas — Vale Verde | Materiais | 4 itens | alterado em 08/09/2026 16:05:12 | yes |
| Médicos do centro cirúrgico | Médicos | 2 itens | alterado em 05/09/2026 10:21:40 | no |
| Convênios da semana 36 | Convênios | 3 itens | alterado em 03/09/2026 09:02:11 | no |
| Depósitos de consignado | Depósitos | 2 itens | alterado em 01/09/2026 14:47:03 | no |

**One list** — "Parafusos e placas — Vale Verde", meta "salvo em 08/09/2026
16:05:12 · 4 itens · de Materiais", one code missing:

| Código | Descrição | ANVISA | Unidade | Situação | Código alt. |
| ------ | --------- | ------ | ------- | -------- | ----------- |
| 10245 | PARAFUSO CORTICAL 3,5 MM X 24 MM TITÂNIO | 80000000011 | UN | ativo | FAB-3524 |
| 10311 | PLACA BLOQUEADA 3,5 MM 6 FUROS | 80000000012 | UN | ativo | FAB-3506 |
| 10877 | CIMENTO ÓSSEO 40 G | 80000000013 | UN | ativo | — |
| 10955 | não está no espelho *(amber row)* | — | — | — | — |

Banner: "**1 item(ns) não estão mais no espelho** — Podem ter saído do ERP, ou
o catálogo ainda não foi recoletado. Nada foi apagado da lista." · codes
"10955" · [Tirar as que sumiram]. Pager end: "ordenado por Descrição ↑".

**Convênios da semana 36**: 000388 · AMPARO SAÚDE · ativo; 000391 · AMPARO
SAÚDE · inativo; 000412 · VIDA PLENA SAÚDE · "—".

## Design opportunities

1. **Show which mirror the list is looking at** — the catalog's stamp ("espelho
   de 08/09/2026") beside "de Materiais". Observation: the list is live, but
   "today's mirror" may be weeks old, and this screen never says when it was
   read. Reason: the stamp-in-sight rule of Catálogos applies to what a list
   shows too.
2. **Link "de {Catálogo}" to Catálogos with that catalog chosen.** Reason:
   re-collecting is the answer to one of the banner's two causes, and today the
   door lands on Materiais.
3. **Offer a real "Descartar".** The banner asks to "Salve ou descarte suas
   alterações", but no discard gesture exists (leaving is the only discard, and
   it asks nothing). Reason: the sentence names a gesture the screen does not
   have.
4. **Say why Salvar is grey**, and when no codes are left, point to deleting
   the list. Reason: a grey button with no reason reads as a bug.
5. **Hold Copiar tudo / Exportar CSV until the mirror is in memory.** Today, in
   the never-collected and opening states, they write every code as "não está
   no espelho" — an absence nobody checked.
6. **Refresh the meta line after Salvar** (count and "salvo em"). Reason: it
   keeps showing the file as it was when opened.
7. **Say when a list could not be opened** on Listas (today an unreadable file
   is simply left out). Reason: a list that vanishes without a word looks
   deleted.
8. **Give the footer an icon that says "live"** — the lock is Relatórios'
   "sealed" note borrowed with the shape, and here the sentence is about
   liveness.
9. **Open the new list from Catálogos' "Lista salva." moment.** Reason: the
   list is born on one screen and found on another.

## Open questions

- Several labels on these screens are Relatórios' words, shared through the
  common component: "Excluir relatório" (trash tooltip on Listas and the ⋮
  item on a list), "Renomear o relatório", "Voltar aos relatórios", "Ordenar
  relatórios", "Nenhum relatório fixado" / "Fixe um relatório pelo alfinete da
  lista para ele subir ao topo e ficar aqui." Should the redesign draw list
  nouns? (A wording change is the operator's call.)
- The key and file refusals read "do lista" in some messages and "da lista" in
  others (both quoted above as rendered). Draw them as they are, or propose the
  pt-BR agreement?
- The grid filter's placeholder mentions "referência", which no catalog column
  carries today (same question as [Catálogos](08-catalogs.md#open-questions)).
- Should a list be reachable from its catalog in Catálogos (e.g. "listas deste
  catálogo")?
