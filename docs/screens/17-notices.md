# Avisos — the bell (Central de avisos) and `/notices`

> Where: the bell in the titlebar, between the search field and the user button; its panel's footer "Ver todos os avisos" opens `/notices`, a screen with no sidebar item (the titlebar reads "Bridgr Pro · Avisos") · Status: built · Code: `src/lib/components/NoticeCenter.svelte`, `src/routes/notices/+page.svelte`, `src/lib/notices.svelte.ts` (`NOTICE_KINDS` and the store), `src/lib/components/FilterField.svelte`, `src/lib/ipc.ts` (`Notice`, `readNotices`, `setNoticeRead`, `archiveNotice`, `setNoticeMuted`, `markNoticesRead`), `src-tauri/src/notices.rs`, `src-tauri/src/lib.rs` (producers), `src/routes/+layout.svelte` (refresh points, `idle_autolock`), `src/lib/opmenexo.svelte.ts` (`opmenexo_news`), `src/lib/i18n/locales/pt-BR.ts` (`notices`)

## The question

- **The bell's panel**: what is asking for me right now? It lists pending notices only.
- **The Avisos screen**: "O que pediu você, e o que aconteceu sozinho?" (`question.notices`; the sentence is not drawn). It holds the last 30 days of subjects that asked for a human decision or happened without a hand, each with what the app did and what it did not do.

## Where it sits in the work

The app has two feedback channels (decision 010):

- **Toasts** are feedback on an action (five seconds, never stored); see [04-shell](../04-shell.md).
- **Notices** are pending subjects, and only two kinds of thing earn one: what asks for a human decision, and what happened without a hand. Examples: a service that did not rise with the app, an extraction that finished or failed while the operator was elsewhere, the outcome of a portal send, the window locking itself.

Every notice leads to the screen where its subject lives and resolves nothing on its own. The bell belongs to the window chrome and is present on every screen except the first-assembly one. `/notices` is reached only from the panel's footer.

## Anatomy

### A. The bell

- A small button (27 × 25 px today) with a bell icon. Its tooltip and accessible name are "Central de avisos", and it looks pressed while the panel is open.
- A **badge** over the icon's top-right corner, in danger red with the titlebar's background as a ring and a white number. It counts unread notices whose kind is not silenced, is hidden at 0, and has no upper cap.
- It is absent on the first-assembly screen and disabled while the window is locked; the panel closes when the lock falls.

### B. The panel ("Central de avisos")

A dropdown about 352 px wide, anchored under the bell and aligned to its right.

1. **Head.** The title "Central de avisos" and a summary line under it:
   - "<N> aviso não lido · nada foi decidido sozinho" or "<N> avisos não lidos · nada foi decidido sozinho";
   - "nada pendente por aqui" when nothing is pending.

   At the right sits a small "Marcar todos como lidos", disabled when nothing is pending.
2. **List** (scrolls past 340 px). It holds **pending notices only**: unread and not silenced, newest first. Each item is one button, made of:
   - the unread dot (accent);
   - a tone tile with the kind's icon;
   - the title (bold), with the elapsed time at the right in tiny monospace;
   - the text;
   - the detail, only for the kinds that draw one;
   - a target chip "→ <TARGET>" in uppercase.

   With nothing pending, the list shows "Nada pendente por aqui. O que já foi lido fica em Avisos."
3. **Footer.** "Ver todos os avisos" (bell icon) opens `/notices`.

### C. The Avisos screen (`/notices`)

1. **Header.** The title "Avisos" with the summary under it (see Data shown). At the right, "Marcar todos como lidos" (checks icon), disabled when nothing on the screen is unread.
2. **Toolbar** (wraps onto more lines when narrow):
   - the screen's search field: placeholder "Procurar nos avisos…", with `Ctrl ⇧ K` ([05-components](../05-components.md) › FilterField);
   - three segmented tabs, "Todos", "Não lidos" and "Pedem você". The count in parentheses is appended only when it is above 0: "Não lidos (3)", "Pedem você (4)";
   - the source chips: a pill with a coloured dot and the source's name; they toggle, several can be on at once;
   - the switch "Agrupar repetições", on by default, with the tooltip "Juntar avisos idênticos numa linha só".
3. **Body.** A list pane that scrolls, and a **detail panel 344 px wide** at the right on wide windows. **[functional]** The detail is where the "o que o app fez / o que não fez" narration is read.

**The list pane:**

- **Day sections**, newest day first. Each has a sticky header drawn in uppercase: the label ("HOJE · 10/09/2026", "ONTEM · 09/09/2026", or the weekday, such as "TERÇA-FEIRA · 08/09/2026"), then "<N> aviso"/"<N> avisos", then a hairline.
- **Rows** are cards 6 px apart, newest first within the day. A row holds:
  - the unread dot and the tone tile;
  - the title, bold while unread, followed by a badge: "silenciado" when the kind is silenced, otherwise "<N>×" in the tone's colour when the row folds more than one notice;
  - the text, and the detail for the kinds that draw one (monospace);
  - a meta line: "→ <TARGET>" in uppercase, then a source dot and the source's name.
- **The right side of a row**: the time ("hh:mm", of the newest occurrence) and three icon actions, visible on hover or keyboard focus:
  - the read toggle: envelope-open for "Marcar como lido", envelope for "Marcar como não lido";
  - the silence toggle: bell-slash for "Silenciar este tipo", bell for "Voltar a avisar";
  - archive, labelled "Arquivar".
- **A folded row** also has a footer: a caret with "aconteceu <N> vezes neste dia", which expands into one small chip per occurrence ("hh:mm"). Expanded, the footer reads "esconder os horários".
- **Row looks**: unread has an accent-tinted border; the selected row has an accent border and tint; a silenced row is drawn at about 55 % opacity; hover strengthens the border.
- **Under the list**: the empty face when nothing matches (see States), then a divider carrying "avisos ficam 30 dias", then a button-like link, "Ver o histórico completo nos Logs" (list-magnifying-glass icon), that opens [Logs](11-logs.md).

**The detail panel** shows the selected row, or the first row when none is selected. From top to bottom:

- a 34 px tone tile, the title in the heading face, and two badges, both uppercase: the tone word, and "não lido" while unread;
- the text, then the detail (for the kinds that draw one);
- the meta rows, with uppercase labels:

  | Label | Value |
  | --- | --- |
  | "De onde veio" | the source |
  | "Quando" | "<day label>, hh:mm", e.g. "Hoje · 10/09/2026, 14:34" |
  | "Vezes" | "uma vez", or "<N> vezes neste dia" |
  | "Decidido pelo app" | "nada", in green |

- a box with the kicker "o que o app fez" and two lines: an arrow icon with what the app did, and a prohibit icon with what it did not do;
- when the row folds more than one notice, a "quando aconteceu" timeline: one mark and "hh:mm" per occurrence, newest first. The newest mark takes the tone's colour and reads "mais recente"; the oldest reads "primeira vez".
- **Foot**: a primary button "Abrir <target>" (e.g. "Abrir Configurações · NFe Pack"), then the same three toggles as the row, with text labels.

## Data shown

### Every kind of notice that exists today

The stored identifier is in the first column. The tone word is the detail panel's badge. The last column says who posts the notice and when.

| Kind | Title (verbatim) | Target → opens | Source | Tone | Icon | Pedem você | Detail drawn | Posted |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `idle_autolock` | "A janela se trancou sozinha" | "Perfil" → `/profile` | Sistema | accent ("chegou coisa") | lock | no | — | by the app, when the idle timer locks the window (PIN active) |
| `pin_reset_by_code` | "O PIN foi desligado pelo código" | "Perfil" → `/profile` | Sistema | warn ("pede atenção") | envelope-open | no | — | when the locked window is opened with the MailBridgr code |
| `medlens_autostart_failed` | "O MedLens não subiu com o app" | "Configurações · MedLens" → `/settings?section=medlens` | MedLens | danger ("parou") | eye | yes | — | at launch, when the configured rise fails |
| `fantasma_autostart_failed` | "O Fantasma não subiu com o app" | "Configurações · Fantasma" → `/settings?section=fantasma` | Fantasma | danger | robot | yes | — | at launch, when the configured rise fails |
| `nfepack_autostart_failed` | "O NFePackMonitor não subiu com o app" | "Configurações · NFe Pack" → `/settings?section=nfepack` | NFe Pack | danger | receipt | yes | — | at launch, when the configured rise fails |
| `nfepack_errors` | "O monitor registrou erros novos" | "NFePackMonitor" → `/nfe-pack-monitor` | NFe Pack | danger | receipt | yes | **yes**: how much the error count grew, e.g. "+3" | by the monitor's status check, when its log gained error lines ([NFePackMonitor](12-nfe-pack-monitor.md)) |
| `dock_pieces_extracted` | "Extração concluída" | "Revisão" → `/review` | Esteira | accent | tray | no | no (stored: the piece count) | when an extraction run ends with pieces read |
| `dock_extraction_failed` | "Uma extração falhou" | "Triagem" → `/triage` | Esteira | danger | tray | yes | no (stored: the piece count) | when pieces fail extraction |
| `opmenexo_invoices_sent` | "Notas enviadas ao portal" | "Opmenexo" → `/opmenexo` | Spook | accent | paper-plane-tilt | no | no (stored: the order number) | when an invoice send to the portal answers |
| `opmenexo_invoices_failed` | "Envio de notas não respondeu" | "Opmenexo" → `/opmenexo` | Spook | danger | paper-plane-tilt | yes | no (stored: the order number) | when a send ends unanswered or refused |
| `opmenexo_quotation_answered` | "Cotação respondida no portal" | "Opmenexo" → `/opmenexo` | Spook | accent | currency-circle-dollar | no | no (stored: the order number) | when a quotation answer goes through |
| `opmenexo_quotation_failed` | "Resposta de cotação não saiu" | "Opmenexo" → `/opmenexo` | Spook | danger | currency-circle-dollar | yes | no (stored: the order number) | when a quotation answer ends unconfirmed or refused |
| `opmenexo_news` | "Novidades no portal" | "Opmenexo" → `/opmenexo` | Spook | accent | bell-ringing | yes | **yes**: what grew, e.g. "+2 cotações · +3 mensagens" | by the automatic portal re-read (the clock set in Configurações › Spook), when a count grew, and only while the operator is not on the OPMENEXO screen |

The `opmenexo_news` detail joins with " · " only the pieces that grew, among "+{n} cotações", "+{n} prontas para nota" and "+{n} mensagens". The words do not change when n is 1.

**Text and narration of each kind (verbatim):**

| Kind | Text | "o que o app fez" | what it did not do |
| --- | --- | --- | --- |
| `idle_autolock` | "Ninguém mexeu na estação pelo tempo configurado e o PIN assumiu. Nada foi fechado nem perdido." | "Trancou a janela e manteve tudo aberto por baixo." | "Nada foi fechado nem perdido — nenhum trabalho em andamento parou." |
| `pin_reset_by_code` | "A janela destrancou com o código do MailBridgr e o PIN antigo saiu de cena. Grave um novo quando quiser." | "Destrancou a janela com o código e desligou o PIN antigo." | "Não gravou um PIN novo — gravar um é gesto seu, no Perfil." |
| `medlens_autostart_failed` | "O serviço estava configurado para subir sozinho e não veio. A esteira fica sem os olhos até alguém olhar a configuração." | "Tentou subir o serviço na abertura do app e registrou a falha." | "Não mandou as páginas para outro lugar — a fila de leitura ficou como estava." |
| `fantasma_autostart_failed` | "O robô estava configurado para subir sozinho e não veio — container ou login no Everest falharam. As mãos ficam paradas até alguém olhar." | "Tentou a subida configurada e registrou a falha." | "Não mexeu nas mãos paradas: elas continuam esperando alguém olhar." |
| `nfepack_autostart_failed` | "O monitor estava configurado para subir sozinho e não veio. As notas do portal param de descer até alguém olhar a configuração." | "Tentou subir o monitor na abertura do app e registrou a falha." | "Não tocou no que já foi baixado — as notas em disco ficaram como estavam." |
| `nfepack_errors` | "O log do NFePackMonitor cresceu em linhas de erro desde o último aviso — o número abaixo é o quanto cresceu. As linhas estão em Logs · NFe Pack." | "Contou as linhas de erro novas no log e avisou uma vez, com o total." | "Não interpretou as linhas — elas estão cruas em Logs · NFe Pack." |
| `dock_pieces_extracted` | "A leitura terminou e as peças saíram da fila de decisão — estão em Já extraídos, a caminho da Revisão." | "Enviou as peças ao MedLens e as moveu para Já extraídos." | "Não aprovou nem enfileirou nada — conferir continua sendo seu." |
| `dock_extraction_failed` | "O MedLens não conseguiu ler. A peça continua na doca, marcada, e diz o motivo — nada foi extraído dela." | "Guardou a recusa do MedLens, com as palavras dele, na peça." | "Não inventou uma leitura nem tirou a peça da doca." |
| `opmenexo_invoices_sent` | "O envio respondeu: a linha da lista diz o estado, e a ordem guarda o relatório arquivo por arquivo." | "Subiu os arquivos pela ordem e guardou o relatório arquivo por arquivo." | "Não deu baixa em nada além do que o portal confirmou." |
| `opmenexo_invoices_failed` | "Um envio ao portal terminou sem resposta ou recusado. Não reenvie antes de olhar a ordem: se o arquivo já subiu, a segunda tentativa é segunda via." | "Guardou o desfecho do envio, com a resposta do portal quando houve." | "Não reenviou sozinho — segunda tentativa sem olhar a ordem pode ser segunda via." |
| `opmenexo_quotation_answered` | "A resposta saiu: a linha da lista diz o estado, e a ordem guarda o relatório linha a linha." | "Enviou a resposta montada na folha e guardou o relatório linha a linha." | "Não mudou preço nenhum — o que saiu é o que você conferiu." |
| `opmenexo_quotation_failed` | "Uma resposta ao portal terminou sem confirmação ou recusada. Não reenvie antes de olhar a ordem no portal: preço é proposta comercial." | "Guardou o desfecho da tentativa, com a resposta do portal quando houve." | "Não tentou de novo sozinho — preço é proposta comercial." |
| `opmenexo_news` | "A releitura automática achou mais trabalho que a leitura anterior — as contagens estão no aviso; o painel tem as linhas." | "Releu o portal no relógio que você marcou e contou o que chegou." | "Não respondeu nem baixou nada sozinho — o trabalho continua lá." |

**Tone words.** "correu bem" (ok; no kind uses it today), "pede atenção" (warn), "parou" (danger), "chegou coisa" (accent).

**Sources and their dots.** Sistema (neutral grey), MedLens (accent), Fantasma (red), Esteira (amber), Spook (a lighter accent), NFe Pack (a darker accent). "MailBridgr" (green) is also a defined source, but no kind speaks for it, so its chip is not drawn: **six chips today**.

### What each number counts

| Where | Counts |
| --- | --- |
| Bell badge and panel summary | unread notices of known kinds, not archived, whose kind is **not** silenced |
| Screen summary | "<N> aviso não lido" / "<N> avisos não lidos" (or "nada não lido"), counting unread notices **including** silenced kinds; then " · <M> nos últimos 30 dias", M being every notice on the screen (known kinds, not archived); then " · nada foi decidido sozinho". The search, tabs and chips do not change it |
| "Não lidos (N)" | the same unread count as the screen summary |
| "Pedem você (N)" | every notice whose kind asks for a human, **read or not** |
| Day header "<N> avisos" | rows in that section after filtering; a folded row counts once |
| Row "<N>×", "Vezes", timeline | the notices folded into that row |

- **Elapsed time in the panel**: "agora" (under a minute), "<N> min" (under an hour), "<N> h" (under 24 h), "ontem" (24 to 48 h ago), then "dd/mm/aaaa". It is recomputed every 30 s while the panel is open.
- **Times on the screen**: "hh:mm", local. The day labels are "Hoje · dd/mm/aaaa", "Ontem · dd/mm/aaaa" (the calendar day), or the capitalised weekday followed by "· dd/mm/aaaa".
- **Grouping** folds the notices that share the kind, the stored detail and the local day. With the switch off, every notice is its own row.

## States

**The bell**

| State | Trigger | Drawn |
| --- | --- | --- |
| Nothing pending | no unread, non-silenced notice | no badge |
| Pending | at least one | the badge with the number |
| Only silenced kinds unread | — | no badge (silenced kinds do not count) |
| Locked | the window lock | bell disabled, panel closed |
| First assembly | the `/welcome` screen | no bell |
| Stale badge | a notice posted in the background (a boot failure, the monitor's error count, an extraction's outcome) | the badge shows it only at the next re-read: at launch, on unlock, when the panel opens, when the Avisos screen opens, or after this side posts a notice |

**The panel**

| State | Trigger | Drawn |
| --- | --- | --- |
| Pending | — | the list |
| Nothing pending | — | "Nada pendente por aqui. O que já foi lido fica em Avisos."; the summary "nada pendente por aqui"; the mark-all button disabled |
| Opening | click | re-reads the notices from disk; elapsed times re-tick every 30 s |
| An unknown kind | a stored kind this version does not know (an older or newer file) | never drawn, never counted |

**The screen**

| State | Trigger | Drawn |
| --- | --- | --- |
| Empty, "Todos" | no notices | bell-slash icon, "Nenhum aviso por aqui", "Quando algo acontecer sozinho — ou precisar de você — aparece aqui." |
| Empty, search | a term that matches nothing | magnifier icon, "Nada com “<term>”", "A busca olha o título, o texto e a origem do aviso." |
| Empty, chips | chosen sources that match nothing | magnifier icon, "Nenhum aviso dessas origens", with the same note |
| Empty, "Não lidos" | nothing unread | check-circle icon, "Você leu tudo", "Os avisos lidos continuam na aba Todos por 30 dias." |
| Empty, "Pedem você" | no asking kind | check-circle icon, "Nada esperando por você", "O que sobrou é registro: coisas que aconteceram e não pedem decisão." |
| No rows at all | any empty state | the detail panel is absent |
| Selected row | click | accent border and tint; the detail shows it |
| None selected yet | on entry | the detail shows the first row, without marking it read |
| Silenced kind | the kind was silenced | its rows are dimmed and carry "silenciado" instead of "<N>×"; they still count in the screen's numbers |
| Folded row | more than one identical notice in a day | the "<N>×" badge, "aconteceu <N> vezes neste dia", and a timeline in the detail |
| Grouping off | switch off | one row per notice |
| Narrow window | below 1180 px | no detail panel and no selected look; clicking a row follows its notice at once (and marks it read) |
| A write that did not persist | reading, archiving or silencing failed on disk | nothing is shown: these are optimistic, and the notice returns to its previous state the next time the list is re-read from disk |

When the screen is empty, the search term decides which empty face shows first, then the chips, then the tab.

## Actions

**Bell and panel**

| Gesture | Effect | Confirmation · reversible |
| --- | --- | --- |
| Click the bell | opens or closes the panel | — |
| `Esc`, or a click outside | closes the panel | — |
| Click an item | closes the panel, marks that notice read and opens its target | — · reversible on the Avisos screen |
| "Marcar todos como lidos" | marks **every** notice read, silenced kinds included | none · reversible one row at a time on the Avisos screen |
| "Ver todos os avisos" | closes the panel and opens `/notices` | — |

**The Avisos screen**

| Gesture | Where | Effect | Confirmation · reversible |
| --- | --- | --- | --- |
| Opening the screen | — | re-reads the notices from disk | — |
| Click a row (wide window) | list | selects it and marks every notice folded in it read ("looking is reading") | — · the read toggle walks it back |
| `Enter` on a focused row | list | same as a click | — |
| Click a row (narrow window) | list | marks it read and opens its target | — |
| Read toggle | row or detail | if every folded notice is read, all become unread; otherwise all become read | — · yes |
| "Silenciar este tipo" / "Voltar a avisar" | row or detail | per **kind**, not per row: every notice of that kind dims and stops counting on the bell | none · yes, from any row of that kind |
| "Arquivar" | row or detail | archives every notice folded in the row and marks them read. The row leaves every view, and no screen offers a way back; the file keeps the notices until the cap or the 30 days remove them | ⚠️ **none** · not from the screen |
| "Abrir <target>" | detail foot | marks the row read and opens the kind's screen | — |
| "aconteceu <N> vezes neste dia" / "esconder os horários" | folded row | shows or hides the time chips | — |
| Tabs, chips, search | toolbar | filter the list; `Ctrl ⇧ K` focuses the search | — |
| "Agrupar repetições" | toolbar | folds or unfolds repeats | — |
| "Marcar todos como lidos" | header | same as the panel's | none |
| "Ver o histórico completo nos Logs" | under the list | opens [Logs](11-logs.md) on the app's journal | — |

The tab, search term, chips, grouping, selection and expanded rows last only while the screen is open; coming back resets them.

## Business rules

1. **Two channels.** A toast is feedback on an action and is never stored. A notice is a pending subject: something that asks for a human decision or happened without a hand (decision 010).
2. **A notice leads somewhere and resolves nothing.** This covenant is the screen's spine: "Decidido pelo app: nada" on every detail, and "nada foi decidido sozinho" in both summaries (038).
3. **The panel is "what is asking for me"**: pending only. The history is the screen's (010).
4. **Stored as a kind, worded when drawn.** A notice is stored as a kind (an English `snake_case` identifier) with an optional short detail. The title, text and narration come from the language layer at draw time, so a language switch rewords every stored notice (010).
5. **Unknown kinds are invisible.** A kind this version does not know draws nothing and rings nothing, so an old or new file never breaks the bell (010).
6. **No values in a notice.** The journal's rule applies to notices too. The details are an order number (OPMENEXO outcomes: no file name, no patient, no product, no price), counts (extractions, portal news) or a growth delta (the monitor's errors) (`lib.rs`, `opmenexo.svelte.ts`).
7. **Every posted notice is also a journal line** (`notice_posted` plus the kind), because the screen promises the full story in [Logs](11-logs.md) (010).
8. **Silencing quiets the bell, never the record.** A silenced kind is still posted and listed, dimmed and labelled "silenciado", and stops counting on the bell; the screen's own numbers keep counting it (038).
9. **Read is a flag the operator owns**: a read notice can walk back to unread (038).
10. **Archiving is for good.** An archived notice leaves every view, and archiving marks it read too, so a put-away notice cannot keep ringing. The design draws no way back, and none is built (038).
11. **Retention: 30 days, and at most 100 notices.** Older notices are dropped whenever the file is read, which makes the screen's "avisos ficam 30 dias" true; beyond 100, the oldest fall off (010, 038).
12. **A folded row's identity is kind + detail + local day.** Two different error deltas are two different facts and stay apart; actions on a folded row land on all its notices (038).
13. **"Pedem você" and the source belong to the kind**, not to each notice, so old files gain those answers for free (038).
14. **Chips exist only for sources that some kind speaks for.** A chip no notice could ever wear would claim a producer that does not exist (038).
15. **No polling.** The list is re-read from disk at launch, on every unlock, when the panel opens, when the screen opens, and after this side posts a notice. Producers are rare (010).
16. **Doors.** No bell on the first-assembly screen; the bell is disabled under the lock; toasts are never drawn over the lock (010).

## Must survive the redesign

- Two channels: notices only for pending subjects and hand-free events; toasts are never stored or listed here.
- Every notice leads to where its subject lives and resolves nothing. The "o que o app fez / o que não fez" narration and "Decidido pelo app: nada" stay on every detail.
- The panel lists pending notices only, and the badge counts unread, non-silenced ones.
- Silencing is not deleting: silenced rows stay listed, dimmed and labelled.
- Archiving also reads the notice.
- "avisos ficam 30 dias" is said on the screen.
- Unknown kinds stay invisible.
- The bell is absent on the first-assembly screen and disabled under the lock.
- No patient data in any notice: order numbers, counts and deltas only.
- Tone is never colour alone: each tone has a word ("parou", "pede atenção", "correu bem", "chegou coisa") and every kind has an icon.
- `Ctrl ⇧ K` focuses the screen's search, and its hint is drawn on it.

## Free to redesign

The panel's form (dropdown or side sheet); the look of rows and cards; chips versus a select; the timeline's form; the detail's layout and where it sits; the tone palette (within the fixed state semantics); the badge's shape.

## Sample content for mockups

Today is 10/09/2026 (a Thursday), 14:40.

**Panel** (3 pending), summary "3 avisos não lidos · nada foi decidido sozinho":

- "O monitor registrou erros novos" · "6 min" · its text · "+3" · "→ NFEPACKMONITOR"
- "Uma extração falhou" · "1 h" · its text · "→ TRIAGEM"
- "Novidades no portal" · "ontem" · its text · "+2 cotações · +3 mensagens" · "→ OPMENEXO"

**Screen.** Summary "3 avisos não lidos · 9 nos últimos 30 dias · nada foi decidido sozinho". Tabs: "Todos", "Não lidos (3)", "Pedem você (4)". Six chips.

| Section | Time | Row | Tone · source | State |
| --- | --- | --- | --- | --- |
| HOJE · 10/09/2026 — 3 avisos | 14:34 | O monitor registrou erros novos, detail "+3" | parou · NFe Pack | unread |
| | 13:39 | Uma extração falhou | parou · Esteira | unread |
| | 13:22 | A janela se trancou sozinha, "3×"; times 13:22 · 11:05 · 09:47 | chegou coisa · Sistema | read |
| ONTEM · 09/09/2026 — 2 avisos | 17:45 | Novidades no portal, "+2 cotações · +3 mensagens" | chegou coisa · Spook | unread |
| | 08:02 | O NFePackMonitor não subiu com o app | parou · NFe Pack | read |
| TERÇA-FEIRA · 08/09/2026 — 2 avisos | 16:10 | Notas enviadas ao portal | chegou coisa · Spook | read |
| | 10:31 | Extração concluída | chegou coisa · Esteira | read, "silenciado" |

**Detail of the first row:**

- badges "PAROU" and "NÃO LIDO", the text, and "+3";
- De onde veio: NFe Pack · Quando: "Hoje · 10/09/2026, 14:34" · Vezes: "uma vez" · Decidido pelo app: "nada";
- "o que o app fez": "Contou as linhas de erro novas no log e avisou uma vez, com o total." / "Não interpretou as linhas — elas estão cruas em Logs · NFe Pack.";
- the button "Abrir NFePackMonitor".

## Design opportunities

1. **Keep source colours out of the state roles.** The Fantasma's dot is red and the Esteira's is amber, while the brief fixes amber as "a human must decide here" and red as "failed". A source dot in those colours reads as a state it does not carry.
2. **Tell the OPMENEXO outcome rows apart.** Their order number is stored as the detail but not drawn, so two sends to two orders give two identical-looking rows that do not fold, because their details differ. An order number is not patient data; draw it (e.g. "ordem 7100231").
3. **A place to see and undo silenced kinds.** Today a kind is un-silenced only from one of its rows. Once every notice of a silenced kind has been archived or aged out, nothing on screen can reverse the silence. (The design's "Escolher quais avisos aparecem" button was cut because no preferences section exists.)
4. **Make "Pedem você" count what is still pending**, for example "2 de 4" or unread asks only. Today a handled, read failure still counts.
5. **An undo for Arquivar** (a toast with "Desfazer"). Today it is final and asks nothing.
6. **A narrow-window detail.** Below 1180 px the narration disappears and a click navigates at once; a slide-over would keep "o que o app fez / o que não fez" readable at 1024 px.
7. **Quick actions in the panel** (mark one read without leaving the current screen).
8. **A badge cap** ("9+").
9. **One meaning of "ontem."** The panel's "ontem" means 24 to 48 hours ago; the screen's "Ontem" is the calendar day.
10. **A bell that updates live.** Notices posted in the background reach the badge only at the next re-read (decision 010 already names an event channel as the expected evolution).

## Open questions

- Should Arquivar be undoable, and should it ask first?
- Should silenced kinds count in the screen's numbers and in "Pedem você"?
- Where should the list of silenced kinds live, and how is a kind un-silenced when no notice of it remains?
- Should "Pedem você" describe a state (still pending) rather than the kind of notice?
- The text of "O monitor registrou erros novos" points to "Logs · NFe Pack", but the row opens the NFePackMonitor console. Which door is right?
