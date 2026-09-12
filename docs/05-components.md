# Shared components

> Code: `src/lib/components/` — `Grid.svelte`, `GridSurface.svelte`, `ColumnsMenu.svelte`, `FilterField.svelte`, `SavedList.svelte`, `ConfirmButton.svelte`, `EmptyState.svelte`, `EmptyScreen.svelte`, `DatePicker.svelte`, `SqlEditor.svelte`; `src/lib/grid-view.svelte.ts`, `src/lib/saved-list.ts`, `src/lib/sheet.ts` (view rules), `src/lib/sorting.ts` (sort and alignment rules), `src/app.css` (checkbox, spinner). Decisions 001, 006 § 2, 013, 014, 016, 026.

## One grid, wrapped once

- **Every table in the app is the same component.** The table logic (row model, sorting) comes from a _headless_ library — `@tanstack/table-core`, pinned to its v8 line — while the markup and the CSS are the app's own. That is what lets the grid follow the theme tokens, and every theme the operator writes, with nothing to override. [001 § 2]
- **Around it, one shell and one state object.** `GridSurface` (tools row, selection bar, grid, pager) and `GridView` (filter, sort, columns, widths, paging, selection) serve all six grid screens. A screen brings only what its rows _mean_ — the noun in the count, the gestures on ticked rows, the per-row buttons — as props and snippets. There were three hand-written copies before, and they drifted in ways no single screen showed: **a fourth copy of a table is a defect, not a shortcut**. [014 § 8]
- **The same principle** governs the screen of saved documents (`SavedList`, shared by Relatórios and Listas) and the in-place confirmation (`ConfirmButton`).
- **Two coordinate spaces** matter to anyone drawing columns: a column's _original_ position (where the data was born — what files and saved layouts store) and its _projected_ position (after hiding and reordering — what the screen shows). Hiding and reordering happen before the grid; the grid draws exactly what it is given.
- **For the redesign**: redesign each component once and every screen inherits it. The screen files ([screens/](screens/)) describe only how each screen configures these pieces.

## Map

| Component              | What it is                                                                | Used by                                                                                                                         |
| ---------------------- | ------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| Grid                   | The table                                                                 | Inside GridSurface (six screens); directly in the Opmenexo order subtela (materials grid, quotation answer sheet)               |
| GridSurface + GridView | Tools row, selection bar, grid and pager — and the view state behind them | Consulta, Relatórios (one sheet), Catálogos, Listas (one list), Opmenexo, NFePackMonitor                                        |
| ColumnsMenu            | The "Colunas" popover                                                     | Inside GridSurface                                                                                                              |
| FilterField            | A filter box, owner of the screen's one `Ctrl ⇧ K`                        | GridSurface, SavedList, Logs, Avisos, Triagem; the Consulta's trail                                                             |
| SavedList + SavedItem  | A screen of saved documents                                               | Relatórios, Listas                                                                                                              |
| ConfirmButton          | The in-place question before a destructive act                            | Selection bars of Consulta, Relatórios (one sheet), Listas (one list); the Listas warning block; Opmenexo and its order subtela |
| EmptyState             | The in-content empty                                                      | GridSurface, SavedList, Consulta, Triagem, Revisão, Faturamento, Ignorados, Catálogos, Opmenexo, Listas (one list)              |
| EmptyScreen            | A route whose screen is not built                                         | The three future studies                                                                                                        |
| DatePicker             | The house calendar                                                        | Revisão (one bill) — the bill header's date fields                                                                              |
| SqlEditor              | The SQL editor                                                            | Consulta; the Catálogos SQL dialog                                                                                              |

## House primitives these components lean on

- **Checkbox** — drawn once for the whole app (`app.css`), never per screen and never left to GTK (WebKitGTK paints native controls in GTK's theme, not the app's). A 15 px square, small radius, border at 35 % ink (accent on hover); ticked = accent fill with a white check; indeterminate = accent fill with a white **minus**.
- **Sign of life** — the `girando` spinner (one turn per 1.1 s; static when the system asks for reduced motion). A long action shows it _before_ it answers.
- **Popovers** (page size, Colunas, the saved-list order, the sheet's ⋮) — surface fill, radius md, large shadow, opening under their button; a press outside or Escape closes them. Menus are drawn by the app, never native `<select>` popups (GTK would paint those too).
- **Bar button (`sel-act`)** — the small outlined button of the selection bar: accent outline at 40 %, accent-200 text, 11 px; a `danger` variant for destructive gestures.

## 1. Grid

**Purpose.** Draws a header row and rows of text cells — sortable, optionally selectable, optionally with per-row buttons or custom cells. It knows nothing about what its rows are.

**Anatomy** — one table that scrolls inside its own box, in both directions. Left to right:

| Part               | Drawing                                                      | Notes                                                                                                                       |
| ------------------ | ------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| Tick column        | 34 px; the house checkbox in the header cell and in each row | Only when the screen offers gestures on a selection                                                                         |
| Data columns       | Header cell + body cells                                     | —                                                                                                                           |
| Filler column      | Empty; takes the leftover width                              | Keeps real columns at their content width and the row buttons pinned right; never sortable, hideable, reordered or exported |
| Row-actions column | The row's own buttons, right-aligned, as narrow as they are  | Outside the table model — never sortable, hidden, reordered or exported                                                     |

- **Header cell** — 30 px tall, **frozen at the top** while scrolling. The column's name in uppercase monospace 10.5 px, tracking .06 em, 50 % ink, followed by a direction icon: two-way arrows at 35 % while unsorted, ↑ or ↓ at full strength once sorted (the name then turns accent-200). Hover: quiet fill. Tooltip **"Ordenar por <NOME>"**. A name longer than its column truncates with an ellipsis (the whole name is in the tooltip). A numeric column right-aligns its header too.
- **Resize grip** — an invisible 7 px strip on the right edge of each header cell; under the pointer or during a drag it shows as a 2 px accent line, with the column-resize cursor. Accessible name **"Redimensionar coluna"**, tooltip **"Arraste para redimensionar; duplo clique volta ao automático."** Drawn only when the screen listens (every GridSurface screen does).
- **Body cell** — 82 % ink, 11.5 px, padding ≈ 8 × 11 px, top-aligned, one line (spaces kept, never wrapped), a hairline under each row (none under the last). Width: sized to content between **44 px and 280 px**; a longer value ends in an ellipsis and the full value is the cell's tooltip. A dragged width replaces both bounds for that column.

**Cell voices**

| Voice                   | When                                                                                                                                                                                                                     | Drawing                                                                                                                                       |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------- |
| Plain text              | Default                                                                                                                                                                                                                  | 82 % ink. Font: Inter on every GridSurface screen; the grid's own default, monospace, shows in the order subtela's materials grid             |
| Code / number           | Every non-blank cell of the column, across the **whole** dataset, reads as a number (`1.234,56`, `1234`, `000412`). A column with no values is not numeric. **A date is never numeric** (`dd/mm/aaaa` is read as a date) | Right-aligned, monospace, tabular figures, accent-300                                                                                         |
| Identity (first column) | On by default                                                                                                                                                                                                            | Accent ink, **frozen to the left** while scrolling right, on its own surface fill. Off where the first column is prose (the order subtela)    |
| Empty                   | Blank cell                                                                                                                                                                                                               | "—" at 28 % ink, tooltip **"célula vazia"** — never "NULL": the ERP's export cannot tell empty from NULL, and saying NULL would invent a fact |
| Custom                  | The screen supplies a cell renderer                                                                                                                                                                                      | The quotation answer sheet draws inputs in its editable columns; every other cell stays text ([13-opmenexo](screens/13-opmenexo.md))          |

**Row states**

| State       | Drawing                                                                             | Notes                                                                                                                                                                         |
| ----------- | ----------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Hover       | Quiet fill across the row                                                           | —                                                                                                                                                                             |
| Ticked      | Accent at 22 % across the row + a 2 px accent bar inside the first cell's left edge | Wins over flagged and highlighted. Accent, never amber: a selection is a state, not an alert. It was 10 % once, and the operator could not tell a ticked row from a plain one |
| Flagged     | `--st-warn` at 9 % + a 2 px warn bar                                                | Only for a row that needs a human decision, and the row must also say why in words — the tint is never the only signal (e.g. a Listas item "não está no espelho")             |
| Highlighted | Accent at 8 % + a 2 px accent bar                                                   | "This row carries your change" — the answer sheet also writes "alterada" in the row                                                                                           |

**Density** — "confortável" (default) or "compacto": compact takes the rows' vertical padding down to 3 px; the header keeps its height (it is a landmark). The choice is a stored preference shared by every grid, separate from the app-wide density in Configurações — "more rows" and "a tighter app" are two different wishes. [006 § 2]

**Interactions**

| Gesture                         | Effect                                                                                                                                                                                                                                                   |
| ------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Click a header                  | Sort by that column. First click: ascending — or descending when the first row the grid holds has that cell empty (the library's automatic rule); second click: the other direction; third: unsorted. Verified in the installed library's code, not live |
| Shift + click a header          | The library adds a **second** sort key: two headers show arrows, but only the first key orders the rows, names the pager's "ordenado por" and is saved — a latent defect (verified in code, not live)                                                    |
| Sorting rules                   | Empty cells always last, in both directions; numbers as numbers (42 before 1024); dates as dates (15/12/2025 before 02/01/2026); text by pt-BR collation (accents and case handled, digits inside text in numeric order)                                 |
| Header checkbox                 | Ticks or unticks every row **of this page**; shows a minus when only some are ticked. Accessible names **"Marcar os desta página"** / **"Desmarcar os desta página"**; tooltip **"Marca só o que está nesta página."**                                   |
| Row checkbox                    | Toggles that row (accessible name "Marcar <chave>")                                                                                                                                                                                                      |
| Shift + click (checkbox or row) | A range from the last row touched to this one, within the page; the clicked row's new state applies to the whole range (the file-manager grammar)                                                                                                        |
| Ctrl/⌘ + click on a row         | Toggles that row                                                                                                                                                                                                                                         |
| Plain click on a row            | Nothing — the cells' text stays selectable for copying                                                                                                                                                                                                   |
| Drag a header edge              | The width follows the pointer and is saved on release; minimum 44 px                                                                                                                                                                                     |
| Double-click a header edge      | Back to the automatic width                                                                                                                                                                                                                              |

**Rules it carries**

- Selection is stored by row **key**, never by position — sorting would re-aim a position at another row. No key → no selection and no row buttons: a row that is not an entity cannot be ticked.
- The screen may hand rows already sorted and paged; the grid then only draws the arrows and reports the click — one sorter, never two.
- The alignment verdict may come from the screen (whole dataset), so a page turn never flips a column's alignment.

**Used by** — every GridSurface screen (§ 2); directly by the Opmenexo order subtela (a framed materials grid without the identity column; the quotation answer sheet with custom cells, highlighted rows and no ticks).

**Must keep** — the frozen header and the frozen identity column; content-sized columns between the caps, plus the filler (a 37-column `SELECT *` stays readable, a 5-column cut does not stretch); the numeric-as-code voice and "a date is not a number"; "—" for empty, never NULL; the half-ticked minus; ticked in the accent (never amber), clearly stronger than hover, with its bar; flagged in amber only for a human decision and always with words; empty cells last in both directions; the header checkbox's page scope and its tooltip; the Shift/Ctrl grammar with a free plain click; the resize grip visible only under the pointer; the grid's density separate from the app's.

**Free** — row height, header styling, the arrow glyphs, the grip's look, the hover colour, exact tint strengths (keeping ticked, flagged and highlighted distinct from each other and from hover), the body font except for codes and numbers.

## 2. GridSurface + GridView

**Purpose.** The whole looked-at table: the tools over it, the selection bar, the grid and the pager in one framed box. `GridView` is the state behind it and every rule derived from that state; both are identical on every grid screen.

**Anatomy**

1. **Tools row** — wraps onto more lines when the pane is narrow:
   - the **filter** (FilterField, primary: 320 px, "Ctrl ⇧ K");
   - a spacer;
   - the **count**, worded by the screen ("42 linhas" on the Consulta); a narrow screen may draw it on its own line under the tools instead (NFePackMonitor);
   - the screen's extra chip, if any ("3 linhas removidas");
   - **"Colunas"** (ColumnsMenu, § 3);
   - the **density pair** — two icon buttons in a quiet segmented well: a rows icon (**"Linhas confortáveis"**) and a dashed-list icon (**"Linhas compactas"**); the chosen one raised on the surface with a small shadow;
   - the **page size** — a button "50 ▾" and the label **"por página"**; a 74 px menu with 25 / 50 / 100 / 250, the current size in the accent.
2. **Result box** (framed, radius md):
   - **Selection bar** — only when the screen offers selection gestures and ≥ 1 row is ticked. Accent tint (11 %), an accent hairline under it, accent-200 text at 11.5 px. Contents: the screen's label ("3 linhas selecionadas") · [**"Marcar as 3.653 que o filtro achou"**] · the screen's buttons · a spacer · **"Limpar"**. A destructive button is marked `danger` and keeps its red **inside** the accent bar — the bar says "these are selected", the button says "this erases them".
   - **The grid** — or, when no row survives, the header row alone followed by an EmptyState (magnifying glass) with the screen's sentence and optional second line, **inside** the box, where the rows would be. The header stays to say the table is still there; the message must never fall below the fold (it once did, and read as "no message at all").
   - **Pager** (surface fill, hairline above): **"← Anterior"** · **"1–50 de 212"** · **"Próxima →"** (accent text buttons, greyed at the ends) · a spacer · the screen's extra at the right end ("ordenado por NOME ↑").

**States** — default; filtering (page back to 1); no match (header + empty state; pager "0–0 de 0"); rows ticked (bar); the select-all offer (only when every row on the page is ticked **and** the filter still holds unticked rows — the moment the page-sized gesture runs out — carrying the count it would tick); a popover open (Colunas or page size; a press outside or Escape closes it); narrow pane (the tools wrap).

**Rules `GridView` applies on every screen**

- **Filter**: case-insensitive substring over the **visible** columns only — a hidden column never answers a search the operator cannot see matching.
- **Sort**: one column; empty cells last; the rules of § 1. It follows its column by identity across hiding and reordering, and clears when its column is hidden (a grid sorted by an invisible column is a lie with an arrow on it).
- **Columns**: display order and the hidden set by original position; widths by original position. "Desfazer edições" restores order, visibility and widths (the sort stays), plus the screen's own edits (removed rows) in the same gesture.
- **Paging**: 25 / 50 / 100 / 250, default 50; the page clamps when rows shrink; typing in the filter or changing the size returns to page 1.
- **Selection**: by key, across **all** rows — not just the filtered ones, not just the page — so the bar's number is what an action acts on. "Marcar as N" adds to what is ticked (never replaces) and keeps the on-screen order.
- **Edited** = a column hidden, reordered or resized; the sort is not an edit.
- **A new dataset under the same view** (the Consulta's next answer) resets filter, page, sort, columns, widths and ticks.
- **Copy and export take the view as shown** — visible columns, their order, the sort — never the raw data.

**Used by**

| Screen                                           | What it adds                                                                                |
| ------------------------------------------------ | ------------------------------------------------------------------------------------------- |
| [Consulta](screens/06-query.md)                  | Ticks with "Excluir selecionadas" and "Exportar seleção"; removed-rows chip; "ordenado por" |
| [Relatórios — one sheet](screens/07-sheets.md)   | The same                                                                                    |
| [Catálogos](screens/08-catalogs.md)              | Ticks with "Salvar como lista" and more; no row removal                                     |
| [Listas — one list](screens/09-shortlists.md)    | Ticks with "Tirar da lista (N)"; flagged rows                                               |
| [Opmenexo](screens/13-opmenexo.md)               | Ticks, per-row buttons, flagged rows                                                        |
| [NFePackMonitor](screens/12-nfe-pack-monitor.md) | Per-row buttons; the count on its own line                                                  |

**Must keep** — filter, count, Colunas, density and page size together over the grid; the bar, the grid and the pager reading as one object in one frame; the bar in the accent (a state, not an alert) with the danger exception; the select-all offer appearing only when the page runs out, carrying its count; the no-match message inside the box with the header kept; the selection spanning all rows; one `Ctrl ⇧ K` per surface.

**Free** — the arrangement of the tools, the pager's form, icons, the menus' shape.

## 3. ColumnsMenu

**Purpose.** One popover for what is one question in the operator's head: which columns, and in what order.

**Anatomy**

- **Button**: a columns icon + **"Colunas"** + a badge **"7 de 9"** (visible of all — it says, permanently, both "some are hidden" and "there are more to choose from") + a caret (pointing up while open); tinted while open.
- **Popover** (250 px, right-aligned under the button):
  - the master row **"Selecionar tudo"** — a filled checked square when nothing is hidden, a minus square when some are (never an empty box: "nothing shown" cannot happen);
  - a divider, then the kicker **"MOSTRAR COLUNA"**;
  - one row per column **in display order** (the list scrolls beyond 320 px): a **grip** (six dots; grab cursor; 35 % ink, 60 % when the row is hovered; accessible name **"Reordenar (arraste, ou use as setas): <NOME>"**) and a **pick** button (a square mark — filled and accent when shown, empty at 30 % ink when hidden — and the column's name in mono 11 px, ellipsis, full name in the tooltip);
  - only while there is something to undo: a divider and **"↺ Desfazer edições"**.

**States** — closed; open; a column shown or hidden; the **last visible column**'s pick disabled (its mark in a faded accent); dragging (the carried row at 45 % opacity; a 2 px accent line above or below the row where it would land — a line, never a filled row); something to undo (footer present).

**Interactions**

| Gesture                            | Effect                                                                                                 |
| ---------------------------------- | ------------------------------------------------------------------------------------------------------ |
| Click a pick                       | Show or hide that column (never the last visible one)                                                  |
| Click "Selecionar tudo"            | Some hidden → show all; none hidden → hide all but the first column of the current order               |
| Drag a row by its grip             | Moves the column to the gap the line marks; a drop in its own gaps changes nothing                     |
| Focus a grip + ArrowUp / ArrowDown | Moves it one step — the keyboard's way in, since a drag is pointer-only                                |
| "Desfazer edições"                 | Order, visibility and widths back to how the data was born, plus the screen's own edits (removed rows) |
| Press outside / Escape             | Closes (handled by the surface)                                                                        |

**Rules** — the grid can never reach zero columns (an inexplicable void, and getting out would need this very menu); a hidden column keeps its place in the order, so it returns where it was; no native checkbox inside (GTK would paint it in its own theme); the footer is a button rather than a note because it is the recovery that makes row removal safe. The master row and the footer are recorded departures from the design (decision 006 § 2).

**Used by** — GridSurface only, hence every grid screen.

**Must keep** — the "N de M" badge; the master row's two states; the last column protected; the drag with a visible landing line, plus a keyboard alternative; "Desfazer edições" present exactly when there is something to undo; column names in mono.

**Free** — the popover's size and styling, the grip glyph, the mark glyphs.

## 4. FilterField

**Purpose.** A screen's filter box — and the owner of the screen's `Ctrl ⇧ K`.

**Anatomy** — one box (320 px wide, at least 30 px tall, surface fill, a divider border that turns accent while focused, radius md): a funnel icon; the input (12 px, the screen's placeholder); a small **×** (tooltip **"Limpar filtro"**) only while there is text; and, on the primary field only, the key hint **"Ctrl ⇧ K"** (9.5 px on a quiet fill).

**Interactions** — typing filters (the screen decides what the text reaches, and says so in its placeholder); × clears; **Ctrl + Shift + K** anywhere on the screen focuses the primary field and selects its text.

**Rules** — **the hint and the binding are one thing**: a drawn shortcut that focuses nothing asserts what nobody verified, and a working shortcut nobody advertises does not exist (two screens once shipped a filter with neither). **One `Ctrl ⇧ K` per screen**: a screen with two filters marks exactly one as primary — the Consulta's result filter owns it, and its trail filter draws no hint. Plain `Ctrl K` belongs to the global search in the titlebar ([04-shell](04-shell.md)). The × is on every filter.

**Used by** — primary: every GridSurface, SavedList (Relatórios, Listas), Logs, Avisos, Triagem (which stretches it across its column — see [02-triage](screens/02-triage.md)); non-primary: the Consulta's trail. When a screen has no primary field on screen (the Consulta before its first answer), the key does nothing.

**Must keep** — the one-key-per-screen rule with its visible hint on exactly that field; the clear ×.

**Free** — the width (320 px is the design's number), the icon, the hint's styling.

## 5. SavedList + SavedItem

**Purpose.** A screen of saved documents, one line per document. The caller brings what its documents mean — nouns, the second line, the counts, the footer; the shape, the filter, the tabs, the order, pinning and the inline confirmation belong to the component.

**One line (`SavedItem`)** — an id; a title; a quiet second line (a redacted SQL, a catalog's name); a count label for the right-hand column ("21 linhas · 9 col."); an amount (what the summary adds up and what the third order sorts by); the last-change stamp; pinned or not.

**Anatomy**

1. **Header** — the title with the screen's question under it; right: one primary button that leads **out**, to where documents are made ("+ Novo recorte").
2. **Tools row** (only when loaded with ≥ 1 document) — the primary filter; a segmented pair **"Todos N"** | **"Fixados N"**; a spacer; the caller's summary; the order button (sort icon, current order, caret) with a menu of three orders — **"Mais recentes"**, **"Nome (A–Z)"** and the caller's third ("Mais linhas") — the current one checked.
3. **List** — one line per document (not cards: names read down a column, numbers compare in their own column): an icon (accent when pinned) · the title (13.5 px, 500) + the tag **"fixado"** (uppercase) when pinned · the second line in mono with an ellipsis · a 150 px right column with the count label over **"alterado em <dd/mm/aaaa hh:mm:ss>"** · icon buttons: pin (**"Fixar no topo"** / **"Desafixar"**, filled when pinned), the caller's extra buttons, trash (**"Excluir relatório"**, danger hover). A pinned line also takes an accent-tinted border; any line's border tints on hover. Clicking the line opens the document.
4. **Inline confirmation** — directly under the line: an amber-bordered strip with the caller's sentence, **"Manter"** and **"Excluir"** (red on hover). One at a time.
5. **Footer** (≥ 1 document and no failure) — a lock icon + the caller's sentence about what a saved document is.

**States** — not yet loaded (header only, blank body — it never says "nothing saved" before it has asked, but it draws no sign of loading either); failure (the body is only the red sentence — a lost key is loss and must never look like an empty list); loaded and empty (EmptyState with the caller's icon, title and note; no tools, no footer); filter miss (magnifying glass, the caller's filtered sentence); pinned tab empty (**"Nenhum relatório fixado"** / **"Fixe um relatório pelo alfinete da lista para ele subir ao topo e ficar aqui."**); a delete being confirmed; the order menu open (closes on a press outside or Escape).

**Rules** — pinned lines always rise, whatever the order (a mark with no consequence is not a gesture); within that: newest change, name in pt-BR collation, or amount; three empties told apart (nothing saved / nothing pinned / nothing matching); the filter reaches only what the caller says it reaches; delete always asks, inline.

**Words the component fixes** — the tabs, the "fixado" tag, "alterado em", the pin tooltips, the order labels, the order button's accessible name (**"Ordenar relatórios"**), the trash tooltip (**"Excluir relatório"**), the pinned-tab empty and "Manter" / "Excluir" all come from the Relatórios vocabulary — so **Listas shows "relatório" in four places** (the trash tooltip, the order button's name, the pinned-tab empty's title and note). A defect to fix when it is redrawn.

**Used by** — [Relatórios](screens/07-sheets.md) and [Listas](screens/09-shortlists.md).

**Must keep** — one line per document with the numbers in their own column; pinned first and stated twice (border and tag, never the tag alone); the three empties; the failure shown, never swallowed; delete confirmed inline under its own line; the primary button leading out; the footer sentence saying what the artifact is.

**Free** — the line's internal layout, icons, tab and tag styling, the order control's shape.

## 6. ConfirmButton — and the amber strip

**Purpose.** A gesture that asks before it acts, in place.

**Anatomy and states**

- **Idle** — a bar button (the `sel-act` look, optional icon + label, e.g. "🗑 Excluir selecionadas (3)"); the `danger` variant takes a red outline and red ink and hovers to a red tint.
- **Asking** — the button is replaced, in the same spot, by a strip that wraps onto more lines if it needs to: the question (meant in a secondary ink — the token it names is not defined today, so it renders in the surrounding text colour), then the cancel answer and the confirm answer (small outlined buttons; the confirm one in red when `danger`, the cancel one never).
- **Disabled** applies to both the idle button and the confirm answer.

**Interactions** — click → asking; cancel → idle; confirm → idle, then the act. No keyboard shortcut of its own.

**Rules** — **never a modal**: a dialog over the grid would hide the very rows the question is about. **The question carries the count and dies when it changes**: the grid stays live behind it, and a question about 3 must never act on 4 — the strip collapses back into the button the moment its label changes. It exists because the operator overturned "row removal needs no question" once one click could remove hundreds of rows. [014 § 6a]

**Used by** — the selection bars of the Consulta ("Excluir 3 do resultado? Nada muda no Everest."), of one Relatórios sheet ("Excluir 3 deste relatório? Só some do arquivo ao salvar.") and of one Listas list ("Tirar da lista (N)"); standalone in the Listas warning block ("Tirar as que sumiram"); Opmenexo and its order subtela ([13-opmenexo](screens/13-opmenexo.md)).

**The other idiom — the amber strip.** Deleting a whole document asks with a hand-drawn strip instead: an amber border and tint (the house's "a human decides here"), the sentence, a spacer, **"Manter"** and **"Excluir"** (red only on hover). It appears under a SavedList line, at the top of one Relatórios sheet's body, and on Ignorados. Same idea, two drawings — a candidate for one idiom.

**Must keep** — in place, never modal; the question names what is at stake, with its count; collapse when the count changes; the act wears the danger tone, the retreat never does.

**Free** — the strip's shape and layout, and whether the two idioms become one.

## 7. EmptyState

**Purpose.** The one in-content empty: a built screen that has nothing to show right now — no rows after a filter, no bill in a pile, nothing saved yet. One shape everywhere, so a blank area never reads as a screen somebody forgot to build.

**Anatomy** — centred in the space it is given (padding 48 × 20 px): an icon (26 px, 30 % ink; a tray by default), a title (13 px, 500), an optional note (11.5 px, 55 % ink, about 52 characters wide, line height 1.5) and, rarely, an action (Opmenexo's unconfigured state carries a button to Configurações · Spook).

**Rules** — it names the absence and, when there is one, the gesture that fills it; each kind of absence has its own sentence (three on Relatórios, three on the Consulta); never a warning colour — absence is calm; distinct from EmptyScreen.

**Used by** — GridSurface (no match), SavedList (its three empties), Consulta (never asked; proven empty), Triagem, Revisão, Faturamento, Ignorados, Catálogos, Opmenexo, Listas (one list).

**Must keep** — one shape app-wide; one sentence per kind of absence; room for an action.

**Free** — the drawing entirely.

## 8. EmptyScreen

**Purpose.** What a route shows before its screen exists.

**Anatomy** — centred: the screen's name as a heading (23 px, 500), the screen's question under it (70 % ink, about 52 characters wide), and a quiet line **"Esta tela ainda não foi construída."** (11 px, 45 % ink); optional content below, for a screen that already has something real to show.

**Rule** — it states the **question**, not only the name: a blank route tells nobody where they are or what will be there, and the question is the sentence the screen must keep answering once it is built.

**Used by** — the three future-study routes ([14-future-studies](screens/14-future-studies.md)).

**Must keep** — name + question + the honest "not built" line.

**Free** — the drawing.

## 9. DatePicker

**Purpose.** The house calendar. The native `input[type=date]` is **forbidden**: WebKitGTK's calendar ignores the theme and does not close on a click outside.

**Anatomy**

- **Trigger** — a calendar icon button (13 px, 45 % ink, accent on hover); accessible name and tooltip **"Abrir o calendário"**. It sits beside the caller's date field; the field shows `dd/mm/aaaa` (or "—" when empty) and is edited as text on its own.
- **Popover** (252 px wide; surface, divider border, radius 10 px, medium shadow, 10 px padding), placed under the trigger, kept inside the window with an 8 px margin, and flipped above when there is no room below:
  - a bar with the month and year, capitalised (**"Setembro de 2026"**), and two accent caret buttons (**"Mês anterior"**, **"Mês seguinte"**);
  - the weekday row, **Sunday first**, from the locale ("dom seg ter qua qui sex sáb", periods stripped — exact rendering unverified);
  - a fixed **6 × 7** day grid, so the popover never changes height between months: days of the neighbouring months dimmed to 30 % ink; the selected day in an accent-22 % circle with accent-200 text; round hover.

**States** — closed; open on the value's month (today's month when the field is empty); a selected day marked when the value is a valid ISO date. No "today" marker; no minimum or maximum.

**Interactions** — the trigger opens and closes it; the carets change the month; a click on a day picks it (the popover closes and the caller receives the ISO date); it closes on Escape, on a click outside, on any scroll and on a window resize (a fixed box would otherwise float away from its field). Days are focusable buttons; there is no arrow-key navigation inside the grid.

**Rules** — ISO (`aaaa-mm-dd`) inside, `dd/mm/aaaa` on screen, four-digit years, converted in one place; the popover escapes the scrolling pane that holds the field (a calendar clipped in half was found live on a narrow window).

**Used by** — Revisão (one bill): the date fields of the bill header ([03-review](screens/03-review.md)).

**Must keep** — no native date control; themed; closes on a click outside and on Escape; always fully visible; ISO in, `dd/mm/aaaa` out.

**Free** — the whole drawing; Sunday-first (the design's choice); a today marker; keyboard navigation.

## 10. SqlEditor

A CodeMirror 6 editor painted entirely with the app's tokens — no library stylesheet, including the completion popup, the one piece of the library's DOM that reaches the screen — so it follows every theme. It validates nothing; it _shows_: syntax colours, comments painted as errors, non-ASCII characters marked in red. Used by the [Consulta](screens/06-query.md) (with the ERP map's completion and `Ctrl+Enter` to run — described there) and by the Catálogos SQL dialog, whose two editors (the page query and the count) show the four SQL hints under them as a lightbulb list — hints, never a gate ([08-catalogs](screens/08-catalogs.md)).

## Cross-component design opportunities

1. **One confirmation idiom.** ConfirmButton (swap in place) and the amber strip say the same thing in two drawings.
2. **Counts and plurals.** Several counts have no singular form ("1 linhas"), and numbers are written sometimes with a thousands dot ("3.653") and sometimes without ("3653"). One formatter for every count.
3. **SavedList's fixed words.** Make the trash tooltip, the order button's name and the pinned-tab empty the caller's, so Listas stops saying "relatório".
4. **Sorting by more than one column.** Either make Shift + click a real secondary sort (and name both keys in the pager) or refuse it; today it draws two arrows and applies one.
5. **Loading.** SavedList draws nothing while loading; an EmptyState-shaped "carregando" would keep "absence looks like absence" honest in both directions.
6. **Signs of life on exports.** Every CSV export only disables its button while the system save dialog is open.
7. **The secondary ink.** ConfirmButton asks for a secondary text token the themes do not define; decide its value.
8. **DatePicker.** A "today" mark, arrow-key navigation, and a clearer pairing between the typed field and its calendar.
