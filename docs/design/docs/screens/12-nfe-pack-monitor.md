# NFePackMonitor — `/nfe-pack-monitor`

> Where: sidebar group **Ideias futuras** › **NFePackMonitor** (not a link while no monitor folder is configured; dimmed while the monitor is stopped); also the dashboard's Serviços card ("Abrir") and the notice "O monitor registrou erros novos" · Status: built · Code: `src/routes/nfe-pack-monitor/+page.svelte`, `src/lib/nfepack.svelte.ts`, `src/lib/components/service-fields/NfepackMetrics.svelte`, `src/lib/components/service-fields/ServiceToggle.svelte`, `src/lib/components/NfepackNoteDialog.svelte`, `src/lib/components/GridSurface.svelte`, `src/lib/nfe-nature.ts`, `src/lib/xml.ts`, `src/lib/ipc.ts` (`NfepackStatus`, `NfepackPanel`, `NfepackNotesListing`, `NfepackNoteFile`), `src-tauri/src/nfepack_service.rs`, `src-tauri/src/lib.rs` (`nfepack_status`, `nfepack_notes`, `nfepack_panel`, `nfepack_note_file`, `start_nfepack`, `stop_nfepack`, `read_nfepack_log`, `nfepack_error_watch`), `src/lib/i18n/locales/pt-BR.ts` (`nfepack`)

## The question

"As notas estão descendo do portal sozinhas?" is the screen's subtitle, verbatim. Behind it: is the monitor downloading, when was its last good cycle, what came down, and what did it say?

## Where it sits in the work

The **NFePackMonitor** is a separate program, a scraper. It downloads NF-e XMLs and their DANFE PDFs from the ACOM portal into a notes folder, and records every download in its own offline-first queue (a SQLite file that belongs to it). Bridgr Pro raises it as its fifth service, watches it, lists what it downloaded and opens notes **read-only**. This screen is the monitor's console. Configuration (project folder, notes folder, cycle interval, portal login) lives in [Configurações › NFe Pack](10-settings.md), which also carries the same service card and metrics card. The downloaded notes are what [OPMENEXO](13-opmenexo.md) attaches by number and suggests by patient; that work does not happen on this screen.

| From | Lands |
| --- | --- |
| Sidebar › Ideias futuras › NFePackMonitor | here. With no folder configured, the item is not a link and its hover says "Precisa do NFePackMonitor configurado — Configurações · NFe Pack". Configured but not running, the icon is dimmed, the item is still a link, and its hover says "Monitor parado — Configurações · NFe Pack" |
| [Dashboard](01-dashboard.md), Serviços card, monitor block up → "Abrir" | here (block down → "Configurar" → Configurações › NFe Pack) |
| The notice "O monitor registrou erros novos" ([Avisos](17-notices.md)) | here |
| The status bar's NFe block ([04-shell](../04-shell.md)) | Configurações › NFe Pack, not here |

## Anatomy

Wide window (at least 1180 px), top to bottom. The header, cards and band keep their natural height; the split below takes the remaining height, and each of its panes scrolls on its own.

1. **Header**: the title "NFePackMonitor" and the subtitle question.
2. **Service card** **[functional]**: the state and its switch live together on one card.
   - A receipt icon, green while the monitor runs and grey otherwise.
   - "Serviço · <state>" (the state words are in Data shown).
   - A quiet line: "As NF-e do portal ACOM — sobe com o app e fica no ar; só Desligar derruba".
   - A second quiet line: "Último ciclo: <heartbeat line>".
   - At the right, the on/off switch shared by every service card (on = the service is up). A spinner sits inside the switch while a rise or a stop runs; the switch is disabled, with a hover reason, when it cannot move.
3. **Gesture failure strip** (danger), only after a refusal: the backend's message, verbatim.
4. **Métricas card** (the same card as in Configurações › NFe Pack): the title "Métricas", "exportadas em dd/mm/aaaa · hh:mm" at the right, then six tiles (a monospace value over a label). When nothing was ever exported, one sentence replaces the tiles.
5. **Band failure strip** (danger), only when the band's read failed: "A leitura do painel falhou — <message>".
6. **The band**: two cards side by side, equal width.
   - **"Fila de sincronização"**. Header: the title, "<total> eventos", and a "Reler" button that reloads **both** cards. Body: four or five count tiles (a value over an uppercase label); then, only when failed items exist, "Falhas de sincronização" with up to five rows; then a quiet explanatory note.
   - **"Ciclos"**. Header: the title and "os 5 últimos · o monitor guarda só 10". Body: a totals line, then the last five cycles, newest first, one row each (status icon, stamp, facts, and the monitor's own error sentence under a row that recorded one). The list scrolls past 190 px.
7. **The split: 2/3 notes + 1/3 log** **[functional]**. Both panes are visible side by side and scroll separately: a single scroll would hide the log exactly while the operator reads the listing beside it (the operator's drawing, 26/08/2026).
   - **"Notas baixadas"** (2/3): the pane title and "Reler"; under them the house grid ([05-components](../05-components.md) › GridSurface): the filter field "Filtrar notas…" with `Ctrl ⇧ K`, the count line on its own row under the tools, the columns menu, the row-density toggle and the pager. Five columns, plus a last column with two icon buttons per row (XML and DANFE). A danger line appears under the grid when an opening was refused.
   - **"O que o monitor disse"** (1/3): the pane title, "Reler", and "Seguindo"/"Seguir"; then a monospace panel like the one on [Logs](11-logs.md): the monitor's full log, newest line on top, chronological numbers in a 40 px gutter. No filter, no copy.
8. **The note dialog**: an overlay opened from a row (see States and Actions).

Narrow window (below 1180 px, where the sidebar also folds to a rail): the band stacks into one column, and a segmented switcher, "Notas baixadas" and "O que o monitor disse", appears above the split. The split then shows only the chosen pane (notes by default).

## Data shown

**Service state** (the words after "Serviço · "):

| Condition | Word (verbatim) | Icon | Switch |
| --- | --- | --- | --- |
| no answer yet | "…" | grey | off, disabled, no reason given |
| no project folder configured | "sem pasta configurada" | grey | off, disabled, no reason given |
| running, raised or adopted by this app | "baixando" | green | on, movable |
| running, started by another program from the same folder | "no ar — subiu fora deste app" | green | on and **disabled**; hover: "de fora do app — quem subiu foi outro programa; usar, sim; derrubar daqui, não" |
| configured, not running | "parado" | grey | off, movable |

"Running" is read from the operating system's process list, not from a health call: the monitor has no port.

**Último ciclo** comes from the heartbeat file, which the monitor rewrites **after every successful cycle**: "dd/mm/aaaa · hh:mm · N notas no ciclo · M ciclos no total". Each count joins only when the file carries it, written as the file has it (no thousands separator); a stamp that does not parse is shown raw. Without a heartbeat: "nenhum ciclo registrado ainda".

**Métricas** are the counters the monitor exports to a text file every 10 successful cycles. The stamp is that file's modification time ("exportadas em …"), because the numbers can be minutes old while the monitor keeps working.

| Tile label (verbatim) | Counter | Format |
| --- | --- | --- |
| ciclos | total cycles | integer with thousands separator |
| bem-sucedidos | successful cycles | same |
| falhas | failed cycles | same; ⚠️ the value turns amber whenever it is above 0 (a lifetime count) |
| notas baixadas | notes downloaded | same |
| duração do último ciclo | the last cycle's duration | "12,4 s" |
| notas no último ciclo | the last cycle's note count | integer with thousands separator |

A counter the file does not carry shows "—", never zero. With no file (or no known counter in it): "Nenhuma exportação ainda — o monitor grava as métricas a cada 10 ciclos bons."

**Fila de sincronização** counts every event in the monitor's queue by the queue's own status:

| Tile (label as written; drawn uppercase) | Counts | Tone |
| --- | --- | --- |
| Pendentes | `pending` | neutral: thousands here is the designed state while the monitor's sync is off |
| Processando | `processing` | neutral |
| Enviadas | `sent` | neutral |
| Com falha | `failed` | ⚠️ amber border and value when above 0: the band's one alarm |
| Outros estados | any other status | neutral; the tile exists only when above 0 |

"<total> eventos" is the sum of all the tiles, so the total always agrees with what is drawn. Failed items (at most the five most recent) read "#<id> · <queue name> · <N> tentativas · dd/mm/aaaa · hh:mm" in monospace (each piece only when present; the stamp is when the event was created), with the recorded error beneath, verbatim, in danger ink. The note under the tiles, verbatim: "A fila offline-first do monitor: cada download vira um evento, e pendente aguarda a sincronização com o Redis — que só corre quando está habilitada no monitor."

**Ciclos.** The totals line comes from the monitor's running totals, which are fresher than the metrics export: "<N> ciclos desde dd/mm/aaaa · <F> com falha". The "desde …" part appears only when a start date exists; "· F com falha" only when F is above 0. Each cycle row: a check-circle (green) or a warning-circle (red), "dd/mm/aaaa · hh:mm", then "12,4 s · 3 notas", plus " · falhou" on a failed cycle, and the monitor's own error sentence on its own line when it recorded one.

**Notas baixadas**: one row per download the monitor recorded, newest first.

| Column (verbatim) | Meaning | Presentation | Example |
| --- | --- | --- | --- |
| Nota | the note's number as the monitor recorded it | text | `100245` |
| Cliente | the client the monitor recorded | text | `HOSPITAL VALE VERDE` |
| Operação | the note's _natureza da operação_ reduced to one word | "Devolução", "Retorno", "Bonificação", "Transferência", "Compra", "Venda" or "Remessa", tested in that order and ignoring accents (so "DEVOLUCAO DE VENDA" is Devolução). A nature that matches none is shown **as the note says it**; no nature leaves the cell empty | `Venda` |
| Valor | the note's total (`vNF`) | Brazilian format with two decimals in **both** app languages ("12.480,00"), right-aligned and sorted as a number; when absent the cell is empty, never zero | `12.480,00` |
| Emitida em | the note's own emission date (not when it was downloaded) | `dd/mm/aaaa`; raw text when it does not parse; empty when absent | `02/09/2026` |

The natures are domain words and are never translated. The count line under the grid's tools, while the history streams in: "<loaded> de <total> baixadas — as mais recentes primeiro" (for example "500 de 1.208 baixadas — as mais recentes primeiro"). Once the history is whole: "<total> baixadas — as mais recentes primeiro".

**O que o monitor disse**: the monitor's log, the last 1000 lines at every level, verbatim, newest on top, numbered with 1 = the oldest line in the window.

## States

| Region | State | Trigger | Drawn |
| --- | --- | --- | --- |
| Card | not answered yet | the status store has never answered (rare: the status bar keeps it warm app-wide) | "Serviço · …", "Último ciclo: nenhum ciclo registrado ainda", switch off and disabled; the metrics card shows its never-exported sentence |
| Card | unconfigured | no project folder (reachable by URL or an old notice; the sidebar item is not a link) | "sem pasta configurada"; every other region shows its absent-queue sentence, and the log shows its empty sentence |
| Card | stopped | configured, and nothing runs from the folder | "parado", switch off and movable |
| Card | rising or stopping | a switch gesture in flight | a spinner inside the switch, which is locked; the state word changes when the status is re-read after the gesture |
| Card | running, ours | — | "baixando", green icon, switch on and movable |
| Card | running, foreign | a scraper that another program started from the same folder | "no ar — subiu fora deste app", switch on and locked, with the hover reason |
| Card | refused | Ligar or Desligar failed, or a status read failed | a danger strip under the card with the message verbatim (English, prefixed "NFePackMonitor: "), e.g. "NFePackMonitor: uv was not found (PATH, ~/.local/bin, ~/.cargo/bin, UV_BIN)"; it stays until the next gesture |
| Card | process up, portal failing | the monitor's known trap: a failed portal login leaves the process up and the cycles empty | nothing special, by design. Per decision 033 (the monitor's own behaviour, not verifiable in this repository), the heartbeat stops moving and the failure appears in the log; the screen interprets neither |
| Metrics | never exported | no metrics file | "Nenhuma exportação ainda — o monitor grava as métricas a cada 10 ciclos bons." |
| Metrics | a counter missing | the file lacks that counter | the tile shows "—" |
| Band | not asked yet | before the first answer | "…" in both cards |
| Band | no queue | no folder, or no queue file | both cards: "O monitor ainda não registrou downloads desta pasta — a fila nasce no primeiro ciclo com nota." |
| Band | reloading | Reler, or a new cycle | the Reler icon spins and the button is disabled; the previous content stays |
| Band | failed | the queue refused the read (e.g. it lost to the monitor writing) | a strip above the band, "A leitura do painel falhou — " + the message; the last good content stays |
| Band | failures present | `failed` above 0 | the amber "Com falha" tile and the "Falhas de sincronização" list |
| Band | an unknown status | a status outside the four | an extra "Outros estados" tile |
| Band | no cycles | an older monitor without the metrics tables, or none recorded | Ciclos: "O monitor ainda não registrou nenhum ciclo aqui." (no totals line when there are no totals) |
| Notes | not asked yet | before the first answer | "…" |
| Notes | no queue | the queue file does not exist | "O monitor ainda não registrou downloads desta pasta — a fila nasce no primeiro ciclo com nota." |
| Notes | empty queue | the file exists with zero downloads | "A fila existe e está vazia — nenhuma nota baixada ainda." |
| Notes | streaming | the history is larger than one batch | the first 500 rows paint at once and the rest stream in 500 at a time; the count line reads "N de M baixadas…"; Reler spins and stays disabled for the whole walk |
| Notes | reload on a new cycle | the heartbeat's cycle count moved | the listing restarts from the top: the newest 500 replace the loaded set and the rest streams in again (the count line returns to "500 de M…") |
| Notes | read failed | the queue refused (it lost to the monitor writing) | a danger line, "Não deu para ler a fila do monitor — " + the message. ⚠️ The last listing stays: a lost read never looks like an emptied history |
| Notes | the filter matches nothing | — | inside the grid: "Nenhuma nota passa no filtro." |
| Notes | opening | a row's XML or DANFE button pressed | that button's icon becomes a spinner; every row button is disabled until it answers |
| Notes | opening refused | the document cannot be opened | a danger line under the grid with the message verbatim (list below); it stays until the next opening |
| Log | empty | no folder or no file | "Nada ainda — o monitor nunca rodou a partir da pasta configurada." |
| Log | following / not following | the toggle | re-read every 2 s with the pin to the top / "Reler" only |
| Dialog | XML | an XML opened | a read-only code editor: line numbers, wrapped lines, the XML indented for reading, with tags, attributes, values and comments coloured from theme tokens |
| Dialog | XML search | `Ctrl+F` | a search bar across the top of the editor: the field "Buscar", buttons "próxima", "anterior" and "todas", checkboxes "maiúsculas/minúsculas", "palavra inteira" and "regexp", and a close button ("fechar"). Matches are lit in amber, stronger on the current one. No replace |
| Dialog | PDF | a DANFE opened | the platform's built-in PDF viewer, with its own zoom and pager, filling the dialog (the same viewer [Revisão](03-review.md) uses by default) |
| Dialog | full window | expand toggled | the dialog fills the window, keeping a 14 px margin |
| Screen | narrow | below 1180 px | the band in one column; the switcher "Notas baixadas" / "O que o monitor disse"; one pane at a time |

Opening refusals, verbatim, each prefixed "NFePackMonitor: ":

- "the DANFE is no longer beside the XML — a cancelled note is cleaned up, and the queue keeps no copy of a PDF"
- "the file is no longer where the monitor put it — a cancelled note is cleaned up, and the queue kept no copy of this one"
- "the queue keeps no path for this note" · "the queue keeps neither the XML nor a path to it"
- "the monitor's queue does not answer that id: …" · "the monitor's queue refused to open read-only: …"
- "no NFePackMonitor project folder configured"

## Actions

| Gesture | Where | Precondition / disabled reason | Effect | Confirmation · reversible | Feedback |
| --- | --- | --- | --- | --- | --- |
| **Ligar** (switch on) | service card | a folder is configured and nothing runs from it; disabled otherwise (no reason drawn when unconfigured) | raises the monitor from the project folder, passing at launch what was typed in Configurações. Nothing is started beside a scraper that another program already runs from that folder | none · reversible with Desligar | the spinner on the switch appears before the command is sent, then the new state; a refusal gives the danger strip |
| **Desligar** (switch off) | service card | running and **ours**; for a foreign monitor the switch is locked with the reason "de fora do app — quem subiu foi outro programa; usar, sim; derrubar daqui, não" | stops the process group this app raised | ⚠️ **no confirmation** · reversible with Ligar | spinner on the switch, then "parado" |
| Reler (band) | Fila de sincronização header | disabled while reloading | re-reads the queue counts, failed items, totals and cycles | — | spinning icon |
| Reler (notes) | Notas baixadas header | disabled while the listing loads or streams | reloads the listing from the top | — | spinning icon; the count line restarts |
| Filter, sort, columns, density, pages | the grid's tools, headers and pager | — | apply to the rows loaded so far (a set that converges to the whole history) | — | [05-components](../05-components.md) › GridSurface; `Ctrl ⇧ K` focuses the filter |
| **Abrir o XML aqui** (code icon) | the row's last column | no other opening in flight | opens the note's XML in the dialog, from the copy kept in the queue first (so a cancelled note still opens), otherwise from the file | — | spinner on that button; a refusal gives the danger line |
| **Abrir a DANFE aqui** (PDF icon) | the row's last column | no other opening in flight | opens the PDF the monitor wrote beside the XML | — | spinner on that button; a refusal gives the danger line (a PDF has no copy in the queue) |
| Reler / Seguir (log) | log pane header | — | as on [Logs](11-logs.md): one re-read / toggles the 2 s re-read and the pin to the top | — | label and tint |
| Switch pane | the narrow switcher | below 1180 px only | shows the notes or the log | — | pressed segment |
| Search in the XML | dialog | an XML is open | `Ctrl+F` anywhere in the dialog opens the search bar | — | matches lit |
| Expand / restore | dialog header (arrows icon) | — | toggles the full-window size; the titles are "Ocupar a janela inteira" / "Voltar ao tamanho do desenho" | — | — |
| Close | dialog header X ("Fechar"), a click outside, or `Esc` | — | closes the dialog; the decoded document is discarded | — | `Esc` closes an open search bar first, then the dialog; the header hints "Esc fecha" |

## Business rules

1. **Configuration is elsewhere.** This screen watches, lists and opens; the folders, the interval and the portal login are in Configurações › NFe Pack (decision 033).
2. **The monitor has no port**, so "running" is read from the process list. A scraper that somebody else started from the configured folder is **foreign: usable, never stopped from here**, and nothing is started beside it. Two scrapers on one portal and one folder would download everything twice and race for the same files (033).
3. **It rises with the app and does NOT fall with it** (the operator's choice, 26/08/2026). The monitor keeps downloading after the app closes; Desligar is the only gesture that takes it down, and the card's quiet line says so. The next launch adopts whatever the previous one left running. Whether it rises at launch at all is a switch in Configurações › Sobre ([Configurações](10-settings.md)).
4. **A living process is not a working portal.** A failed portal login leaves the process up and the cycles empty. The screen shows the heartbeat and the log and interprets neither: a health state is never derived from the log's sentences (033; [02-invariants](../02-invariants.md)).
5. **Sign of life first.** The switch shows its spinner before the command is sent (a rise can take a while as the monitor's environment resolves); each opening shows a spinner on its own button; every Reler spins.
6. **The stamp is part of the answer.** The metrics say when they were exported, and a counter the file lacks is a dash, never zero (033).
7. **Only `failed > 0` warns on the queue.** Thousands of "Pendentes" while the monitor's Redis sync is disabled is the queue's designed offline-first state, and painting it amber would be a permanent lie. The note describes the queue's design and asserts nothing about the monitor's live configuration, which this app cannot read (036).
8. **Never the payload.** Failed items show bounded fields only (id, queue, attempts, stamp, recorded error), because a queue event's payload carries the note's whole XML and a patient's name (036). The listing never reads the XML either: it cuts the emission date and the total out of it, and nothing else (034).
9. **Five of ten, and both numbers said.** The card shows the last 5 cycles, and the monitor itself keeps only 10. The header says both, or five rows would read as the whole story (036).
10. **The listing is the monitor's own download history, read-only**, never a scan of the network share. It arrives **whole, streamed 500 at a time**, newest first, and the count line says how far it got: no silent caps. Filter and sort work on what is loaded, which converges to everything (034).
11. **Absence, emptiness and failure are three different sentences**: no queue file ("O monitor ainda não registrou…"), an empty queue ("A fila existe e está vazia…"), and a read that lost to the writer (loud, keeping the last listing) (034).
12. **The cycle is the clock.** The listing and the band reload when the heartbeat's cycle count moves (the queue only grows on a cycle) and on the manual Reler; they have no polling of their own. The status itself is re-read every 5 s while this screen is open, and the log re-reads every 2 s while following (033, 034, 036).
13. **An id, never a path.** A row's buttons send the queue row's id, and only a file that the monitor's own queue names can be opened (034).
14. **The XML is indented for the eyes only.** Nothing formatted is written back or sent anywhere, so reshaping whitespace cannot touch what was signed. The opened XML names the patient (in the NF-e's additional information), and the decoded document is discarded when the dialog closes (034; LGPD).
15. **A PDF that is gone is a loud refusal, never an empty viewer.** The monitor cleans a cancelled note from disk; its XML still opens from the queue's copy, its DANFE does not (034).
16. **The error bell.** The status check counts the log's `[ERROR]`/`[CRITICAL]` lines and remembers the count. Growth rings **one** notice carrying the difference ("+3"), never one notice per line; the daily log rotation shrinks the count and resets the memory without ringing. The notice leads here ([Avisos](17-notices.md)). The bell says that the log grew, nothing about what the lines mean (034).
17. **Natures are domain data**: Portuguese words that are never translated. An unknown nature crosses as the note says it; inventing a bucket for it would assert what nobody verified.
18. **Money is in the Brazilian format in both languages**, because an NF-e total is in reais and the grid sorts "1.234,56" as a number.

## Must survive the redesign

- The service's state and its switch on one card. Foreign = on and locked, **with the reason**; no Desligar for what this app did not raise.
- The card's statement that the monitor stays up after the app closes and that only Desligar stops it.
- The heartbeat line and the raw log side by side with the listing; no health light, badge or verdict derived from log text.
- The metrics' export stamp, and "—" for a missing counter.
- On the queue, amber only for `failed > 0`. "Pendentes" never warns, and the design note stays.
- Failed items and cycles show their recorded errors verbatim, and never any payload or patient data.
- "os 5 últimos · o monitor guarda só 10": both numbers.
- The count line while streaming and once whole; three distinct sentences for no queue, empty queue and failed read; the last listing kept on failure.
- Notes open inside the app, read-only, from their row. The XML's indentation is display-only. A gone PDF is a refusal shown, never an empty viewer.
- 2/3 notes + 1/3 log side by side on a wide window, each scrolling on its own.
- Operação words untranslated; Valor as "1.234,56" in both languages; dates `dd/mm/aaaa`.
- A spinner before every slow answer (the switch, the row buttons, Reler).

## Free to redesign

The styling of cards and tiles; the arrangement of the metrics tiles; how the band's two cards sit (side by side, tabs, a strip), as long as both stay on this screen; the form of a cycle row (list, timeline, strip of marks); the dialog's chrome; the form of the narrow-window switcher; icons.

## Sample content for mockups

- Card: "Serviço · baixando" · "As NF-e do portal ACOM — sobe com o app e fica no ar; só Desligar derruba" · "Último ciclo: 09/09/2026 · 14:31 · 3 notas no ciclo · 1482 ciclos no total". Foreign variant: "Serviço · no ar — subiu fora deste app".
- Métricas, "exportadas em 09/09/2026 · 14:20": ciclos 1.480 · bem-sucedidos 1.477 · falhas 3 (amber) · notas baixadas 1.206 · duração do último ciclo 12,4 s · notas no último ciclo 3.
- Fila de sincronização, "1.208 eventos": Pendentes 1.206 · Processando 0 · Enviadas 0 · Com falha 2 (amber). Falhas de sincronização: "#1187 · nfe_downloads_queue · 5 tentativas · 08/09/2026 · 17:02", with "Error 111 connecting to localhost:6379. Connection refused." under it (illustrative error text).
- Ciclos: "1.482 ciclos desde 03/08/2026 · 3 com falha"; rows "09/09/2026 · 14:31 · 12,4 s · 3 notas" (ok), "09/09/2026 · 14:30 · 9,8 s · 0 notas" (ok), "09/09/2026 · 14:29 · 31 s · 0 notas · falhou" (failed) with "Authentication failed" under it.
- Notas baixadas, count line "1.208 baixadas — as mais recentes primeiro":

| Nota | Cliente | Operação | Valor | Emitida em |
| --- | --- | --- | --- | --- |
| 100246 | HOSPITAL SANTA AURORA | Venda | 3.215,40 | 03/09/2026 |
| 100245 | HOSPITAL VALE VERDE | Venda | 12.480,00 | 02/09/2026 |
| 100239 | INSTITUTO ORTOPEDICO HORIZONTE | Devolução | 842,10 | 01/09/2026 |
| 100231 | HOSPITAL INFANTIL SOL NASCENTE | Remessa |  | 31/08/2026 |
| 100228 | HOSPITAL VALE VERDE | OUTRA SAIDA NAO ESPECIFICADA | 150,00 | 29/08/2026 |

- Dialog: a code icon, the file name `NFe-100245.xml` (illustrative; the real name is whatever the monitor recorded), "Esc fecha", expand, close. An excerpt of the body as indented for reading:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<nfeProc versao="4.00">
  <NFe>
    <infNFe versao="4.00">
      <ide>
        <natOp>VENDA DE MERCADORIA</natOp>
        <nNF>100245</nNF>
        <dhEmi>2026-09-02T10:14:00-03:00</dhEmi>
      </ide>
      <emit>
        <xNome>DISTRIBUIDORA EXEMPLO LTDA</xNome>
      </emit>
      <total>
        <ICMSTot>
          <vNF>12480.00</vNF>
        </ICMSTot>
      </total>
      <infAdic>
        <infCpl>PACIENTE: MARTA LEMOS FERREIRA · ATEND. 2026004517</infCpl>
      </infAdic>
```

- Log pane (⚠️ the monitor's line format was not verified; draw it as opaque text): `5120  [2026-09-09 14:31:58] [INFO] cycle finished — 3 NF-e`.

## Design opportunities

1. **Say the heartbeat's age** ("há 40 s") beside its stamp. The known trap (process up, portal login failing) shows only as a stamp that stops moving; an age makes that readable at a glance without interpreting anything. Colouring it past a threshold would be a judgment call (see Open questions).
2. **Keep the listing steady across cycles.** Each new cycle restarts the stream from the top, so a filtered or paged view shrinks to the newest 500 rows and regrows every cycle. Adding only the new rows on top would keep the operator's place (needs engineering).
3. **Stamp the band's read** ("lido às hh:mm"). It reloads only on successful cycles and on Reler, so a streak of failed cycles stays invisible until the next success or a manual Reler.
4. **One Reler for the band**, placed on the band. Today it sits on the Fila card and reloads both cards.
5. **Group the heartbeat's numbers** as the Ciclos line does ("1.482", where the heartbeat line shows "1482" today).
6. **Show the failed item's last attempt.** The backend reads it; the card does not draw it.
7. **Make "stays up after the app closes" impossible to miss** where it matters: next to a Desligar that asks for no confirmation, and perhaps when the app closes with the monitor running (proposal).
8. **Move the item out of "Ideias futuras".** The screen is built, and the group's name says the opposite.
9. **A dialog header with the note's facts** (Nota, Operação, Emitida em, Valor) beside the file name, which alone means little to a person.
10. **Counts in the narrow switcher** ("Notas baixadas · 1.208").
11. **Localize the opening refusals**: they reach the screen in English today (this needs i18n keys per refusal).

## Open questions

- Desligar asks for no confirmation, while the Fantasma's Derrubar does. Is that intended, given that the monitor can simply be raised again?
- Should the "falhas" metrics tile stay amber for a lifetime count (one failed cycle ever keeps it amber), when the band's rule is that only current queue failures warn?
- Should failed cycles also reload the band? Today only successful cycles move the reload clock.
- The heartbeat's age: show it only, or also warn past N times the cycle interval? And who sets N?
- Should the log pane get the Logs screen's filter and copy?
