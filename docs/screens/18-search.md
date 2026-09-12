# Busca — the titlebar field and `/search`

> Where: the search field in the centre of the titlebar (`Ctrl K`); Enter opens `/search`, a screen with no sidebar item (the titlebar reads "Bridgr Pro · Busca") · Status: built · Code: `src/lib/components/Titlebar.svelte` (the search slot), `src/lib/search.svelte.ts`, `src/routes/search/+page.svelte`, `src/lib/nav.ts` (`OFF_NAV`, `screenAt`), `src/lib/format.ts` (`shownBytes`), `src/lib/i18n/locales/pt-BR.ts` (`search`, `logs.sources`, `triage.states`, `catalogs.name`)

## The question

"Onde está o que você procura — contas, arquivos, pacientes, códigos do ERP e logs" (`question.search`; the sentence is not drawn on the screen). One term, typed once, over everything this station holds, and from every answer a door to the screen where the thing lives.

## Where it sits in the work

A navigation tool, usable from any screen. The operator types a patient's name, an atendimento, a file name, an ERP code or a word from a log, and jumps to [Revisão](03-review.md), [Faturamento](04-billing.md), [Triagem](02-triage.md), [Catálogos](08-catalogs.md) or [Logs](11-logs.md). The search changes nothing and writes nothing to disk: the term and the results live in the app's memory until the next search or until the app closes. It is reached only from the titlebar field; no menu item or dashboard door leads to it.

## Anatomy

**A. The titlebar field** (the whole titlebar is in [04-shell](../04-shell.md)). It sits in the centre slot, between the brand and the bell: a magnifying-glass icon, the input with the placeholder "Buscar paciente, atendimento, arquivo ou código do ERP…", and a "Ctrl K" key hint that hides while the field has focus. Up to 430 px wide, quiet until focused (then an accent border). It is absent on the first-assembly screen and while the window is locked. The typed text stays in the field after Enter.

**B. The `/search` screen**, top to bottom:

1. **Header.** "Resultados para" followed by the term in a chip (monospace, accent ink, cut with an ellipsis past about 40 characters) and, while a scan runs, a spinning icon. Under it, once a scan has landed: "<N> ocorrências em contas, arquivos, pacientes, catálogos e logs · varredura de hh:mm". At the right: "Voltar para <screen of origin>" (arrow-u-up-left icon).
2. **Failure strips** (danger), one per source that refused: "<group> — <message>".
3. **Group chips**, a wrapping row of pills: "Tudo" (list icon) with the total, then "Contas", "Arquivos da doca", "Pacientes", "Códigos do ERP" and "Linhas de log", each with its count in a small monospace badge, zeros included. One chip is active (accent).
4. **Body** (scrolls; content at most 1000 px wide). For each group with at least one hit, in the fixed order above: a group header (small uppercase label, the count text, a hairline) and one card holding the result rows. A row is one button: a tone-coloured tile with the group's icon · the title (monospace for codes and log lines) · one meta line (cut with an ellipsis; its full text is the row's tooltip) · a tone-coloured "where" pill · an arrow.
5. **Footer note**, always drawn: a dashed box with an info icon reading "A busca olha a doca, as contas lidas, o espelho de catálogos e os logs. Ignorados e contas excluídas ficam de fora — abra Ignorados.", where "Ignorados" links to [Ignorados](05-discarded.md). **[functional]** the statement of what is searched and what stays out.

## Data shown

Matching is a plain, case-insensitive substring test, with **no accent folding and no ranking**: "joao" does not find "JOÃO". Results keep the order of their sources.

| Group (chip)          | What is searched                                                                                                                                | Title                                                                        | Meta                                                                         | "Where" pill                                                                                                                                                        | Tone                                                                                                                              | Icon                                         | Opens                                             |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- | ------------------------------------------------- |
| Contas: read accounts | each account in the reading results: the patient name, or the name of the file it was read from. Only accounts **not** already queued or worked | the patient                                                                  | `<adapter id> · <file name>`                                                 | "Revisão · a conferir", or "Revisão · não enviar" for an account marked not to send                                                                                 | amber when one of its items lacks a lot, otherwise accent                                                                         | receipt                                      | that file's review, `/review/<id>`                |
| Contas: billing rows  | each bill in the billing record: patient, atendimento or convênio                                                                               | "<atendimento> · <patient>" (the patient alone when there is no atendimento) | `<adapter id>`                                                               | "Faturamento · na fila do Fantasma" (waiting, not started) or "Faturamento"                                                                                         | red if the robot's run failed; amber if it needs investigation and is not finished; green for "Faturamento"; accent for "na fila" | receipt                                      | `/billing`                                        |
| Arquivos da doca      | each dock piece's file name; ignored pieces never come back                                                                                     | the file name                                                                | `<size> · <adapter id> · <state word>` (each only when known)                | "Triagem"                                                                                                                                                           | red for "falhou", amber for "sem cliente", neutral otherwise                                                                      | a PDF icon for `.pdf`, a paperclip otherwise | `/triage`                                         |
| Pacientes             | patient names gathered from **every** read account, including queued, worked and not-to-send ones                                               | the name                                                                     | "<N> conta(s) · <M> arquivo(s)"                                              | "Revisão"                                                                                                                                                           | accent                                                                                                                            | user                                         | `/review`                                         |
| Códigos do ERP        | every collected catalog mirror: code, description, alias, reference, ANVISA number, alternate code                                              | the ERP code, verbatim (monospace)                                           | `<description> · <reference>` (the ANVISA number when there is no reference) | the catalog's name: "Materiais", "Médicos", "Convênios", "Locais de uso", "Depósitos", "Comissionados", "Tipos de pedido", "Tabelas de preço", "Condições de venda" | neutral                                                                                                                           | barcode                                      | `/catalogs` (the screen, not that catalog or row) |
| Linhas de log         | each line of the five [Logs](11-logs.md) sources: the app journal as "event detail" (no stamp), and the NF-e monitor's **full** log             | "<source> · linha <N>" (monospace)                                           | the line, verbatim                                                           | the source's name: "Aplicativo", "MedLens · Serviço", "Spook · Serviço", "NFe Pack · Serviço", "Fantasma · Sessão"                                                  | neutral                                                                                                                           | terminal-window                              | `/logs?source=<source>`                           |

Dock state words: "sem cliente", "falhou", "aguarda extração", "extraindo…", "pronta". Sizes read "319 KB" or "2,1 MB".

Counts:

- Group header: "<N> ocorrência" / "<N> ocorrências"; when the group was cut, "mostrando 80 de <N>".
- The chip badges and the summary use the **full** totals (not cut), with a thousands separator ("1.482").
- The summary always says "ocorrências", even for one.

## States

| State                     | Trigger                                                                          | Drawn                                                                                                                                                        |
| ------------------------- | -------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Field absent              | first-assembly screen, or window locked                                          | no field                                                                                                                                                     |
| Field idle / focused      | —                                                                                | the "Ctrl K" hint / an accent border with the hint hidden                                                                                                    |
| Never searched            | `/search` reached with no scan in this session (e.g. through history navigation) | "Digite um termo e pressione Enter — a busca corre uma vez e diz quando correu."                                                                             |
| First scan running        | Enter                                                                            | the header's term plus the spinner; no summary; chips at 0; the body is empty except for the footer                                                          |
| Scan running again        | Enter with a new term while results are shown                                    | the header already shows the new term and the spinner; the previous results, counts and "varredura de" stamp stay until the new scan lands                   |
| Results                   | the scan landed                                                                  | the groups with hits and the stamped summary                                                                                                                 |
| No results                | the scan landed with a total of 0                                                | "Nada com “<term>”" and, under it, "A busca olha títulos, nomes, códigos e linhas de log — não o conteúdo dos documentos."                                   |
| A chip with no hits       | a group chip whose count is 0 is chosen                                          | the body is empty except for the footer (no sentence)                                                                                                        |
| A group cut               | more than 80 hits in a group                                                     | the group header reads "mostrando 80 de <N>"                                                                                                                 |
| Partial failure           | a source refused                                                                 | one strip per refusal, e.g. "Contas — MedLens: …"; the groups that answered are still drawn. A failing log source names only its group ("Linhas de log — …") |
| A catalog never collected | its mirror does not exist                                                        | it contributes nothing, silently: absence, not failure                                                                                                       |
| Narrow window             | —                                                                                | the chips wrap; there is no other rule                                                                                                                       |

## Actions

| Gesture                  | Where               | Precondition                                 | Effect                                                                                                               | Feedback                |
| ------------------------ | ------------------- | -------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- | ----------------------- |
| `Ctrl K` (without Shift) | anywhere            | not locked, not on the first-assembly screen | focuses and selects the titlebar field                                                                               | —                       |
| Enter                    | titlebar field      | the term is not blank after trimming         | remembers the term and the screen it was typed on (unless already on `/search`), starts the scan and opens `/search` | the spinner by the term |
| `Esc`                    | titlebar field      | —                                            | blurs the field (on `/search`, the same keypress also leaves the screen)                                             | —                       |
| `Esc`                    | `/search`, anywhere | —                                            | returns to the screen of origin (the Dashboard when none was recorded)                                               | —                       |
| "Voltar para <screen>"   | header              | —                                            | same as `Esc`                                                                                                        | —                       |
| Choose a chip            | chip row            | —                                            | shows one group (or "Tudo"); the choice is kept across the next scans                                                | the active chip         |
| Open a result            | a row               | —                                            | navigates to its door                                                                                                | —                       |
| "Ignorados"              | footer note         | —                                            | opens [Ignorados](05-discarded.md)                                                                                   | —                       |
| Search again             | Enter in the field  | —                                            | runs a new scan; the newest scan always wins over an older one still running                                         | a new stamp             |

No gesture asks for confirmation, and nothing is written.

## Business rules

1. **The gesture lives in a module, not in the screen.** Enter starts the scan and then navigates; a scan that finishes while no screen is mounted still lands its answer (decision 039).
2. **A scan is a snapshot and says when it ran** ("varredura de hh:mm"). Nothing watches or refreshes; Enter runs it again.
3. **Each source is asked on its own.** A refusal becomes a strip naming its group, and the other groups still answer. The read accounts come through MedLens, and MedLens being down must not hide the catalog codes (039; [02-invariants](../02-invariants.md): never one all-or-nothing call).
4. **No silent caps.** Each group keeps at most 80 hits but counts all of them, and its header says "mostrando 80 de N" (039).
5. **One bill, one row (the token model).** A queued or worked bill answers only as its Faturamento row, never also as a Revisão row (039).
6. **Ignored dock pieces stay out**, and the footer says so and opens Ignorados (039).
7. **ERP codes verbatim**, only trimmed: re-casing a code invents one the ERP does not have ([02-invariants](../02-invariants.md)).
8. **Log lines verbatim**, never interpreted.
9. **Plain matching, on purpose**: lowercase substring, no accent folding, no ranking. A clever navigation search would be a second matching engine to keep honest (039).
10. **A patient exists regardless of where the bill is.** The Pacientes group gathers every read account, keyed by the exact name.
11. **`Ctrl K` is the global search; `Ctrl ⇧ K` is each screen's own filter.** Two shortcuts, never merged (039; [05-components](../05-components.md) › FilterField).
12. **Titles, names, codes and log lines, never the documents' content** (PDF or XML). The no-results note says so.
13. **The journal never carries patient names** ([Logs](11-logs.md), rule 7), so a patient search finds no Aplicativo line.

## Must survive the redesign

- `Ctrl K` for this field; `Ctrl ⇧ K` left to the screens' filters.
- The snapshot stamp "varredura de hh:mm", and nothing that suggests live results.
- Refusals named per source, with the other groups still drawn.
- "mostrando 80 de N" on a cut group, and honest totals everywhere.
- One row per bill (the token model).
- The footer's statement of scope and exclusions, with the door to Ignorados.
- Codes and log lines verbatim, in monospace.
- The idle sentence before the first search: no results asserted before anyone asked.
- `Esc` and "Voltar para …" return to the screen of origin.
- Every result is a door; no gesture acts on a result in place.

## Free to redesign

The field's size and look (a command-palette overlay is acceptable if the shortcuts and the Enter-to-scan behaviour hold); chips, tabs or a side list for the groups; the row layout and the tone treatment. The group order is fixed today, but a different order could be argued (proposal).

## Sample content for mockups

Term `marta`, searched from Revisão, "varredura de 14:32":

- Header: "Resultados para `marta`" · "2 ocorrências em contas, arquivos, pacientes, catálogos e logs · varredura de 14:32" · "Voltar para Revisão".
- Chips: Tudo 2 · Contas 1 · Arquivos da doca 0 · Pacientes 1 · Códigos do ERP 0 · Linhas de log 0.
- Contas, "1 ocorrência": receipt tile in accent · "2026004517 · MARTA LEMOS FERREIRA" · `vale_verde` · the pill "Faturamento · na fila do Fantasma".
- Pacientes, "1 ocorrência": user tile · "MARTA LEMOS FERREIRA" · "1 conta · 1 arquivo" · the pill "Revisão".

Term `vale_verde`: in Arquivos da doca, "informe_vale_verde_2026-09-02.pdf" · "319 KB · vale_verde · pronta" · "Triagem"; in Contas, "JOÃO CARLOS PRADO" · "vale_verde · informe_vale_verde_2026-09-02.pdf" · "Revisão · a conferir" (amber: an item without its lot).

Term `PC35`: in Códigos do ERP, "0007310" · "PARAFUSO CORTICAL 3,5 MM X 24 MM TITÂNIO · PC35-24T" · "Materiais" (the ERP code is fictitious, in the cast's spirit).

A cut group: term `a` gives "Pacientes — mostrando 80 de 213".

A refusal: "Contas — MedLens: <the service's message>" above the groups that answered.

## Design opportunities

1. **Show which scan is on screen.** During a new scan the header already carries the new term over the previous results. Dim the old results, or keep the old term until the new answer lands.
2. **A sentence for an empty chip** ("nada em Códigos do ERP para …"). Today the body goes blank.
3. **Deeper doors.** A code hit could open its catalog at its row; a log hit could land on its line; a Faturamento hit could select its row.
4. **The client's name, not the adapter id**, in the meta lines ("Hospital Vale Verde" rather than `vale_verde`).
5. **Keyboard through the results** (↑/↓/Enter), so a search can be finished without the mouse.
6. **Say that matching is accent-sensitive**, near the field or in the no-results note, since patient names carry accents. Or propose accent folding, which needs a decision (see Open questions).
7. **A way past the cap** ("ver os 213"), or a hint to refine the term.
8. **A singular summary** ("1 ocorrência").
9. **Dim the zero-count chips** instead of drawing them like the others.
10. ⚠️ A caution for any "recent searches" idea: the terms are mostly patient names, so a saved history would be patient data at rest.

## Open questions

- Accent-insensitive matching: decision 039 chose plain matching. Does the placeholder's invitation to search for patients argue for folding?
- The footer says "contas excluídas ficam de fora", yet accounts marked "não enviar" appear as "Revisão · não enviar". Which accounts is that sentence about?
- Should the NF-e monitor's downloaded notes, or the OPMENEXO orders, become groups of their own?
