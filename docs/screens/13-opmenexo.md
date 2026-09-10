# Opmenexo — `/opmenexo`

> Where: Sidebar › **Ideias futuras** › **Opmenexo** (icon `ph-plugs-connected`); drawn disabled with the tooltip `Precisa do Spook configurado — Configurações · Spook` while no Spook folder **or** address is stored, and with a dimmed icon (`Spook parado — Configurações · Spook`) while the Spook is stopped — see [shell](../04-shell.md). Also reached from every Opmenexo notice in the bell. · Status: **built** (panel, piles, order subtela for invoices and quotations, messages, conversation dialog, operator's clock) · Code: `src/routes/opmenexo/+page.svelte`, `src/lib/opmenexo.svelte.ts` (store, jobs, clock), `src/lib/components/OrderDialog.svelte` (subtela), `src/lib/components/OrderMessages.svelte`, `src/lib/components/ConversationDialog.svelte`, `src/lib/spook.svelte.ts`, `src/lib/ipc.ts`; Rust `src-tauri/src/lib.rs` (commands), `spook.rs`, `portal.rs`, `invoice_find.rs`.

This is the largest surface of the app. It is written in five parts, each with its own question, anatomy, data, states and actions; the rules, the must-survive list, the sample content, the proposals and the open questions are shared at the end.

- **A** — the panel (three counts and the truth about them)
- **B** — the three piles (the lists behind the numbers)
- **C** — the order subtela, invoice flow (`FA`)
- **D** — the order subtela, quotation flow (`SP`, and `PL`/`P` read-only)
- **E** — messages: the subtela's _Mensagens do pedido_, the conversation dialog, marking read

Vocabulary used below: **portal** = OPMENEXO, the hospitals' portal; **Spook** = the local service that is the only thing that talks to the portal; **order** (_ordem_) = one portal order, identified by the portal's own number; **pile** = one of the three lists; **subtela** = the full-screen order dialog; **write** = any gesture that changes something in the portal (invoice send, quotation answer, laudo upload, message send, mark-read sweep).

---

## Part A — The panel

### The question

_Há trabalho me esperando no portal, e de que tipo?_ — three counts (orders ready for an invoice, quotations waiting for an answer, conversations with unread messages), each drawn with **when** it was read, **whether** the read was complete, and **whether it happened at all**. The screen's own subtitle (verbatim): `Há trabalho esperando no portal, e de que tipo — responder cotação e subir nota continuam lá dentro`.

### Where it sits in the work

- **Outside the conveyor.** The conveyor (MailBridgr → Bridgr → Fantasma) ends at the ERP. The portal is the hospital's side: it authorises billing (status `FA`), asks for the post-surgery quotation (`SP`) and carries a conversation per order. The Spook reads and writes the portal; this screen only talks to the Spook.
- **Upstream of the invoice flow**: the ERP issues the NF-e and the NFePackMonitor downloads it into a notes folder — the source of "Anexar pelo número" and of the patient-note suggestions (Part C; see [NFe Pack](12-nfe-pack-monitor.md)).
- **Configured in** [Settings](10-settings.md) → Spook: folder/address, the app-issued credential and its scopes, the portal login, the OPMENEXO clients block, and the automatic-read clock (`Releitura automática` · `A cada quantos minutos`, inside the OPMENEXO clients dialog).
- **Outcomes land** in the bell and [Avisos](17-notices.md) (sends, answers, news), in the journal ([Logs](11-logs.md): message sends, sweeps, failed automatic reads) and in toasts. The Spook's live state is the status bar's block ([shell](../04-shell.md)).
- **Rhythm**: the faturista reads the panel a few times a day (or lets the clock do it), opens a pile, opens an order to send its invoice or answer its quotation, and opens a conversation to read and answer.

### Anatomy

Reading order, top to bottom. The screen never scrolls as a page — only the pile's grid scrolls inside its frame **[functional]** (otherwise the scrollbar lands on the window edge, outside any frame).

1. **Header row** (single row, items bottom-aligned, divider underneath)
   - Left block: title `Opmenexo`, beside it a dashed pill with an eye icon reading `fatia 1 · só leitura` (rendered in uppercase, tiny, muted); under the title the subtitle quoted above.
   - Right: the **read stamp** — an icon and a line of text (see _Data shown_). Icon: clock; spinning arrows while reading; a countdown clock, and the whole stamp in amber, when the service is unconfigured or stopped.
   - Far right: button `Reler` with an arrows icon that spins while a read is in flight. **[functional]** The stamp sits next to the gesture that renews it.
2. **Body**
   - **Unconfigured** (no Spook folder and no address stored): only an empty state — plugs icon, title `O serviço de portais não está configurado`, note `Sem o Spook o app não tem como entrar no portal para perguntar. Nenhum número é desenhado aqui — inventar zero seria pior que não mostrar nada.`, primary button (gear icon) `Configurações → Spook`. No banners, no cards, no numbers.
   - Otherwise, **banners** stacked at the top of the body **[functional]** (a failure drawn in the middle read as if the whole screen were an error and covered the numbers):
     - danger banner (warning-circle icon) — either the "outdated" sentence with a button `Ir ao Spook`, or the raw failure message with no button;
     - warn banner (plugs icon) — `O serviço de portais não está no ar. Os blocos abaixo estão apagados porque a leitura não pôde acontecer — não porque não haja trabalho.` with a button `Ver o Spook`.
   - **Never read**: empty state with a pointing-hand icon, title `Ninguém perguntou ainda`, note `Os números aparecem quando você mandar ler. Cada leitura percorre as filas do portal, cliente por cliente — por isso ela é um gesto, e não um relógio.`
   - **Three cards**, one row of three equal columns across the full window width (no maximum width, no wrapping). Each card is one button, in this order inside: top line (pile icon · spacer · status chip when the count is unknown or partial), the big number, the label, the support line, and an optional note separated by a thin rule. The card whose pile is open wears the accent border plus an accent bar along its bottom edge. While the service is stopped the whole row is drawn at half opacity.

     | Card | Icon | Label | Support line |
     | --- | --- | --- | --- |
     | invoices | `ph-file-text` | `Prontas para nota` | `Ordens com faturamento autorizado, esperando a NF subir.` |
     | quotations | `ph-chat-teardrop-dots` | `Cotações a responder` | `Pós-cirúrgico solicitado, esperando resposta.` |
     | messages | `ph-envelope-simple` | `Mensagens não lidas` | `Conversas do portal com mensagem nova.` |

   - **Under the cards**: either an empty state with a tapping-hand icon, `Escolha um bloco acima para ver a lista por trás do número.`, or the open pile (Part B), which takes the remaining height.
3. **Footer bar** (divider above, pointing-hand icon): `Leitura sob demanda: nada aqui muda sozinho — só quando você pede. Nenhuma ação sai desta tela.` — or, unconfigured, `A tela só volta a desenhar números quando o serviço de portais estiver configurado.`

### Data shown

| Element | Meaning | Presentation rule | Example |
| --- | --- | --- | --- |
| Card number | How much work the pile holds | `null` from the Spook → `—` in amber (**never 0**). If the reading brought rows for that pile, the number is **the count of that pile's rows the operator has not hidden**; if it brought no rows, the Spook's own count. | `3` |
| Messages card number | Same rule | With rows, it counts **non-hidden conversations**; without rows, the Spook's `unread_messages` (unverified whether the Spook counts messages or conversations — see Open questions). | `4` |
| Zero | Nothing waiting | Number drawn muted grey, note `Nada esperando.` — judged on the Spook's count, so a pile whose rows are **all hidden** shows `0` in the normal (not muted) tone and without that note. | `0` |
| Chip `não sabido` | This count could not be read | Amber pill, question icon; the number is `—` in amber. Note: `Não conseguiu perguntar ao portal — a leitura falhou.` or, while the service is stopped, `Não perguntou: o serviço de portais não está no ar.` | — |
| Chip `incompleto` | Some clients did not answer this read | Amber pill, warning icon, on **every** card whose count is not `null`. Note: `Incompleto: {N} cliente(s) não responderam.` — N = number of clients whose read failed (their names are not drawn). | `Incompleto: 1 cliente(s) não responderam.` |
| Read stamp | When the numbers on screen were read | `lendo o portal…` while reading · `sem leitura` if never read · `lido agora` under one minute · `lido há {N} min` in whole minutes (never converted to hours) · plus ` · relê a cada {n} min` when the operator's clock is on. Computed when the panel draws — there is no ticking timer; it refreshes when the panel's state changes (a read, reopening the screen). | `lido há 9 min · relê a cada 15 min` |

The reading is **kept on disk with its stamp**: reopening the screen or restarting the app draws the last numbers with their age, never a blank screen and never numbers without a "when".

Deliberately **not** drawn (the operator removed them on 21/08/2026 — an exclusion, not something unbuilt): the "asked" cut under each number (status, etapa, client count — the Spook still sends it), the amber "clients without an etapa cut" warning, and a hidden-rows count on the cards. There is no counter badge on the sidebar item either.

### States

| State | Trigger | What is drawn |
| --- | --- | --- |
| Unconfigured | No Spook folder and no address in the stored configuration | Empty state + `Configurações → Spook`; `Reler` disabled; stamp amber with countdown icon; unconfigured footer. |
| Never read | Configured, no kept reading | Banners if any, then the `Ninguém perguntou ainda` empty state; stamp `sem leitura`. |
| Reading (sign of life) | `Reler` or the clock | Marked **before** the call: stamp `lendo o portal…` with spinning icon, `Reler` disabled with its icon spinning. The previous numbers stay on screen. |
| Read | The Spook answered | Cards with numbers, stamp aged from the read time. |
| Partial | At least one client failed | Every non-null card: `incompleto` chip + the partial note. |
| Unknown count | A count came back `null` | That card: `—` amber, `não sabido` chip, note; its pile shows the unknown-pile panel (Part B). |
| Service stopped | Configured and the last status check says unreachable | Warn banner + `Ver o Spook`; cards at half opacity; `Reler` disabled; stamp in amber; unknown notes switch to `Não perguntou: …`. |
| Service outdated | The Spook answered 404 for the panel route (running, but older than this screen) | Danger banner: `Este Spook está no ar, mas é de antes desta tela: ele não conhece a rota do painel. Desligue e ligue o serviço para ele pegar a versão nova.` + `Ir ao Spook`. The kept numbers stay. |
| Read failed | Any other failure (or the disk load failed) | Danger banner with the raw message as the service/bridge phrased it (it may be English). The kept numbers stay — **replacing true numbers with nothing because one attempt failed is worse than showing them beside the failure**. |
| Automatic read failed | The clock's read failed | Same banner as a manual failure (visible next time the panel is looked at); no toast (nobody clicked); journal event `opmenexo_auto_read_failed`. |
| News while away | An automatic read found a pile that **grew** while the operator is on another screen | One bell notice `Novidades no portal` with the deltas as its visible detail: `+{n} cotações`, `+{n} prontas para nota`, `+{n} mensagens`, joined with ` · `. Counts only — never an order, a patient or a message. Never on `/opmenexo` itself, never after a manual read, never for a shrinking pile. |
| Narrow window | Window at its minimum (1024×768) | No special layout: the three cards shrink in their fixed three columns; header and footer keep one line each. |

### Actions

| Gesture | Where | Preconditions | Effect | Confirmation | Reversible | Feedback |
| --- | --- | --- | --- | --- | --- | --- |
| `Reler` | Header | Configured, service not stopped, no read in flight (disabled otherwise, **no tooltip saying why**) | Reads the whole summary through the Spook (walks every client's queue — about 14 s warm) and keeps it on disk with the stamp; resets the operator's clock | None (a read changes nothing in the portal) | n/a | Stamp and numbers change; failures as banners |
| Automatic read | App-wide store, every 30 s check | Clock set (≥ 5 min) **and** Spook reachable **and** no read in flight **and** no portal write in flight **and** app not locked by the PIN **and** last read older than the interval. Any unmet guard skips silently. | The same read as `Reler` | — | n/a | Bell `Novidades no portal` only for growth and only off-panel; journal on failure |
| Click a card | Card | A reading exists | Opens that pile below; clicking the open card closes it | — | Yes | Card wears the picked style |
| `Configurações → Spook` / `Ir ao Spook` / `Ver o Spook` | Empty state / banners | — | Navigates to Settings, Spook section | — | — | — |

Shortcut: none on the panel itself; `Esc` belongs to the dialogs (Parts C–E).

---

## Part B — The three piles

### The question

_Quais ordens estão por trás do número, quais delas não são minhas, e o que eu já fiz em cada uma?_ — the rows that compose a count, the operator's own "not mine" decision over them, and what **this app** did to each order since the last read (invoice sent, quotation answered, conversation marked read).

### Where it sits in the work

Between the count and the action. The rows are the **kept snapshot** of the last read (minutes or hours old); the row's door opens the order or the conversation, and those read the portal **live**. An order leaves its pile only at the next read — which is why each pile carries a column with this app's own memory of what it just did.

### Anatomy

The open pile takes the remaining height under the cards:

1. **List header** (one baseline-aligned line): the pile's title (the card's label), `{n} ordem` / `{n} ordens`, spacer, the link `Ver as ocultas ({N})` / `Esconder as ocultas ({N})` (only when N > 0), and a muted note `somente leitura · o portal responde e fatura, não esta tela`.
2. **Sweep row — messages pile only**, right-aligned, drawn only when the credential may write (or its scopes are unknown): `Varrer mensagens` (envelope-open icon), which turns in place into its confirmation strip; while the sweep flies, a spinner line `Varrendo as mensagens no portal…`. **[functional]** It has its own row under the header because the long question broke the header line.
3. **Body**: one of the panels in _States_, or the grid.
4. **The grid** is the house `GridSurface` ([components](../05-components.md)), configured here with: filter placeholder `Filtrar as linhas…`; count `{n} ordem` / `{n} ordens`; a tick column and the selection bar `{n} linha marcada` / `{n} linhas marcadas` carrying two actions, `Não é minha — ocultar` (eye-slash) and `Trazer de volta` (eye); hidden rows flagged with the amber tint and amber left bar when they are shown; a **per-row action column** that is not a column of the table (never sortable, hideable, reordered or exported); no-match sentence `Nenhuma linha responde a este filtro.`. Column menu, density, pagination and column resize come with the surface. The column layout (order, hidden columns, sort, widths) is **remembered per pile** across sessions; switching piles restores that pile's layout and drops the other pile's filter and sort.

| Pile | Columns, default order |
| --- | --- |
| Prontas para nota | `Ordem` · `Cliente` · `Paciente` · `Cirurgia` · `Convênio` · `Nota` |
| Cotações a responder | `Ordem` · `Cliente` · `Paciente` · `Convênio` · `Data-limite` · `Esperando` · `Resposta` |
| Mensagens não lidas | `Cliente` · `Ordem` · `Novas` · `Última mensagem` · `Lida` |

**Row door**: on invoices and quotations an arrow-out icon, tooltip `Abrir a ordem`, opening the subtela (Parts C/D); on messages a chat-bubbles icon, tooltip `Abrir a conversa`, opening the conversation dialog (Part E). Quiet grey until hover. A row without a positive order number has no door. **[functional]** On quotations the convênio sits beside the patient so that `Data-limite` and `Esperando` stay together at the end — that pair carries the urgency.

### Data shown

| Cell | Meaning | Rule | Example |
| --- | --- | --- | --- |
| `Ordem` | The portal's order number | Text; `—` when the Spook sent none | `7100231` |
| `Cliente` | The requesting hospital | As sent | `Hospital Vale Verde` |
| `Paciente` | Patient (LGPD data) | As sent | `MARTA LEMOS FERREIRA` |
| `Cirurgia` / `Data-limite` | Surgery date / quotation deadline | First ten characters of the ISO value shown as `dd/mm/aaaa`; anything not shaped like a date is shown **as it came** (a portal date with a wrong year travels wrong — fixing it would be inventing); `—` when empty | `03/09/2026` |
| `Convênio` | Who pays | As sent | `VIDA PLENA SAÚDE` |
| `Esperando` | Whole hours elapsed since the `Data-limite` instant | `{d} d {h} h` from 24 h, `{h} h` below; never negative — a future date reads `0 h`; `—` when empty or unparsable | `1 d 4 h` |
| `Nota` | This app's memory of its invoice send (never the portal's truth) | `enviando…` while this app's send for the order flies; otherwise, if this app ever sent invoices for it, `enviada em {dd/mm/aaaa · hh:mm}` + ` · confirmada no portal` (every file of the **latest** send confirmed) or ` · sem confirmação`; otherwise blank | `enviada em 08/09/2026 · 10:42 · confirmada no portal` |
| `Resposta` | This app's memory of its quotation answer | `respondendo…` in flight; otherwise `respondida em {when}` + ` · confirmada no portal` or ` · sem confirmação` (latest answer); otherwise blank | `respondida em 08/09/2026 · 11:05 · sem confirmação` |
| `Novas` | Unread messages in that conversation | Integer as sent | `2` |
| `Última mensagem` | Stamp of the latest message | ISO → `dd/mm/aaaa · hh:mm` local; unparsable → as it came; none → `—` | `09/09/2026 · 16:20` |
| `Lida` | This app's memory of marking that one conversation read | `marcando…` in flight; otherwise `lida em {when}`; otherwise blank. The bulk sweep leaves no mark here. | `lida em 09/09/2026 · 17:02` |

Counters, exactly: the **header** count is the pile's rows after the hidden filter (before the text filter); the **grid** count is the rows matching the text filter; `Ver as ocultas (N)` counts this pile's hidden rows in the current reading.

### States

| State | Trigger | What is drawn |
| --- | --- | --- |
| Unknown pile | That pile's count is `null` | Dashed amber box, question icon, `Esta pilha não pôde ser lida` / `A leitura falhou para esta pilha — o número acima é um traço pelo mesmo motivo. Tente reler.` |
| Rows missing | The card counts more than zero but no rows came (a Spook older than the lists) | Dashed amber box, warning icon, `O serviço não mandou as linhas` / `O número acima veio, mas a lista não: este Spook é de antes das listas e ignorou o pedido delas. Desligue e ligue o serviço — os números continuam valendo enquanto isso.` + `Ir ao Spook`. Never drawn as "none". |
| Empty | No rows left after the hidden filter | Plain box, check-circle icon: `Nenhuma ordem esperando nota agora.` / `Nenhuma cotação esperando resposta agora.` / `Nenhuma conversa com mensagem nova.` |
| No match | The text filter matches nothing | Header row kept, `Nenhuma linha responde a este filtro.` |
| Showing hidden | Toggle on | Hidden rows back, flagged amber. The toggle is one setting for the three piles and lasts until the app closes. |
| Write in flight | Any portal write by this app | Its row shows `enviando…` / `respondendo…` / `marcando…`; `Varrer mensagens` disabled. |
| Bulk sweep in flight | `Varrer e marcar` confirmed | Spinner line `Varrendo as mensagens no portal…` instead of the button. |
| Service stopped | Spook unreachable | Piles stay browsable (the dimmed cards still open them); rows are the kept snapshot; doors still open, and the subtela/dialog then fail loudly. |

### Actions

| Gesture | Where | Preconditions | Effect | Confirmation | Reversible | Feedback |
| --- | --- | --- | --- | --- | --- | --- |
| Filter / sort / columns / density / page | Grid surface | — | View only; layout saved per pile | — | Yes | — |
| `Não é minha — ocultar` | Selection bar | Rows ticked | Hides each ticked order by its portal number, stored apart from the reading: survives every re-read and restart, and an order that leaves and comes back stays hidden | None | Yes | Rows vanish, card number drops, `Ver as ocultas (N)` appears |
| `Trazer de volta` | Selection bar | Rows ticked (meaningful while hidden rows are shown) | Un-hides them | None | Yes | Flag disappears, number rises |
| `Ver as ocultas` / `Esconder as ocultas` | List header | N > 0 | Shows / hides hidden rows | — | Yes | — |
| Row door | Row action column | Positive order number | Invoices/quotations → subtela; messages → conversation dialog | — | — | Dialog opens with its own sign of life |
| `Varrer mensagens` | Sweep row (messages) | Credential may write or scopes unknown (else not drawn); disabled while any portal write flies or when the Spook's unread count is 0 or unknown | The Spook **exports every unread conversation first**, then marks read what it exported; conversations this login cannot reach are never marked and never named | In-place strip: `Marcar as {n} mensagens não lidas como lidas no portal? O Spook exporta cada conversa antes de marcar; as que este login não alcança ficam como estão.` + `Cancelar` / `Varrer e marcar` (n = the Spook's unread count, not the grid's rows) | No (the portal holds the read state) | Toast `{n} mensagens marcadas como lidas no portal.` or `Nada para marcar — nenhuma mensagem não lida ao alcance.`; failure toast `A varredura não respondeu — nada garantido. O journal tem o registro.`; per-conversation counts go to the journal only. The pile changes only at the next read. |

---

## Part C — The order subtela, invoice flow (`FA`)

### The question

_O que esta ordem é **agora**, o que já está anexado nela, e posso subir a nota daqui?_ — the order read live from the portal; what it already carries shown **before** any gesture; the reasons it cannot receive a note said **before** the gesture; and the send, one order at a time, with one confirmation.

### Where it sits in the work

Opened from the invoices pile's door (the quotations pile opens the same subtela — Part D; the conversation dialog's order number opens it too). It is **the one place an invoice leaves for the portal**. Files come from the operator's disk (system chooser), from the NFePackMonitor's notes found by NF number, or from the monitor's notes that name this order's patient. After a successful send the portal takes the order out of `FA` (typically within the minute); the pile only drops it at the next read, and meanwhile its `Nota` cell carries this app's memory.

### Anatomy — the frame (shared with Part D)

- **Scrim** over the whole window (black at 55%); clicking it closes the subtela (it is a real button labelled `Fechar` — `Esc` is never the only way out).
- **Sheet**: centred, capped at **1440×900** (the design's frame), and never closer than 14 px to the window edge; the **fill-window toggle** stretches it to the window minus 14 px on each side. The app's frame stays visible around it **[functional]**, so it never reads as another program's window. Rounded corners, border, large shadow.
- **Header**, one line with a divider below: document icon · the order number in mono · `{Cliente} · {Paciente}` muted and truncated (once the order has loaded) · the **seal**, a mono pill with the portal's own words `{status} · {etapa}` (the status alone when the etapa is empty) — accent-tinted for `FA`, amber for `SP`, green for every other status · spacer · the `Esc` key cap followed by `fecha` · the fill-window toggle (arrows-out / arrows-in; tooltips `Ocupar a janela inteira` / `Voltar ao tamanho do desenho`) · close X (`Fechar`).
- **Body**: two columns and a draggable divider **[functional]**:
  - left, the **work column**, born at **60%** of the body width, scrolling on its own;
  - the divider: a 7 px grab area drawing a 1 px line (accent on hover and while dragging, resize cursor); dragging keeps the work column between **38% and 72%**; not remembered — each opening starts at 60/40;
  - right, the **documents column** (the rest): list on top, viewer filling the remaining height.
  - **[functional]** Each column scrolls by itself: one shared scroll would push the document out of sight exactly when the person scrolls down to the materials to compare them.

### Anatomy — work column (FA and every non-quotation status), top to bottom

Section rules everywhere in the subtela are a tiny uppercase label, an optional muted count/sub, and a hairline filling the rest of the row, with generous air above each section.

1. **`Identificação`** — label-over-value pairs in a fixed **three-column** grid; **[functional]** each pair is one cell, so a value can never fall into a label's place. Pairs: `Cliente`, `Paciente`, `Data da cirurgia`, `Convênio` (`—` when empty), `Médico`, `Atendimento` (mono, accent), `Valor da ordem` (mono, accent). A pair whose value is empty is omitted (Convênio excepted). Values stay on one line with ellipsis. Order number and status are not repeated (the header says them).
2. **`Materiais`** + sub `{n} materiais · texto como o portal escreveu` — the house `Grid` ([components](../05-components.md)) with no surface frame, filter, pager or selection (an order has dozens of lines, not thousands): `Descrição` · `ANVISA` · `Solicitado` · `Autorizado` · `Utilizado` · `Total` · `Status`. The quantity and total columns are declared numeric (right-aligned); the first column is prose, not an identifier (no accent, not frozen); clicking a header sorts by the text as sent. Height follows the content up to **300 px**, then scrolls inside. Footnote: `A ordenação é do texto que veio do portal — nada aqui é recalculado nem reformatado.` No items: package icon + `O portal devolveu a ordem sem itens. Não é falha de leitura — é uma ordem sem material listado.` (Manufacturer and unit price are deliberately left out: side by side with the guia, the operator reads description, registration, quantities and total.)
3. **`Notas já anexadas`** — below.
4. **`Anexar nota fiscal`** — below.
5. **`Mensagens do pedido`** — Part E.

### Anatomy — documents column (shared with Part D)

- Rule `Anexos do pedido` + the number of documents the order lists.
- Before the order has loaded: `Lendo a ordem no portal…`. No documents: `Este pedido não tem documentos anexados.`
- **List** (about 206 px tall, scrolls): one bordered row per document **name as the portal lists it**, with a type icon from the extension (PDF, image, XML/code, generic file):
  - not downloaded — name + small button `Baixar e ver` (download icon);
  - downloading — name + mono mark `baixando…` with a spinner; several names can download at once, each with its own mark;
  - downloaded and found on this machine — the whole row becomes the button that shows it, mark `baixado`; the row on display wears the accent border and tint;
  - downloaded but not found here — name + `Não achei este arquivo neste computador — pode ser um serviço em outra máquina, ou o arquivo já ter saído de lá.` **[functional]** An observation, never a diagnosis: from here a service on another machine, a file already cleaned and an unresolvable path look identical.
- Footnote: `Selecionar não baixa: cada anexo só desce do portal quando você pede — um por vez, o que você precisa ver.`
- Notices: amber `O download parou antes do fim: {mensagem}` (the service says what it got and what stopped it; shown beside what did download); danger text when the desktop viewer could not be opened.
- **Viewer bar** (only while a document is on display, hairline above): the file name (truncated) · for **images only** `−` (`Afastar`), `{zoom}%`, `+` (`Aproximar`), from 50% to 200% in steps of 25, back to 100% for every new document · small button `No sistema` (tooltip `Abrir no visualizador do sistema`).
- **Viewer frame** (the rest of the column, bordered, faintly tinted): while opening `Abrindo…` with spinner; nothing chosen: cloud-download icon + `Escolha um anexo para ver aqui.`; PDF: the webview's native embedded PDF viewer (its own zoom and paging); image: the picture fitted at 100%, scrollable when zoomed; any other type (XML): file-code icon + `Este tipo de arquivo não se desenha aqui — o visualizador do sistema é a segunda porta: abrir por ele é ver o arquivo como ele é, sem tradução nossa.` + button `Abrir no visualizador do sistema`.

### Anatomy — `Notas já anexadas` (non-quotation orders only)

What the order already carries, read in the same live call as the order and drawn **above** the attach area **[functional]** — a second copy of the same invoice is the expensive, silent mistake this section exists to prevent.

- **Read, with files**: one card per file name the portal lists (receipt icon, mono name). If this app has a send on record for this order, one more card: `NF {n} · NF {m}` · `Enviada por aqui em {dd/mm/aaaa · hh:mm}` · chip `confirmada no portal` (green, when every file of the latest send was confirmed) or `sem confirmação` (amber).
- **Read, empty, and no send on record**: a dashed box `Nenhuma nota anexada nesta ordem — o portal respondeu a lista, e ela veio vazia.` (read-and-empty, never supposed-empty).
- **Service too old for this list**: amber notice `Este Spook não conhece a lista de notas anexadas — suba o serviço de novo para ele passar a responder.`
- **Could not be read**: amber block with a question icon, `Não deu para ler o que já está anexado. Isto não é "não tem nota" — é ninguém ter conseguido olhar, e subir uma nota sem essa resposta é como nasce a segunda via.` + small button `Reler esta ordem`. **Never** drawn as "no invoice". (The attach gestures below are still offered in this state when the order qualifies.)

### Anatomy — `Anexar nota fiscal` (non-quotation orders only)

Top to bottom:

1. **One block panel, the first that applies — always before the gesture** **[functional]** (choosing files only to be refused afterwards is work thrown away):

   | Order | Condition | Panel |
   | --- | --- | --- |
   | 1 | Credential scopes are known and lack the write scope | Amber, key icon — `A credencial deste portal não tem escopo de escrita` / `Esta credencial não tem permissão de envio — ela foi emitida quando o app só lia. Reemita em Configurações → Spook; tentar de novo com a mesma não muda nada.` + button `Configurações → Spook` |
   | 2 | The portal lists at least one attached file, **or** this app has a send on record for the order | **Green** (the only green panel), check icon — `O portal já recebeu a nota desta ordem` / `Não há nada de errado aqui: o que foi enviado está acima. O portal tira a ordem de FA assim que recebe a nota, e por isso não há mais campo de upload.` It silences the stage panel below. |
   | 3 | Status is not `FA` | Quiet grey, prohibit icon — `O portal não oferece campo de upload nesta etapa` / `Status {status · etapa} — o portal só recebe nota enquanto a ordem está em FA (autorizada a faturar).` |
   | 4 | `FA`, but the order has no invoice upload frame | Same title / `Esta ordem não tem onde receber nota fiscal no portal — não há quadro de upload nela.` |

2. **This order's send outcome notices**: credential refused at send time → danger notice with the credential sentence of row 1; no answer → danger `O envio não respondeu. ⚠️ Não mande de novo antes de olhar o portal: se o arquivo já subiu, a segunda tentativa vira uma segunda via da mesma nota.` followed by the cause in small amber text.
3. **The report** (after the service answered; kept until the app closes, surviving closing and reopening the subtela): one card per file, border and icon toned by the answer — file name (mono) · `NF {n}` · verdict:

   | Answer | Tone | Verdict text |
   | --- | --- | --- |
   | upload failed | red, x-circle | `falhou` |
   | skipped (the run stops at the first failed POST) | amber, warning | `não enviada (a corrida parou antes)` |
   | uploaded, confirmed on re-read | green, check | `enviada · confirmada no portal` |
   | uploaded, not found on re-read | amber, warning | `enviada · não encontrada no portal` |
   | uploaded, re-read did not happen (`unknown`) or no verdict | amber, warning | `enviada · não deu para conferir — isto não é envio que falhou, e reenviar aqui é o que duplica a nota` |

   Then the footnote (info icon) `Não há “tentar de novo” aqui: resposta que não chegou não é envio que não aconteceu. Releia a ordem — ou olhe no portal — antes de anexar de novo.`, then `Evidência do serviço:` + the evidence file path in mono (when reported), then small button `Reler esta ordem`. There is **no retry button, by design**.
4. **Gestures** — drawn only when the order is `FA`, has an upload frame, and the credential may write (or its scopes are unknown):
   - a danger notice with the raw message when choosing or locating failed (for example an empty file);
   - **Suggestions** (only when there are any): dashed box titled `Notas deste paciente, baixadas pelo monitor`; one line per note: NF number (mono) · `uso {dd/mm/aaaa}` (when the note carries a use date) · the note's value in pt-BR with two decimals (when known) · spacer · small button `Anexar` (paperclip; disabled while locating or sending). At most five, newest first. A suggestion disappears once its number is in the basket or appears inside an attached file's name. No notes → no box at all (never an empty-state sentence).
   - **The basket**: one card per staged file — PDF icon · file name (mono, wraps anywhere: file names can carry patient names) · size (`B` / `kB` / `MB`) and, when the file was found by its number, the accent badge `pelo número` · label `Nº da NF` + a mono field (placeholder `—`) · remove X (tooltip `Tirar do envio {nome}`). Field and X are disabled while sending.
   - **While this order's send flies**: one line `Enviando ao portal…` with a spinner replaces everything below the basket **[functional]** — one line, never a per-file progress (the bridge does not say which file is in flight).
   - **Otherwise**: the amber results of the last _Localizar_ (`Nenhum número de NF na entrada.`, or one line **per** missing number: `Nenhum arquivo com {número} em {pasta}.`), then the gestures row (wraps when narrow): number field (mono, about 190 px, placeholder `nº da NF — 1234, 1239 ou 1234-1237`, Enter = Localizar) · `Localizar` (magnifier; `Procurando…` with spinner while searching) · `Escolher arquivos` / `Escolher mais arquivos` (paperclip) · with files in the basket, the send button `Enviar 1 nota` / `Enviar {n} notas` (paper-plane) · the why-sentence in amber: `Há um envio em andamento na ordem {ordem}. Um por vez, sem fila — este fica para quando aquele responder.` or `Falta o número da NF de {arquivos}.` (names joined with ` e `). Without the NFePackMonitor folder configured, the field and `Localizar` are disabled with the tooltip `Configure a pasta do NFe Pack Monitor (Configurações → NFe Pack) para localizar pelo número.`
   - Footnote: empty basket → `Um ou vários de uma vez. Cada arquivo leva o seu número de NF, e a confirmação é uma só, para a ordem inteira.`; with files → `O número é o que o portal procura depois para confirmar que a nota chegou. Errado, a conferência volta "não encontrada".`

### Data shown

| Field | Meaning | Rule | Example |
| --- | --- | --- | --- |
| Seal | Portal status code and etapa | Verbatim, never translated (the portal decides whether an upload frame exists) | `FA · AGUARDANDO NF` |
| Identification values | The order as the portal's detail page labels it | **Text as the portal wrote it** — dates are label text, not ISO, and are never reformatted | `Data da cirurgia` `02/09/2026` |
| `Atendimento`, `Valor da ordem` | Codes/amounts checked digit by digit | Mono, accent | `2026004517` · `R$ 2.847,50` |
| Materials cells | Quantities, totals, status | All text as sent; nothing recalculated | `2` · `1.250,00` |
| Attached file names | Invoices already on the order | As the portal lists them | `NF_100245.pdf` |
| Send memory card | What this app sent (latest send) | `NF` numbers of that send, stamp, confirmed only if every file confirmed | `NF 100245 · NF 100246` |
| Suggestion | A monitor note whose own text names this order's patient | Number · use date · value | `100245` · `uso 02/09/2026` · `2.847,50` |
| NF number in the basket | The number the portal later searches to confirm the note | Picked file → pre-filled from the first run of 4+ digits in its name, **a suggestion**; found by number → the number that found it, **a fact** (`pelo número`); editable either way | `100245` |
| Size | Staged bytes | `B` under 1 kB, whole `kB` under 1 MB, `MB` with one decimal | `184 kB` |
| Evidence path | The Spook's evidence artifact, written before the first POST | Mono path as reported | `tmp/evidence/…json` |

### States

| State | Trigger | What is drawn |
| --- | --- | --- |
| Opening (sign of life) | Door clicked | The subtela opens at once: header with the number only; work column `Lendo a ordem no portal…` with spinner; documents column `Lendo a ordem no portal…`. A read may **wait** behind another order's write (the Spook serializes portal access) — it waits, it does not fail. |
| Loaded | Portal answered | Everything above. |
| Service too old | The Spook answers 404 for the order route — or later for the download, send or answer route | The whole work column becomes an amber block (arrow-up icon): `O Spook desta máquina é mais velho que esta tela` / `Este Spook está no ar, mas é anterior a esta tela: ele não conhece a rota da ordem. Desligue e ligue o serviço para ele subir na versão nova.` |
| Read failed | Any other failure | Danger notice with the raw message in the work column; the documents column keeps its `Lendo a ordem no portal…` line. |
| Re-reading | `Reler esta ordem`, or automatically after a successful send | The work column shows only `Lendo a ordem no portal…` until the answer; then everything returns, the report included (a failed re-read keeps what was on screen). |
| Scope unknown | The status poll has not answered yet | Gestures offered; the service refuses if it must. |
| Busy elsewhere | Another order's portal write is in flight | Send button disabled; why-sentence names that order. |
| Sending | This order's send in flight | Basket frozen, one waiting line. Closing the subtela does not stop it; reopening shows the waiting line (the basket itself was given up with the close). |
| Sent | Service answered | Report cards, basket emptied, order re-read (the portal usually moved it out of `FA` → the green "already received" panel, the invoice under `Notas já anexadas`, the send card with its chip). |
| Send failed / refused | No answer, or credential refused | The notices of item 2; the basket stays as it was. |
| Located | `Localizar` answered | Found files join the basket with `pelo número`; one amber line per missing number; the field clears only when every number was found. |
| Narrow window | 1024×768 minimum | Subtela 996×740; at 60/40 the documents column is about 390 px; identification stays in three columns; the gestures row wraps. |

### Actions

| Gesture | Where | Preconditions / disabled reason | Effect | Confirmation | Reversible | Feedback |
| --- | --- | --- | --- | --- | --- | --- |
| Close — `Esc`, scrim, X | Anywhere in the subtela (`Esc` works even while typing in a field) | — | Closes; **discards the basket** (staged bytes dropped on the other side — closing is giving up the send), the typed NF numbers and the shown document; any job in flight keeps running | None | Reopen re-reads the portal | — |
| Fill-window toggle | Header | — | 1440×900 frame ⇄ window size; session only | — | Yes | Icon and tooltip swap |
| Drag divider | Between columns | — | 38–72% work column | — | Yes | Line in accent while dragging |
| `Baixar e ver` | Document row | Not already downloading that name | Downloads that one document through the Spook (a portal **read** — nothing changes there); if found on this machine it opens in the viewer at once | — | — | Row mark `baixando…` → `baixado`, or the not-found observation; partial-download notice |
| Click a downloaded row | Document list | File found here | Shows it in the viewer | — | — | Row picked style |
| Zoom − / + | Viewer bar | Image on display | 50–200% in 25% steps | — | Yes | `{zoom}%` |
| `No sistema` / `Abrir no visualizador do sistema` | Viewer bar / unsupported frame | A document on display | Opens it in the desktop's own viewer (only files the service reported can be opened) | — | — | Danger text if it could not |
| `Reler esta ordem` | Unreadable block, report | — | Re-reads the order live, keeps the report | — | — | Re-reading state |
| `Anexar` (suggestion) | Suggestion line | Not locating, not sending | Runs `Localizar` with that number | — | Remove from basket | Same as Localizar |
| `Localizar` / Enter | Number field | Monitor folder configured; input not empty; not already searching | Parses the input — lists `1234, 1239`, ranges `1234-1237` (at most 100, zero-padded to the wider end), split digits `10024 5` → `100245`; runs under 3 digits ignored — and searches the monitor's note folders for PDF/XML files whose **name** carries the number as a whole digit group (`1234` never matches `12345`); every file found enters the basket with its number as a fact; a number with PDF and XML brings both; a file already in the basket (same content hash) is not added twice. **Nothing is sent.** | — | Remove from basket | Amber lines, badges |
| `Escolher arquivos` | Gestures row | — | System chooser, several files at once (title `Escolha a(s) nota(s) fiscal(is)`, filter `Nota fiscal (PDF, XML)`); an empty file is refused; the same file twice is one line | — | Remove | Cards appear |
| Edit `Nº da NF` / `Tirar do envio` | Basket card | Not sending | Changes the plan | — | Yes | Why-sentence updates |
| `Enviar {n} notas` | Gestures row | Every file has a number (else `Falta o número da NF de …`); no other portal write in flight (else the busy sentence) | Uploads the whole plan to this order in **one** call: the Spook writes its evidence first, posts file by file (stopping at the first failed POST), then re-reads to verify each | **In place** (house `ConfirmButton`, [components](../05-components.md)): `Enviar ao portal, na ordem {ordem}: {arquivo} como NF {número} e {arquivo} como NF {número}.` + `Cancelar` / `Enviar` | **No** — and the app never re-sends by itself | Waiting line; toast `Notas enviadas na ordem {ordem} — o portal respondeu.` or `O envio da ordem {ordem} não respondeu. Olhe a ordem antes de tentar de novo.`; bell `Notas enviadas ao portal` / `Envio de notas não respondeu` (detail: the order number only); pile cell `Nota`; report; re-read |

---

## Part D — The order subtela, quotation flow (`SP`; `PL`/`P` read-only)

### The question

_Quanto foi usado, a que preço — e o portal recebeu a minha resposta?_ — the post-surgery quotation answered from the same subtela: the page's material lines become an **answer sheet** where two columns belong to the operator, new lines can be added when the page lacks them, the pós laudo rides the same send, **one** confirmation names every line and the total, and the report speaks per line, with the portal's own words when it refused.

### Where it sits in the work

Opened from the quotations pile's door (or any door onto an order whose **live** status is `SP`, `PL` or `P`). `SP` = post-surgery quotation requested; a successful answer moves the order to `PL`. A price is a **commercial proposal**, so this is the strictest write in the app. Frame, header, documents column and _Mensagens do pedido_ are Part C's; on quotations the `Materiais` grid, `Notas já anexadas` and `Anexar nota fiscal` are **not drawn** — the sheet is the materials view, and an invoice does not exist at this stage.

### Anatomy — work column

1. **`Identificação`** — Part C's grid with two swapped labels: `Cliente`, `Paciente`, `Data-limite da cotação` (the portal's own deadline text), `Convênio`, `Médico`, `Atendimento`, `Valor solicitado`.
2. **`Responder cotação`** + sub `{n} linhas do portal · duas colunas são suas` / `1 linha do portal · duas colunas são suas` / `a página desta ordem chegou sem linhas`.
3. **The answer sheet**, in a bordered frame: the house `Grid` ([components](../05-components.md)) with custom cells — no outer frame, proportional font, no frozen identity column, no tick column. **[functional]** Its body is static per order: typing never rebuilds the grid, so the field keeps focus. Ten logical columns:

   | # | Column | Page row | New row |
   | --- | --- | --- | --- |
   | 1 | `Código` | The page's child product id, mono, accent | Field, placeholder `referência da etiqueta`, tooltip `A referência da etiqueta do material usado — o DE-PARA a traduz para o produto do portal`; the DE-PARA is asked when the field is left |
   | 2 | `Material` | Description; plus ` · linha sem id na página — responda no portal` when the id is not numeric (that row cannot be answered here) | The DE-PARA state (below) |
   | 3 | `ANVISA` | Mono | Resolved → mono, read-only; otherwise a field, placeholder `registro ANVISA`, tooltip `O registro ANVISA, à mão — a porta quando o DE-PARA não conhece a referência` |
   | 4 | `Situação` | Mono pill with the portal's text, amber for `Em Aberto` | Accent pill `+ nova` |
   | 5 | `Solic.` | Requested quantity, as sent | — |
   | 6 | `Autoriz.` | Authorised quantity, as sent | — |
   | 7 | `Qtd. utilizada` | Field (mono, right-aligned, numeric keyboard), pre-filled with the page's value or `0`; tooltip `Quantidade utilizada — o portal solicitou {sol} e autorizou {aut}` | Field, tooltip `Quantidade utilizada no ato cirúrgico` |
   | 8 | `Valor unit. (R$)` | Field pre-filled with the page's unit price or `0,00`; tooltip `Valor unitário em reais, como vai na proposta` | Field, same tooltip |
   | 9 | `Total da linha` | Live `qtd × valor` in pt-BR with two decimals, or a faint `—` while either is invalid | Same |
   | 10 | `Na resposta` | Pencil pill `alterada` when the row differs from what the page brought | Remove X, tooltip `Remover a linha nova` |

   - **Add-only mode** — the page brought no lines: `Solic.` and `Autoriz.` are not drawn (they would be zero on every new line — dead width where the fields need room); the other eight stay.
   - Changed page rows and filled new rows wear the accent tint and a left accent bar — never the only signal (the pill says it too).
   - **Read-only sheet** — blocked (scope or already answered) or while the answer is in flight: every field becomes plain text, remove buttons disappear.
   - **New-row `Material` states**: idle, faint `digite a referência — o DE-PARA responde o produto`; asking, faint `consultando o DE-PARA…`; resolved, the catalogue description, plus ` · casamento fraco no DE-PARA, confira` for a weak match or ` · ANVISA da planilha, sem casamento no catálogo — confira` when the registration came from the mapping sheet only; not in the map, amber `não está no DE-PARA — informe o registro ANVISA ao lado`; the map answers several registrations, amber `o DE-PARA responde mais de um ANVISA para esta referência — informe o registro ao lado`; could not ask, amber `não deu para consultar o DE-PARA — saia e volte ao campo para tentar de novo, ou informe o ANVISA ao lado`. **[functional]** "not in the map" and "could not ask" are different sentences.
   - **Empty sheet** (no page lines, no new lines): list icon, then — when answering from here is possible — `A página desta ordem chegou sem linhas` / `Responder aqui é adicionar: crie uma linha por material usado — a referência da etiqueta vira o produto do portal pelo DE-PARA.`; otherwise `Não há linha para cotar nesta ordem` / `A página da ordem chegou sem material — e daqui não se responde esta cotação; o painel abaixo diz o porquê.`
   - **Sheet footer** (inside the frame, divider above): small button `Adicionar linha` (+; only on `SP` with write scope and no answer in flight; disabled while the confirmation panel is open) · the travel sentence (amber when nothing travels; silent when read-only) · spacer · label `Total da resposta` (tiny uppercase) + the total in bold mono — drawn only when at least one **page** row travels, otherwise `—`.
   - **Travel sentence** — nothing: `Nenhuma linha alterada ainda — a resposta leva só o que você mudar.`; otherwise `1 linha viaja ao portal` / `{n} linhas viajam ao portal`, then ` · 1 delas é nova` / ` · {k} delas são novas`, then ` · as outras {m} ficam como estão` or ` · todas as linhas vão`, then ` · 1 linha nova vazia fica` / ` · {k} linhas novas vazias ficam`.
4. **Block panel** under the sheet (the first that applies): no write scope → amber, key icon, `A credencial deste portal não tem escopo de escrita` / `A folha acima fica em leitura: daqui não se responde esta cotação. O aviso vem antes de preencher — preencher para ser recusado depois é trabalho jogado fora.` + `Configurações → Spook`; already answered (`PL`/`P`) → **green**, check icon, `O portal já registrou a resposta desta cotação` / `Ela saiu de SP. A folha acima é o que foi respondido, em leitura: não há o que preencher de novo, e nada de errado aconteceu aqui.` + `Respondida por aqui em {dd/mm/aaaa · hh:mm}.` when this app answered it. The scope panel wins when both apply.
5. **Answer outcome notices**: credential refused → danger, the credential sentence of Part C; refused by the service **before anything was written** → amber `O portal de agora discorda da tela — nada foi escrito. Releia a ordem e confira as linhas antes de tentar de novo.` + the service's sentence verbatim + `Reler esta ordem`; no answer → danger `A resposta não respondeu. ⚠️ Não reenvie antes de olhar o portal: preço é proposta comercial, e se a resposta chegou lá, a segunda tentativa é uma segunda proposta.` + the cause.
6. **Answer report** (after the service answered; kept until the app closes): one bordered card per answered line — icon · product (the resolved product when the service named it, else the page text, else the id) · the account `{q} un × {valor} = {total}` (page rows) · verdict · detail · optional citation:

   | Line outcome | Tone | Verdict | Detail |
   | --- | --- | --- | --- |
   | written, and the portal confirmed the answer | green | `respondida · confirmada no portal` | `A página da ordem passou de SP para PL com esta linha.` |
   | written, confirmation missing | amber | `respondida · não deu para reconferir` | `A linha subiu e o portal não devolveu a página para conferir. A cura é olhar a ordem no portal — nunca responder de novo.` |
   | not written | red | `não respondida` | With the portal's message: `O portal recusou esta linha. A frase é dele, sem tradução nossa:` + the sentence in quotes, mono, red-tinted with a red left bar **[functional]** (verbatim, never translated or interpreted). Without: `O portal não registrou a resposta — releia a ordem e confira as linhas antes de tentar de novo.` |

   Then, when added drafts did not show up on the re-read, amber `{n} linha(s) nova(s) entraram como rascunho mas a releitura da página não as mostrou — o envio PAROU antes de confirmar. Olhe a ordem no portal antes de qualquer nova tentativa.`; footnote (info icon) `Não há “responder de novo” aqui: resposta que não voltou não é resposta que não chegou. Releia a ordem — ou olhe a página no portal.`; `Evidência do serviço:` + path; `Reler esta ordem`.
7. **`Laudo pós-cirúrgico`** + sub `um PDF por ordem` (or `enviado` once this session's upload was accepted):
   - uploaded this session: card (PDF icon, file name) + chip `confirmado na página da ordem` (green), `enviado · não apareceu na página ao reconferir — olhe o portal antes de reenviar` or `enviado · não deu para reconferir — olhe o portal antes de reenviar` (amber);
   - chosen, not sent: card with name, size and remove X (`Tirar do envio {nome}`; hidden while sending or while the confirmation is open);
   - outcome notices: credential refused (danger, credential sentence); refused before writing (amber, the same `O portal de agora discorda da tela…` sentence + cause); no answer, danger `O envio do laudo não respondeu. Não reenvie antes de olhar o portal.` + cause; chooser failure (danger, raw);
   - read-only credential: amber notice with key icon `Sem escopo de escrita, o laudo também não sobe daqui — é a mesma credencial da resposta.`;
   - otherwise (not while it uploads): `Escolher o laudo (PDF)` / `Escolher outro` (paperclip; disabled while the confirmation is open) + hint `O laudo é do ato cirúrgico: um PDF por ordem, e ele não substitui a resposta da cotação.` (none chosen) or `O laudo desta ordem já está na página do portal. Trocar é um envio novo — releia a ordem antes.` (after upload). The laudo remains offered on answered (`PL`/`P`) orders — the send then carries only the laudo.
8. **Send zone** (end of the section): laudo uploading → `Enviando o laudo…` with spinner; answer in flight → `Enviando a resposta ao portal… {n} linha(s), e uma leitura da página depois.` with spinner; otherwise, when the credential may write:
   - resting: primary `Enviar ao portal` (paper-plane; disabled when nothing can go or another write flies) + amber why-sentence — busy: `Há um envio em andamento na ordem {ordem}. Um por vez, sem fila — este fica para quando aquele responder.`; crooked lines: `Falta quantidade ou valor válido em: {linhas}.` (names joined with ` e `); nothing ready: `Preencha Qtd. utilizada e Valor unitário nas linhas que vão na resposta, e/ou escolha o laudo — o Enviar leva tudo de uma vez.`
   - confirming: the **confirmation panel, in place** (accent border and tint) — arrow-up icon + `Enviar ao portal, na ordem {ordem}: {o que}. Confira — é isto que o portal vai receber, e preço é proposta comercial.` ({o que} = `1 linha da cotação` / `{n} linhas da cotação` and/or `o laudo pós-cirúrgico`, joined with ` e `); an inner box listing each travelling line (product + account in mono; new lines as `{produto} · nova`), the laudo line (`o laudo pós-cirúrgico` + file name), a divider and `Total da resposta` with the total in large mono; the note `Só estas linhas viajam. As demais ficam no portal exatamente como o portal as escreveu.` (when lines travel); buttons `Confirmar o envio` (primary; disabled while another write flies) and `Cancelar`. **[functional]** The extract is live: editing a field with the panel open changes what it names, and what it names is what goes.

### Data shown

| Field | Meaning | Rule | Example |
| --- | --- | --- | --- |
| `Código` | The page's own child product id | Selects the row **exactly** when answering; never re-resolved by text | `88412031` |
| `Solic.` / `Autoriz.` | Requester's quantities | Text as sent | `2` / `2` |
| Pre-fill | What the page already holds | `Em Aberto` lines arrive `0` / `0,00`; an answered page shows the answer | `0` · `0,00` |
| A line "travels" | Part of the answer | A page row travels only when its quantity or price differs from the pre-fill; a new row travels when any of its four fields holds something | — |
| Valid quantity / price | What may travel | Quantity: whole number ≥ 1. Price: > 0; `R$` ignored, `1.400,00` and `1400.00` both read; the price travels **as typed** | `2` · `312,50` |
| `Total da linha` / account | Proposal arithmetic, on screen | `{q} un × {valor} = {total}`, pt-BR two decimals | `2 un × 312,50 = 625,00` |
| `Total da resposta` | Sum of the travelling lines | pt-BR two decimals | `3.087,50` |
| New-row resolution | Which portal product the reference means | DE-PARA **suggestion** on screen; the send's live catalogue search is the only authority (exactly one registration or refusal, before any write) | `PB35-06` → `PLACA BLOQUEADA 3,5 MM 6 FUROS` |
| Answer memory | What this app answered | `Respondida por aqui em …` (panel) and the pile's `Resposta` cell | `08/09/2026 · 11:05` |
| Laudo chip | Upload re-check | three-way verdict, as for invoices | `confirmado na página da ordem` |

### States

| State | Trigger | What is drawn |
| --- | --- | --- |
| Opening / too old / read failed / re-reading | As Part C | As Part C (re-reading happens automatically after a successful answer or laudo). |
| `SP` with lines | Live status `SP` | Editable sheet, footer, laudo, send zone. |
| `SP` empty page | No lines on the page | Add invitation; `Adicionar linha`; add-only columns once a line exists. |
| Crooked line | A travelling line lacks id, quantity, price or (new rows) a resolution | The whole send is held; the why-sentence names every crooked line. |
| Empty new line | A new row with all four fields blank | Stays on screen, does not travel, and the footer says so — never silently dropped, never sent. |
| Confirming | `Enviar ao portal` | Confirmation panel in place of the button; `Adicionar linha` and the laudo picker disabled. |
| Answering (sign of life) | `Confirmar o envio` | Sheet read-only, waiting line. Survives closing the subtela; reopening shows the page's pre-fill read-only with the waiting line. |
| Answered | Service answered | Toast; per-line report; the order is re-read → seal `PL` (green) → green "already answered" panel. |
| Refused before writing | The service said no (order no longer `SP`, a line the page does not carry, an unresolved or doubled new line, empty plan) | Amber refusal + cause + `Reler esta ordem`. The laudo is not sent. |
| No answer | Anything else | Danger, no retry offered; the laudo stays chosen, waiting for the operator's decision. |
| Drafts missing | Added lines not visible on the re-read | Amber missing-drafts sentence; the service stopped before confirming. |
| Already answered (`PL`/`P`) | Live status | Read-only sheet + green panel; laudo still offered. |
| Read-only credential | Scopes known, no write | Read-only sheet + scope panel + laudo scope notice; no send zone. |
| Busy elsewhere | Another order's write in flight | Button disabled + sentence naming that order. |

### Actions

| Gesture | Where | Preconditions | Effect | Confirmation | Reversible | Feedback |
| --- | --- | --- | --- | --- | --- | --- |
| Type `Qtd. utilizada` / `Valor unit.` | Page row | Sheet editable, row has a numeric id | Marks the row `alterada`; live totals | — | Type the pre-fill back | Footer sentence, totals |
| `Adicionar linha` | Sheet footer | `SP`, write scope, no answer in flight, panel closed | Adds a blank new row | — | Remove X | — |
| Type a reference, leave the field | New row `Código` | Reference not empty and different from the last one asked | Asks the Spook's DE-PARA (a read, outside the one-write-at-a-time rule); a late answer for an already-corrected reference is ignored | — | — | `Material` state |
| Type `registro ANVISA` | New row | Not resolved by the DE-PARA | Makes the row resolvable; travels as an **override** only when the map did not resolve | — | Yes | — |
| `Escolher o laudo (PDF)` / `Escolher outro` | Laudo | Write scope; panel closed | System chooser (title `Escolha o laudo pós-cirúrgico`, filter `Laudo (PDF)`); one file kept, extra files dropped; a new choice replaces the old | — | `Tirar do envio` | Card |
| `Enviar ao portal` | Send zone | Something ready, nothing crooked, no other write in flight | Opens the confirmation panel | — | `Cancelar` | — |
| `Confirmar o envio` | Panel | No other write in flight | Sends the **answer first** (changed page rows by id + new rows by reference/override); the Spook writes its evidence before any POST, adds drafts, re-reads, then submits once; **then** the laudo. If the answer fails, the send stops and the laudo stays chosen. | This panel **is** the confirmation | **No** — never re-sent by the app | Toast `Cotação {ordem} respondida — o portal respondeu.` / `A resposta da cotação {ordem} não saiu. Olhe a ordem antes de tentar de novo.`; bell `Cotação respondida no portal` / `Resposta de cotação não saiu` (order number only); laudo toast `Laudo enviado na ordem {ordem}.` / `O laudo da ordem {ordem} não subiu. Olhe a ordem antes de tentar de novo.` (no bell for the laudo); pile `Resposta` cell; report; re-read |
| Close (`Esc`, scrim, X) | Anywhere | — | Discards typed values, new rows and the chosen laudo; a job in flight keeps running | None | — | — |

---

## Part E — Messages: _Mensagens do pedido_, the conversation dialog, marking read

### The question

_O que o hospital disse nesta ordem, o que eu respondo — e como tiro a conversa da pilha?_ — read the order's thread as the portal holds it (**reading marks nothing**), answer it (a portal write, behind a confirmation), and mark it read in the portal (an explicit gesture; the Spook exports the conversation before marking).

### Where it sits in the work

One thread component with **two homes**: the last section of the order subtela (Parts C and D, under the flow's own sections), and the **conversation dialog** opened from the messages pile (Part B) — the thread alone with a light identification, without the whole-order read; its order number is the door to the full subtela. The mark-read gesture is what finally takes a conversation out of the messages pile (at the next read), including conversations held there only by the operator's own messages that the hospital has not read. Per the project docs, the messages count covers the whole portal session, not only the configured clients (unverified from this app's code — the Spook decides).

### Anatomy — the thread (`Mensagens do pedido`)

1. **Rule line**: `Mensagens do pedido` + the number of messages (once read) + hairline; on the right, two small buttons:
   - `Marcar como lidas no portal` (envelope-open; tooltip `Marca as mensagens desta conversa como lidas no portal — o Spook exporta a conversa antes de marcar, e ela sai da pilha de mensagens.`) — **subtela only**; drawn when the credential may write (or its scopes are unknown), the thread holds at least one unread message of **any** author (the operator's own included — the portal counts them), and no mark of this order is in flight; disabled while any other portal write flies;
   - `Reler` (arrows; disabled while reading).
2. **Status line**: while marking, `Marcando a conversa como lida no portal…` with spinner; after this session's successful mark, once nothing is unread, a green check `conversa marcada como lida no portal`.
3. **Thread body**:
   - first read: `Lendo a conversa no portal…` with spinner;
   - service too old: amber notice `Este Spook não conhece a leitura da conversa — suba o serviço de novo para ele passar a responder.`;
   - failed, nothing read: danger notice `Não deu para ler a conversa — isto não é "sem mensagens", é o portal não ter respondido o fio.` + the cause in mono;
   - read, empty: dashed box `Nenhuma mensagem nesta conversa — o portal respondeu o fio, e ele veio vazio.`;
   - read: a vertical list drawn **as comments** — a round avatar with initials (first letters of the author's first and last words; `?` when there is no author) beside a bubble holding the author (bold), the stamp (`dd/mm/aaaa · hh:mm` when the portal's stamp parsed, as sent otherwise, omitted when absent), the pill `nova` on unread messages **from the other side only**, then the text (line breaks kept, long words wrap). Messages the Spook judges to be the supplier's (the operator's side) sit on the right with accent-tinted avatar and bubble; all others on the left; bubbles take at most 88% of the width. A later failure (a re-read or a send) adds the same danger notice **under** the list, which stays.
4. **Composer** (credential may write, or scopes unknown): a two-row textarea (vertically resizable), placeholder `Escreva ao hospital…`, disabled while sending; a foot line with, on the left, the verdict of the last send — `enviada · confirmada no fio` (green), `enviada · não apareceu no fio ao reconferir — olhe no portal antes de reenviar` or `enviada · não deu para reconferir — olhe no portal antes de reenviar` (amber) — and on the right `Enviando ao portal…` with spinner while sending, or `Enviar mensagem` (paper-plane; disabled while the draft is blank), which turns in place into `Enviar esta mensagem na conversa da ordem {ordem}? Ela chega ao hospital como está.` + `Cancelar` / `Enviar`. **[functional]** A distracted Enter can never send: sending always passes through the confirmation. Read-only credential: instead of the composer, the footnote `Esta credencial só lê: a conversa aparece, mas responder exige o escopo de escrita (Configurações → Spook).`

### Anatomy — the conversation dialog

- Same scrim as the subtela (clicking it closes). The dialog is centred and capped at **820×720** — smaller than the subtela on purpose: a conversation, not a process — never closer than 14 px to the window edge.
- **Header**, one line: chat-bubbles icon · the **order number as a link** (mono, accent, arrow-out icon, underline on hover; tooltip `Abrir a ordem completa — identificação, cotação, anexos e envio`) · the client's name from the pile row (truncated; omitted when unknown) · spacer · the mark-read control, when the credential may write: while marking, spinner + `Marcando a conversa como lida no portal…`; with unread messages, the accent-outlined button `Marcar como lidas no portal` (envelope-open, same tooltip as above); with nothing unread, the same button disabled and faded to green with a check icon, `Sem mensagens não lidas` (also its tooltip); disabled as well while any portal write flies · the `Esc` key cap + `fecha` · close X (`Fechar`). **[functional]** Mark-read always lives in the title here.
- **Identification block**, fixed (only the thread scrolls) **[functional]** — whoever opens a thread needs to know what it is about: rule `Identificação`; while reading, `lendo a identificação da ordem…` with spinner; could not read, amber question icon + `Não deu para ler a identificação desta ordem — o fio abaixo é o mesmo; abra a ordem completa pelo número para ver tudo.` (never a blank identification); read, a three-column grid of label-over-value pairs: `Cliente`, `Paciente`, `Convênio`, `Médico`, `Status · etapa` (the portal's status label — or its code when there is no label — and the etapa), `Data da cirurgia` (shown `dd/mm/aaaa · hh:mm` when it parses, as sent otherwise); `—` for any empty value.
- **Body** (scrolls): the thread of the previous section, without its own mark-read button (it lives in the header).

### Data shown

| Element | Meaning | Rule | Example |
| --- | --- | --- | --- |
| Rule count | Messages in the thread | Integer, after the first read | `3` |
| Author / initials | Who wrote | As sent; initials derived | `Faturamento Vale Verde` → `FV` |
| Stamp | When | Parsed → local `dd/mm/aaaa · hh:mm`; unparsable → as sent | `09/09/2026 · 16:20` |
| `nova` | Unread, from the other side | Only when unread **and** not the operator's side | — |
| "Has unread" (mark-read offer) | Anything the portal still counts | Any unread message, the operator's own included | — |
| Send verdict | The Spook re-read the thread after sending | three-way: confirmed / not found / could not re-check | `enviada · confirmada no fio` |
| Identification (dialog) | The order in brief, read light | Values as sent, `—` when empty | `Fatura Autorizada · AGUARDANDO NF` |

### States

| State | Trigger | What is drawn |
| --- | --- | --- |
| First read (sign of life) | Thread opened (in the subtela it queues behind the order's own read) | `Lendo a conversa no portal…` |
| Re-read | `Reler` | The list stays; only `Reler` is disabled until the answer. |
| Too old / failed / empty / read | Answers above | The four bodies of the anatomy — failure never drawn as "no messages". |
| Sending | `Enviar` confirmed | Textarea disabled, `Enviando ao portal…`. |
| Sent | Spook answered | Toast, draft cleared, verdict line, thread re-read so the message appears. |
| Send failed | No answer, too old, credential refused | Danger notice under the thread (the credential sentence when refused), toast, **draft kept**. |
| Marking | Mark-read clicked | Spinner line (subtela) or spinner in the header (dialog); the pile row shows `marcando…`. |
| Marked | Spook answered | Toast with the count; thread re-read, `nova` pills fall; green line (subtela) or faded `Sem mensagens não lidas` (dialog); pile row `lida em …`. |
| Mark failed | Anything else | Failure toast only; the journal holds the detail. |
| Read-only credential | Scopes known, no write | No composer (footnote instead), no mark-read. |
| A sweep in flight | Any mark-read (this or the bulk one) | A message send is not performed and nothing on screen says so (see Open questions). |
| Identification (dialog) | Light read | Reading / could-not-read / pairs. |

### Actions

| Gesture | Where | Preconditions | Effect | Confirmation | Reversible | Feedback |
| --- | --- | --- | --- | --- | --- | --- |
| `Reler` | Thread rule | Not reading | Reads the thread again; marks nothing | — | n/a | List refreshed |
| `Enviar mensagem` → `Enviar` | Composer | Draft not blank; credential may write; not already sending | Posts one message to the hospital in this order's conversation; the Spook re-reads the thread to verify; the thread is re-read here | In place: `Enviar esta mensagem na conversa da ordem {ordem}? Ela chega ao hospital como está.` | **No** | Toast `Mensagem enviada ao portal.` / `A mensagem não saiu — veja a conversa antes de tentar de novo.`; verdict line; the journal records that a message left, never its text; no bell |
| `Marcar como lidas no portal` | Subtela rule / dialog header | Credential may write; unread present; no portal write in flight | The Spook exports this conversation first, then marks read what it exported; the thread is re-read | **None** (one conversation, explicit gesture) | No | Toast `{n} mensagens marcadas como lidas no portal.` or `Nada para marcar — nenhuma mensagem não lida ao alcance.`; failure toast `A varredura não respondeu — nada garantido. O journal tem o registro.`; pile `Lida` cell; journal counts |
| Order number link | Dialog header | — | Closes the conversation and opens the full subtela for that order (Parts C/D), which reads the order live | — | — | Subtela opening state |
| Close — `Esc`, scrim, X | Dialog / subtela | — | Closes; the draft is lost | None | — | — |
| `Varrer mensagens` | Messages pile | See Part B | Every unread conversation | Inline, with the count | No | See Part B |

---

## Business rules

Shared by the five parts. Decision numbers refer to the Bridgr Pro repository's `.agents/decisions/`.

1. **Absence looks like absence.** A count nobody could read is `—`, never `0`; while nobody has asked, the screen asserts nothing; a stopped service dims the cards and says the read did not happen — not that there is no work; an unconfigured service draws no number at all. A zero that is really ignorance tells the operator there is no work where there may be plenty. (Decision 006 § 2 "a fifth state"; the inherited rule "absence must look like absence".)
2. **A partial read announces itself** with the number of clients that did not answer, on every card it affects. A partial sum that looks complete is how the panel once read a small fraction of the real queue.
3. **Read on demand; the only clock is the operator's.** Each read walks every client's queue in the portal. The automatic read is off by default, cannot run more often than every 5 minutes, runs the same read as `Reler`, and skips silently unless the Spook is reachable, nothing else is reading or writing to the portal, and the app is not locked; a manual read resets it. Its news rings **once**, only for piles that grew, only when the operator is not on the panel, with counts only. (Decision 032.)
4. **The reading is kept with its stamp, and a failure never erases it.** Closing the app must not cost minutes of re-reading, and numbers without a "when" would be drawn as current however old; a failed read (manual or automatic) leaves the previous numbers on screen and on disk, beside the failure.
5. **Hiding is the operator's decision, kept apart from the portal's answer.** Keyed by the portal's order number, stored separately from the reading, surviving every re-read and restart, reversible, idempotent; an order that leaves the portal and returns stays hidden. In one file, "re-read the portal" would silently mean "forget what I decided".
6. **A count without rows is an old service, not "none".** Saying "no conversations" under a number that says otherwise is the screen contradicting itself.
7. **The order is read live, never from the snapshot.** The panel answers "how much work", minutes or hours ago; whoever opens an order is about to act on it, and only what the portal says now counts — including whether the order may still receive an invoice. (Decision 025 § 4.)
8. **What the portal wrote travels as written.** Order dates are label text (one portal order carried a wrong year — fixing it would be inventing), quantities and prices are text, the seal is `status · etapa` verbatim, and a portal refusal is quoted verbatim, never translated or interpreted. Only the dates this app itself converts (pile cells, stamps) follow the house `dd/mm/aaaa` rule.
9. **What is already attached is read before anything is offered, and "could not read" is never "nothing attached".** The two sentences send the person to opposite decisions, and one of them uploads a second copy of the same invoice. (Decision 025 § 3.)
10. **Blocks come before the gesture, one per cause, with the cure.** No write scope (cure: re-issue the credential in Settings), invoice already received (green — nothing wrong), status not `FA`, no upload frame; for quotations: no write scope, already answered. Choosing files or typing prices only to be refused afterwards is the operator's work thrown away. (Decisions 025 § 4, 029 § 4.)
11. **The credential's scope is asked before a write is offered.** Scopes are minted into the credential; one issued when the app only read stays read-only forever. An empty scope list means **not known** — then the gesture is offered and the service is the one that refuses. (Decision 025 § 1.)
12. **One portal write at a time, app-wide, refused and said — never queued.** Invoice send, quotation answer, laudo upload and mark-read sweeps share one lock (a message send is also held while a sweep flies). The screen refuses before the click and names the order in the air; the bridge keeps its own backstop. A queue of portal writes would be "no button processes the whole queue" broken under another name. (Decisions 025 § 7/7b, 029 § 4, 031.)
13. **Never re-send, never offer "try again".** A write whose answer never arrived is not a write that did not happen: the portal may hold the file, the price or the message. The failure sentences tell the person to look at the portal first. **`unknown` verification is a re-read that did not happen — not a failed write**, and it is written out in full because it is the verdict most likely to be misread. (Decision 025 § 5; the inherited "resume, never redo".)
14. **Evidence before the POST.** The Spook writes its evidence artifact before the first write to the portal; the report shows its path. (Decisions 025 § 6, 029 § 4.)
15. **Writes are jobs that outlive the subtela.** Closing an order never cancels or forgets a write; reopening finds it where it stands. Outcomes are said in two channels where a write can end with the subtela closed: toast + bell for invoice sends and quotation answers (bell detail = the order number only); toast only for the laudo, messages and sweeps; sweep counts and message sends go to the journal. (Decisions 025 § 7b, 029, 031 § 3.)
16. **This app's memory is appended, never replaced, and is never the portal's truth.** It records what this app sent, answered and marked read — two sends on one order are two facts — and it is what keeps a just-handled order from looking untouched until the next read (`Nota`, `Resposta`, `Lida`, the send card). (Decision 025 § 8.)
17. **Bytes, never paths.** Files chosen or located are held in memory on the native side; the screen only holds an id; closing the subtela gives up the basket; empty files are refused; the same file never enters twice. (Decision 025 § 6.)
18. **Found by number means one rule shared with MailBridgr.** Lists, ranges (at most 100), split digits; a number matches a file **name** only as a whole digit group; PDF and XML; only the monitor's own note folders are searched. The number that found a file is a fact; each number without a file is said with the folder searched; locating never sends. (Decision 028.)
19. **A suggestion is never an attachment.** Only notes whose own text names the order's patient, at most five; accepting goes through the same locate path; no notes, no block. (Decision 028 § 6.)
20. **Documents come down on demand, one per gesture, and open in place.** Downloading is a portal read; most openings are to check a material or send a note, not to read a guia. Only files the service reported can be opened. "Not found here" is an observation, never a diagnosis. (Decision 025 §§ 8b, 9.)
21. **The quotation answer carries only what the operator changed, and nothing half-done.** Rows are chosen by the page's own product id; a changed row with a bad id, quantity or price holds the whole send and names itself; a completely empty new row stays and is said; the DE-PARA on screen is a suggestion and the send's live search is the only authority (a typed ANVISA travels only as an override); "not in the map" and "could not ask" are different sentences; the answer goes first and a failure stops the laudo. (Decisions 029, 030.)
22. **Reading a conversation marks nothing; marking is explicit.** A failed thread read is never "no messages"; replying is a write behind the scope and an in-place confirmation, and its verdict comes from re-reading the thread; marking read exports before marking, never marks a conversation this login cannot reach, and never names those conversations on screen. (Decisions 025 § 8b, 031.)
23. **No patient data leaves through the bell or the journal.** Bell details carry an order number or counts; the journal records that a message left, never what it said.

## Must survive the redesign

- The five panel truths: `—` for unknown (never 0) with its chip and note; the partial chip with the count of silent clients; the stopped banner and dimmed cards; the "nobody asked yet" state; no numbers while unconfigured.
- The read stamp, always beside the numbers, with its age and the clock's interval; `Reler` disabled while unconfigured, stopped or already reading; the sign of life drawn before the read.
- Failures at the top of the body, never over the numbers; the kept numbers stay under a failure.
- The operator's clock as the only automatic read, and its once-only, counts-only, off-panel news notice.
- Unknown-pile and rows-missing panels, visibly different from the empty pile.
- Hide / bring back as a reversible decision; the hidden count and the show-hidden toggle in the list header; hidden rows flagged when shown.
- The memory columns `Nota`, `Resposta`, `Lida` (in flight, then done with stamp and confirmation).
- One door per row, only when there is an order number; messages open the conversation, whose order number opens the full subtela.
- The subtela: opens at once saying it is reading; live read; header with the order number and the verbatim seal; `Esc`, scrim and X all close; 1440×900 cap plus the fill-window toggle; work and documents side by side, each scrolling on its own, documents on the right, divider draggable within 38–72%.
- Identification label and value always together in one cell.
- Portal text shown as written (dates, quantities, prices, statuses, refusals).
- `Notas já anexadas` above the attach area; "could not read" distinct from "none"; the green already-received panel; every block before its gesture, with its cure.
- The basket: a number per file, suggested vs found (`pelo número`); one in-place confirmation naming the order and every file with its NF; one waiting line, never per-file progress; the busy sentence naming the order in the air; the why-sentence naming what is missing.
- The per-file report with the three-way verdict, no retry, the evidence path, and `Reler esta ordem`.
- Locate: one amber line per missing number with the folder; the no-number sentence; disabled with its reason when the monitor is not configured.
- Suggestions: separate from the basket, never auto-attached, gone once in the basket or attached, absent when empty.
- Documents: `Baixar e ver` per file with its own sign of life; the not-found observation; in-place viewer; the system viewer as the second door; zoom for images only.
- The answer sheet: two operator columns; "changed" said in words, not only colour; add-only projection; read-only when blocked or in flight; the travel sentence (including the empty new row); the live confirmation panel with extract, total and note; answer first, laudo second; the per-line report with the portal's sentence quoted verbatim; the missing-drafts warning; no "answer again".
- DE-PARA row states, "not in the map" distinct from "could not ask".
- The laudo: one PDF per order, its own verdict chip, still offered on answered orders.
- Messages: reading marks nothing; failure distinct from empty; `nova` only on the other side's unread; composer behind confirmation and scope; the verdict after sending; mark-read explicit (single conversation without confirmation, bulk with an inline confirmation naming the count); no per-conversation errors on screen.
- The toasts' and notices' wording, and which outcome rings the bell.

## Free to redesign

- Card look: icons, number size and typeface, chip shapes, where the note sits, the picked-card marker (as long as the open pile is identifiable).
- Header composition: where the badge, subtitle, stamp and `Reler` sit (the stamp stays next to the numbers or the gesture).
- Banner styling, empty-state illustrations and icons.
- The list header arrangement and the sweep row's position (the confirmation needs room for a long question).
- The subtela's frame styling, header layout, section-rule style, block-panel look, card rows for files, report cards, confirmation panel styling (it must stay in place, never a second modal).
- Divider look, documents list rows, viewer bar and zoom controls.
- Seal pill style (the text stays verbatim; see the proposal on its colours).
- Thread bubbles, avatars, alignment, the composer's shape.
- Spacing, density behaviour, and the narrow-window arrangement.

## Sample content for mockups

All fictitious (the cast of [sample data](../06-sample-data.md)). Operator: Camila Duarte Rocha. "Today" is 10/09/2026.

**Panel** — stamp `lido há 9 min · relê a cada 15 min`; cards `3` Prontas para nota · `2` Cotações a responder · `4` Mensagens não lidas. Partial variant: every card with `incompleto` and `Incompleto: 1 cliente(s) não responderam.` Unknown variant: messages card `—` in amber, `não sabido`, `Não conseguiu perguntar ao portal — a leitura falhou.` Bell after an automatic read: `Novidades no portal` with the detail `+2 cotações · +1 prontas para nota`.

**Prontas para nota**

| Ordem | Cliente | Paciente | Cirurgia | Convênio | Nota |
| --- | --- | --- | --- | --- | --- |
| 7100231 | Hospital Vale Verde | MARTA LEMOS FERREIRA | 02/09/2026 | VIDA PLENA SAÚDE | enviada em 08/09/2026 · 10:42 · confirmada no portal |
| 7100244 | Hospital Santa Aurora | JOÃO CARLOS PRADO | 03/09/2026 | AMPARO SAÚDE | |
| 7100258 | Instituto Ortopédico Horizonte | LUÍSA ANDRADE REIS | 04/09/2026 | SUS | enviando… |

**Cotações a responder**

| Ordem | Cliente | Paciente | Convênio | Data-limite | Esperando | Resposta |
| --- | --- | --- | --- | --- | --- | --- |
| 7100262 | Hospital Infantil Sol Nascente | PEDRO HENRIQUE SALLES | PARTICULAR | 05/09/2026 | 5 d 14 h | |
| 7100275 | Hospital Vale Verde | BEATRIZ MONTEIRO | AMPARO SAÚDE | 12/09/2026 | 0 h | respondida em 09/09/2026 · 11:05 · sem confirmação |

**Mensagens não lidas**

| Cliente | Ordem | Novas | Última mensagem | Lida |
| --- | --- | --- | --- | --- |
| Hospital Vale Verde | 7100231 | 2 | 09/09/2026 · 16:20 | |
| Hospital Santa Aurora | 7100244 | 1 | 08/09/2026 · 09:12 | lida em 09/09/2026 · 17:02 |
| Hospital Infantil Sol Nascente | — | 1 | 07/09/2026 · 14:03 | (no door on this row) |

**FA subtela, order 7100244** — header `7100244` · `Hospital Santa Aurora · JOÃO CARLOS PRADO` · seal `FA · AGUARDANDO NF`. Identification: `Cliente` Hospital Santa Aurora · `Paciente` JOÃO CARLOS PRADO · `Data da cirurgia` 03/09/2026 · `Convênio` AMPARO SAÚDE · `Médico` DRA. HELENA CASTRO · `Atendimento` 2026004602 · `Valor da ordem` R$ 3.420,00.

| Descrição | ANVISA | Solicitado | Autorizado | Utilizado | Total | Status |
| --- | --- | --- | --- | --- | --- | --- |
| PARAFUSO CORTICAL 3,5 MM X 24 MM TITÂNIO | 80000000011 | 6 | 6 | 6 | 1.260,00 | (portal text) |
| PLACA BLOQUEADA 3,5 MM 6 FUROS | 80000000022 | 1 | 1 | 1 | 2.160,00 | (portal text) |

(The material `Status` is the portal's own text; the only value this app's code names is `Em Aberto` — unverified what an `FA` order shows there.)

`Notas já anexadas`: the dashed empty sentence. Suggestion box: `100246` · `uso 03/09/2026` · `3.420,00` · `Anexar`. Basket after locating `100246`: `NF_100246.pdf` · `184 kB` · `pelo número` · `Nº da NF` 100246; `NF_100246.xml` · `12 kB` · `pelo número` · 100246. A missing number: `Nenhum arquivo com 100247 em /dados/notas/NF-e 2026.` Confirmation: `Enviar ao portal, na ordem 7100244: NF_100246.pdf como NF 100246 e NF_100246.xml como NF 100246.` Report: `NF_100246.pdf` · `NF 100246` · `enviada · confirmada no portal` (green); `NF_100246.xml` · `NF 100246` · `enviada · não deu para conferir — isto não é envio que falhou, e reenviar aqui é o que duplica a nota` (amber); `Evidência do serviço: tmp/evidence/upload-7100244.json`. Documents column: `Guia de autorização.pdf` (baixado, on display in the viewer), `Laudo cirúrgico.pdf` (`Baixar e ver`), `etiquetas_sala3.jpg` (`baixando…`), `Pedido.xml`.

**SP subtela, order 7100262** — header `7100262` · `Hospital Infantil Sol Nascente · PEDRO HENRIQUE SALLES` · seal `SP`. Identification: `Data-limite da cotação` 05/09/2026 · `Convênio` PARTICULAR · `Médico` DR. RAFAEL NOGUEIRA · `Atendimento` 2026004633 · `Valor solicitado` R$ 1.840,00.

| Código | Material | ANVISA | Situação | Solic. | Autoriz. | Qtd. utilizada | Valor unit. (R$) | Total da linha | Na resposta |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 88412031 | PARAFUSO CORTICAL 3,5 MM X 24 MM TITÂNIO | 80000000011 | Em Aberto | 4 | 4 | [4] | [312,50] | 1.250,00 | alterada |
| 88412032 | CIMENTO ÓSSEO 40 G | 80000000033 | Em Aberto | 1 | 1 | [0] | [0,00] | — | |
| [FK16] | FIO DE KIRSCHNER 1,6 MM | 80000000044 | + nova | | | [2] | [45,00] | 90,00 | × |

(`[ ]` = an input field.) Footer: `2 linhas viajam ao portal · 1 delas é nova · as outras 1 ficam como estão` · `Total da resposta` 1.340,00. Laudo: `laudo_pos_7100262.pdf` · `96 kB`. Confirmation panel: `Enviar ao portal, na ordem 7100262: 2 linhas da cotação e o laudo pós-cirúrgico. Confira — é isto que o portal vai receber, e preço é proposta comercial.` — `PARAFUSO CORTICAL 3,5 MM X 24 MM TITÂNIO` `4 un × 312,50 = 1.250,00` · `FIO DE KIRSCHNER 1,6 MM · nova` `2 un × 45,00 = 90,00` · `o laudo pós-cirúrgico` `laudo_pos_7100262.pdf` · `Total da resposta` `1.340,00`. DE-PARA variants for a new line: `PB35-06` → `PLACA BLOQUEADA 3,5 MM 6 FUROS · casamento fraco no DE-PARA, confira`; `CO-40` → `não está no DE-PARA — informe o registro ANVISA ao lado`. A refused report line: `CIMENTO ÓSSEO 40 G` · `não respondida` · `O portal recusou esta linha. A frase é dele, sem tradução nossa:` · `“Quantidade utilizada maior que a autorizada.”`

**Thread, order 7100231** — `Faturamento Vale Verde` (`FV`, left) · `09/09/2026 · 16:20` · `nova`: "Bom dia! Pode confirmar o lote do parafuso usado na sala 3?" — `Camila Duarte Rocha` (`CR`, right, accent) · `09/09/2026 · 16:41`: "Lote 451288, conforme a etiqueta anexada ao pedido." — `Faturamento Vale Verde` · `09/09/2026 · 17:05` · `nova`: "Obrigado. Seguimos com a autorização." After a reply: `enviada · confirmada no fio`.

**Conversation dialog, order 7100231** — header `7100231` (link) · `Hospital Vale Verde` · `Marcar como lidas no portal`; identification: `Cliente` Hospital Vale Verde · `Paciente` MARTA LEMOS FERREIRA · `Convênio` VIDA PLENA SAÚDE · `Médico` DR. RAFAEL NOGUEIRA · `Status · etapa` Fatura Autorizada · AGUARDANDO NF · `Data da cirurgia` 02/09/2026 · 07:30.

**Toasts** — `Notas enviadas na ordem 7100244 — o portal respondeu.` · `Cotação 7100262 respondida — o portal respondeu.` · `Laudo enviado na ordem 7100262.` · `2 mensagens marcadas como lidas no portal.`

## Design opportunities

Proposals, not decisions — each needs the operator's call.

1. **Let the screen describe itself truthfully.** The badge `fatia 1 · só leitura`, the subtitle ("responder cotação e subir nota continuam lá dentro"), the footer ("nada aqui muda sozinho … Nenhuma ação sai desta tela") and the list note ("somente leitura · o portal responde e fatura, não esta tela") date from the first slice; today the subtela sends invoices, answers quotations, uploads the laudo, replies and marks read, and the operator's clock re-reads on its own. A screen that describes itself wrongly teaches distrust of its other sentences. Proposal: one capability line (reads on demand or on the operator's clock; writes only from inside an order, always confirmed) — wording is the operator's.
2. **A stamp that keeps ageing and speaks hours.** It counts minutes only (a day-old reading says `lido há 1440 min`) and is not redrawn while the screen stays open; decision 032 mentioned showing the next automatic read, which is not drawn. Proposal: `lido há 2 h 15 min` / `lido ontem · 17:40`, live ageing, and the next automatic read when the clock is on.
3. **One visible "write in flight".** Only one portal write can fly app-wide, yet it shows only in a pile cell, as a sentence inside another order's subtela, or in the sweep row. Proposal: a small persistent chip in the panel and subtela headers naming the order and the kind of write in the air.
4. **One verdict grammar.** The same outcomes (confirmed · not found · could not re-check · failed) appear in the invoice report, the laudo chip, the message verdict, the answer report and the pile cells, each styled locally. Proposal: one verdict element (icon + tone + the existing sentence), so "could not re-check" always reads "look before re-sending", never "failed".
5. **Seal colours that do not over-promise.** Every status other than `FA` and `SP` is green, including statuses the app knows nothing about. Proposal: green only for known finished states (e.g. `PL`, `P`), neutral for anything unrecognised — absence looking like absence, applied to status.
6. **Compact cards while a pile is open.** The tall cards keep eating height above the list (tight at 768 px). Proposal: collapse them into a tab strip (number, label, chip) while a pile is open, keeping the `—`/partial/unknown semantics.
7. **Say when a card excludes hidden rows.** The card counts non-hidden rows without saying so (the operator removed that line on 21/08/2026 to keep the cards clean). Proposal: a discreet eye-slash marker only when something is hidden — listed to be re-asked, not decided.
8. **Protect unsent work on close.** `Esc` (even while typing), the scrim and X close at once and discard the basket, typed prices, new lines, the chosen laudo and a message draft; an open inline confirmation does not intercept `Esc`. Proposal: `Esc` first cancels an open confirmation or leaves the focused field; closing with unsent work asks once.
9. **A way through the long FA column.** Identification, materials (up to 300 px), attached invoices, the attach area and messages form one long scroll. Proposal: sticky section rules or a small index, keeping the order (attached always above attach).
10. **Documents column at narrow widths.** At the 1024×768 minimum it is about 390 px wide. Proposal: a collapsible documents rail under a width threshold.
11. **Copy the order number.** The operator cross-references the portal by number; `Copiar o nº da ordem` / `Copiado` exist in the strings and are drawn nowhere. Proposal: a copy affordance on the subtela header and the pile's `Ordem` cell.
12. **Make `Esperando` read as urgency** — once its meaning is confirmed (Open questions): overdue in amber, future deadlines as "faltam N d".
13. **Thread readability.** Day separators and an unread divider, keeping the `nova` pill.
14. **Name the messages pile's rows.** Its counts say `ordem`/`ordens` though the rows are conversations; `conversa`/`conversas` would need new strings.
15. **A proper sidebar home.** The built screen still sits under `Ideias futuras`; a portals / outside-the-conveyor group would say what it is (coordinate with [shell](../04-shell.md)).
16. **"Only the laudo" as its own state.** On an already-answered order the send zone offers the laudo with the same button and panel as a full answer; a smaller "send only the laudo" affordance would keep an answered order from looking re-answerable.

## Open questions

1. Is the first-slice copy (badge, subtitle, footer, list note) meant to stay (proposal 1)?
2. What does the Spook's `unread_messages` count — messages or conversations? With rows present the card shows non-hidden **conversations**; the bulk sweep's question calls the Spook's number "mensagens". Unverified from this app's code.
3. `Esperando` is computed from `Data-limite` (time past the deadline; `0 h` while it is in the future). Intended, or should it be time since the request?
4. On an order still in `FA` with an upload frame, the green "O portal já recebeu a nota desta ordem" panel and the basket appear together whenever the portal lists an attached file or this app ever sent to that order (that memory never expires). Intended (a second invoice), or should the basket hide?
5. Should the card number disclose hidden rows (proposal 7)?
6. Which statuses deserve the green seal (proposal 5)?
7. What is a conversation row without an order number in the portal? Today it shows `—` and has no door; nothing is invented meanwhile (decision 031 § 5).
8. The laudo chooser is labelled `Laudo (PDF)` but also accepts XML files — intended?
9. Should `Esc` close a dialog that holds unsent work (proposal 8)?
10. The DE-PARA failure sentence says `saia e volte ao campo para tentar de novo`, but leaving the field again with the same reference does not ask again (only a changed reference does). Which is right — the sentence or the behaviour?
11. After a failed order read, the documents column keeps saying `Lendo a ordem no portal…`. Should it say the read failed?
12. While a mark-read sweep is in flight, the message `Enviar`, and during the bulk sweep also the invoice send and the quotation `Confirmar o envio`, can be clicked and do nothing, with no sentence. What should they say?
13. The conversation dialog's `Data da cirurgia` is parsed as a date-time and shown with an hour; if the portal sends a bare date, the shown day may shift by the timezone (unverified — depends on the Spook's format). Should this field be shown as the portal wrote it, like the subtela does?

