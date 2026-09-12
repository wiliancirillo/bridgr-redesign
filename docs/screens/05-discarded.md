# Ignorados — `/discarded`

> Where: sidebar › Fora da esteira › Ignorados (icon `ph-archive`); also linked from the global search's footer note · Status: built (12/08/2026) · Code: `src/routes/discarded/+page.svelte`, `src/lib/ipc.ts` (`DiscardedPiece`, `readDiscarded`, `restoreDiscarded`, `deleteDiscarded`), `src-tauri/src/dock.rs` (`discarded_pieces`, `un_ignore`, `excluded`), `src-tauri/src/lib.rs` (`read_discarded`, `restore_discarded`, `delete_discarded`), `src-tauri/src/delivery.rs` (`delete_piece`, `key_path`, `key_revision`, `file_revision`)

## The question

«O que saiu da doca?» (i18n `question.discarded`, drawn as the subtitle when nothing is set aside).

What did I set aside in Triagem — does any of it go back, or does it go for good?

## Where it sits in the work

Outside the conveyor: nothing here moves a bill. It is in the sidebar and not in Configurações because it is **work set aside**, not a preference.

- **In.** Pieces arrive only through [Triagem](02-triage.md)'s «Ignorar» (the focused piece, the selection bar, or `Esc`).
- **Out.**
  - «Restaurar» puts the piece back in Triagem's dock.
  - «Excluir» deletes its file for good — from the spool, or from the watched folder.
- **Reached from.** The sidebar. The global search's footer ([Busca](18-search.md)): «A busca olha a doca, as contas lidas, o espelho de catálogos e os logs. Ignorados e contas excluídas ficam de fora — abra Ignorados.», where «Ignorados» is a link here.
- **Counter.** None — no sidebar item carries a counter today.

## Anatomy

1. **Header** (full width, hairline below).
   - Title «Ignorados».
   - Subtitle (quiet, fixed height):
     - empty while the first read is pending or when it failed;
     - «O que saiu da doca?» when the list is empty;
     - «Trabalho posto de lado — 1 peça que saiu da doca» / «Trabalho posto de lado — {n} peças que saíram da doca».
   - No reload button, no filter, no toolbar.
2. **Content** (scrolls): one column of row cards, at most 920 px wide, 7 px apart.
3. **Row card** (surface, hairline border, rounded):
   - **Line.**
     - A 28 px tile with the kind icon: `ph-file-pdf` for a PDF (and anything unknown), `ph-image` for a photo, `ph-table` for an e-mail body.
     - Body: the file name (monospace, ellipsised) over the meta line (quiet).
     - At the right, «Restaurar» (`ph-arrow-u-up-left`; the icon turns into a spinner while this row's command runs) and «Excluir» (`ph-trash`, danger wash and danger border on hover, with a tooltip).
   - **Inline confirmation** — only on the row being deleted, under the line, inside the same card:
     - amber border and a faint amber wash, because amber means "a human decides here";
     - the confirmation sentence, a spacer, «Manter», then «Excluir» (danger only on hover of the act itself).

## Data shown

| Field       | Meaning                                                                                                            | Presentation                                                                                                                                                     | Example                             |
| ----------- | ------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------- |
| File name   | The attachment's name in the e-mail. For an e-mail body piece, the e-mail's subject. For a watched file, its name. | Monospace, ellipsised.                                                                                                                                           | «informe_vale_verde_2026-08-26.pdf» |
| Kind        | `pdf`, `imagem` or `corpo`.                                                                                        | Icon only.                                                                                                                                                       | —                                   |
| Ignored at  | When the operator ignored it.                                                                                      | «ignorada em dd/mm/aaaa · hh:mm». **Omitted entirely** when the decision predates the recorded instant: never an invented date.                                  | «ignorada em 02/09/2026 · 09:12»    |
| Size        | Bytes of the file.                                                                                                 | «812 B», «240 KB», «1,2 MB».                                                                                                                                     | «1,2 MB»                            |
| Source      | Where it came from, told three ways.                                                                               | An e-mail attachment reads «de {remetente}». An e-mail body piece reads «corpo do e-mail» (the body **is** the e-mail). A watched file reads «da pasta vigiada». | «de faturamento@valeverde.example»  |
| E-mail date | When the e-mail arrived.                                                                                           | **Not shown**; only used to order rows that carry no ignored-at.                                                                                                 | —                                   |

**Meta line.** The parts joined by « · »: [ignored at] · size · source.

**Order.** The newest decision first. A row without an instant is placed by its e-mail's date instead; ties are broken by an internal key.

**Counter.** The subtitle's {n} is the number of rows drawn: ignored pieces whose file still exists in the spool or the watched folder. A decision whose file has left the disk draws nothing and is not counted.

## States

- **Not asked yet** (the first read has not answered): the title only, an empty subtitle, blank content. The screen asserts nothing.
- **Read failed**: an empty subtitle, and a centred empty state with a warning icon (`ph-warning`) whose title is the error message itself (no note). A failure outranks "empty": a loss never dresses as absence.
- **Empty**: the subtitle «O que saiu da doca?» and an empty state (`ph-archive`): «Nada posto de lado», with the note «O que você ignorar na Triagem espera aqui — dá para restaurar ou excluir de vez.»
- **List**: the rows.
- **Busy** (a restore or a delete in flight on one row): that row's «Restaurar» icon spins (for a delete too), and **every** «Restaurar» and «Excluir» on the screen disables (dimmed) until the answer. The mark is drawn before the command leaves.
- **Confirming**: one row at a time. Opening another row's confirmation closes the previous one, and pressing «Restaurar» on any row closes it too.
- **After a success**: a toast, then the list is read again — the row disappears and the subtitle recounts.
- **Gesture failed**: a danger toast with the message (e.g. «{caminho}: {erro do sistema}» for a disk failure); the list is not re-read, and the row stays.
- **No live refresh.** The list is read when the screen opens and after each gesture; nothing polls.

## Actions

| Gesture (where)               | Precondition    | Effect                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | Confirm · reversible                                                                                                                                                                                                                                                                                                            | Feedback                                            | Key |
| ----------------------------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------- | --- |
| Restaurar (row)               | No row is busy. | Undoes the ignore, durably. The piece returns to Triagem's dock as «aguarda extração» if it already had an adapter, or «sem cliente» if not. An extraction it already had is not asserted here: it is recognized again on Triagem's next read with MedLens up, where it reappears as «pronta».                                                                                                                                                                                                                                                                                             | none · yes (ignore it again in Triagem)                                                                                                                                                                                                                                                                                         | Toast «Peça de volta à doca.»; the row disappears.  | —   |
| Excluir (row, first click)    | No row is busy. | Opens the inline confirmation under the row. Tooltip on the button: for an e-mail piece «Some da doca para sempre. Não apaga o e-mail — reenviar pela extensão traz de volta.»; for a watched file «Apaga o arquivo da pasta vigiada. Aqui não há e-mail para trazer de volta.»                                                                                                                                                                                                                                                                                                            | —                                                                                                                                                                                                                                                                                                                               | —                                                   | —   |
| Excluir (in the confirmation) | —               | Deletes the piece's file. For an e-mail delivery it also deletes the piece's companion viewer file when no other piece names it, and the whole delivery folder once no file it names remains. It then forgets the piece's three decisions (the ignore, the adapter choice, the extraction link). For a watched file whose content was replaced since it was ignored (a different modification time), the file is **left alone** and only the stale decision is forgotten. (This happens only if the file changes after the list was read: a replaced file's old decision is never listed.) | Inline confirmation. E-mail: «Excluir de vez? O e-mail continua no Thunderbird — reenviar traz a peça de volta.» Watched: «Excluir de vez? Apaga o arquivo da pasta vigiada — sem e-mail para trazer de volta.» · E-mail: only by re-sending from Thunderbird through the extension (a new delivery). Watched: **no way back**. | Toast «Peça excluída da doca.»; the row disappears. | —   |
| Manter (in the confirmation)  | —               | Closes the confirmation.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | —                                                                                                                                                                                                                                                                                                                               | —                                                   | —   |

No keyboard shortcuts; `Esc` does not close the confirmation. There is no bulk gesture.

## Business rules

1. **One piece per gesture.** There is no bulk restore or delete. [docs/telas.md § /triage; project memory]
2. **It is work set aside, not a preference**, so it lives in the sidebar. [telas.md § /discarded]
3. **Deleting touches the file, never the e-mail.** The e-mail lives in Thunderbird, so the sentence must say re-sending is the way back. A watched file must say the **opposite** honest thing: there is no e-mail to bring it back. [decision 020 § 5; decision 006 § 2]
4. **«Excluir» asks before it acts**, with an inline confirmation under the row: amber for "a human decides here", danger only on the act. This is a recorded departure from the design, which had no confirmation. [006 § 2]
5. **Only a piece the operator already ignored can be deleted.** The backend checks membership in the ignored list before touching any file. [`lib.rs` › delete_discarded]
6. **A watched file's identity includes its content revision.** A replaced file is a different document nobody decided about, so it is left alone. [020 §§ 3, 5]
7. **Absence must look like absence**: no date when the instant was never recorded. [AGENTS.md; `dock.rs` › excluded]
8. **A failed read is said, never shown as an empty list.** [`+page.svelte`]
9. **A sign of life before the command leaves.** [AGENTS.md › The machine]
10. **The disk rules existence.** A decision whose file is gone draws nothing. [`dock.rs` › discarded_pieces]
11. **Restoring never asserts an extraction.** The piece goes back to what the disk would say of it; MedLens's recognition brings back «pronta» later. [`dock.rs` › un_ignore]
12. **Ignored pieces are left out of the global search**, and the search points here instead. [decision 039]

## Must survive the redesign

- One row per ignored piece, with the file name and a meta line naming **when it was ignored** (only if known), **size** and **source** — told three ways (e-mail sender / e-mail body / watched folder).
- «Restaurar» and «Excluir» per row, one piece per gesture.
- The **confirmation before Excluir**, with the two honest sentences (e-mail vs watched file) and the two honest tooltips.
- The not-asked-yet silence, a failure that is said (never an empty list), and the empty state's sentence and note.
- The sign of life while a command runs, and every gesture blocked until it answers.
- dd/mm/aaaa dates; theme tokens only ([02-invariants](../02-invariants.md)).

## Free to redesign

- Cards vs a table ([05-components](../05-components.md) › Grid), row density, the icons, and the list's width.
- The confirmation's form — inline, popover or dialog — as long as the act needs a second, deliberate step, carries the right sentence, and its "keep" choice is at least as easy as the destructive one.
- The subtitle's wording layout; the button styles.

## Sample content for mockups

Fictitious. Subtitle: «Trabalho posto de lado — 4 peças que saíram da doca».

| Icon  | File name                             | Meta                                                                          | Excluir says             |
| ----- | ------------------------------------- | ----------------------------------------------------------------------------- | ------------------------ |
| PDF   | `informe_vale_verde_2026-08-26.pdf`   | ignorada em 02/09/2026 · 09:12 · 1,2 MB · de faturamento@valeverde.example    | e-mail sentences         |
| table | `Informe de consumo OPME — semana 36` | ignorada em 02/09/2026 · 09:10 · 18 KB · corpo do e-mail                      | e-mail sentences         |
| image | `foto_duplicada_sala3.jpg`            | ignorada em 03/09/2026 · 17:02 · 2,3 MB · da pasta vigiada                    | watched-folder sentences |
| image | `etiquetas_antigas_sol_nascente.jpg`  | 1,9 MB · de contas.opme@solnascente.example _(no instant recorded → no date)_ | e-mail sentences         |

Confirmation open on the third row: «Excluir de vez? Apaga o arquivo da pasta vigiada — sem e-mail para trazer de volta.» [Manter] [Excluir].

## Design opportunities

Proposals, not decisions.

1. **Give the e-mail's own context.** The row shows when the piece was ignored, but not the e-mail's date or subject (for an attachment), which is how a faturista recognizes a delivery. The e-mail's date already comes with each row (unused); the subject would need the backend.
2. **Group or filter when the list grows**, e.g. by day of the decision or by e-mail. There is no filter today, and nothing prunes old decisions.
3. **Tell watched files from e-mail pieces visually** — a folder or envelope mark — not only by the phrase at the end of the meta line, because the delete's consequences differ (a way back vs none).
4. **After «Restaurar», offer to open the piece in Triagem** — e.g. a toast action. The dock does not focus it on arrival.
5. **Name the file inside the confirmation, and support `Esc` («Manter») and a focused default on «Manter».**
6. **Say when a restored piece was already extracted** («já extraída — volta como pronta quando o MedLens responder»). The row carries no extraction fact today; it would need the backend.
7. **A sidebar counter**, when the house's counters arrive (telas.md describes this item as "a count that goes up and down"). It must count exactly the rows drawn.

## Open questions

1. Deleting forgets the piece's extraction link, and the Revisão list keeps only extractions linked on this machine (per the backend code). Should «Excluir» warn when the piece had been extracted, since its bills may leave Revisão? (Unverified end to end; see [Revisão](03-review.md).)
2. Should a watched file's delete move the file to the desktop trash instead of deleting it, given there is no e-mail to bring it back?
3. Decisions whose file has left the disk are neither drawn nor cleaned. Should the screen say "N decisões antigas sem arquivo", or clean them silently?
