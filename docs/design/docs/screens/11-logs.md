# Logs — `/logs`

> Where: sidebar group **Sistema** › **Logs** (below Configurações); also opened from the dashboard's Serviços card, a global-search log hit and the Avisos screen's footer · Status: built · Code: `src/routes/logs/+page.svelte`, `src/lib/components/FilterField.svelte`, `src/lib/format.ts` (`shownLogStamp`), `src/lib/ipc.ts` (`readAppLog`, `readMedlensLog`, `readSpookLog`, `readNfepackLog`, `readFantasmaLog`), `src-tauri/src/journal.rs`, `src-tauri/src/lib.rs` (`read_app_log`, `read_medlens_log`, `read_spook_log`, `read_nfepack_log`, `tail_lines`), `src-tauri/src/fantasma.rs` (`read_log`), `src/lib/i18n/locales/pt-BR.ts` (`logs`)

## The question

_O que os serviços disseram?_ (`question.logs`; the sentence itself is not drawn on this screen) — and what did the app itself do. When something "didn't happen", the answer is in one log or another, almost never in both, so the screen shows one source at a time and always says which one.

## Where it sits in the work

Outside the conveyor, and read-only: nothing on this screen changes anything. The operator arrives with a symptom from somewhere else:

| From | Lands on source |
| --- | --- |
| Sidebar › Sistema › Logs | Aplicativo |
| [Dashboard](01-dashboard.md), Serviços card: a service's "Logs" door (MedLens and Spook when their block reads up; the Fantasma per its own rule) | `?source=medlens`, `?source=spook`, `?source=fantasmaSession` |
| [Busca](18-search.md): a hit in "Linhas de log" | `?source=<the hit's source>`: the source only, not the line |
| [Avisos](17-notices.md): "Ver o histórico completo nos Logs" | Aplicativo |

The NF-e monitor's full cycle-by-cycle log lives on [NFePackMonitor](12-nfe-pack-monitor.md); here that source shows errors only.

## Anatomy

1. **Header.** Left: the title "Logs" and, under it, a one-line hint that changes with the chosen source (table below). Right: a segmented control with the five sources, in this order: "Aplicativo", "MedLens · Serviço", "Spook · Serviço", "NFe Pack · Serviço", "Fantasma · Sessão". The chosen segment is tinted with the accent, never flooded with it. **[functional]** the chosen source is always visible and named.
2. **Toolbar**, one row:
   - the screen's filter field: funnel icon, placeholder "Filtrar linhas…", an X that clears it while it holds text, and the "Ctrl ⇧ K" hint ([05-components](../05-components.md) › FilterField, declared `primary`: this is the screen's only filter, so it owns `Ctrl ⇧ K`);
   - "Reler" (arrows-clockwise icon);
   - the follow toggle (clock icon, a switch): it reads "Seguindo" while on, accent-tinted, which is the default, and "Seguir" while off;
   - "Copiar" (copy icon): it reads "Copiado" for 1.5 s after a successful copy, and is disabled while no line is shown;
   - a count, right-aligned (see Data shown).
3. **Body.** One bordered panel in monospace (11.5 px today), one row per line: the line number in a right-aligned 44 px gutter (faint, not selectable), then the text (wraps; long tokens break). Rows tint on hover. **The newest line is at the top.** **[functional]** newest-first order with the number gutter. When there is nothing to show, the panel is replaced by a bordered box with one centred sentence (see States).

| Segment | Hint under the title (verbatim) | What is read | Cut |
| --- | --- | --- | --- |
| Aplicativo | "O diário do próprio app — o que aconteceu nesta estação, evento por evento." | the app's own journal: one file per UTC day in the app's data folder | the last 1000 lines across the daily files; files older than 30 days are deleted at startup |
| MedLens · Serviço | "O log do uvicorn, na pasta do projeto — o que o serviço de leitura disse, cru." | `logs/medlens.log` inside the configured MedLens folder | the last 1000 lines |
| Spook · Serviço | "Tudo o que o Spook imprimiu — a subida e cada leitura do portal depois dela, cru." | `logs/spook-spawn.log` in the app's data folder: everything printed by the Spook this app raised | the last 1000 lines |
| NFe Pack · Serviço | "Só os erros do monitor, crus — o log completo, ciclo a ciclo, vive na tela NFePackMonitor." | `logs/nfepackmonitor.log` inside the configured monitor folder | only the lines carrying the logger's own `] [ERROR]` or `] [CRITICAL]` stamp, selected **before** the 1000-line cap |
| Fantasma · Sessão | "O que este app fez com o robô — ligar, entrar no Everest, derrubar." | `logs/fantasma-session.log` in the app's data folder, written by this app's robot driver | the last 1000 lines |

## Data shown

| Field | Meaning | Presentation | Example |
| --- | --- | --- | --- |
| Aplicativo line | one journal event | `dd/mm/aaaa hh:mm:ss` (local time, with seconds), two spaces, the event identifier, then ` — detail` when the event has a detail | `09/09/2026 13:58:12  preferences_saved — profile.avatar, profile.avatar_seed` |
| Event | what happened, as a stored English `snake_case` identifier | shown as stored: not translated, not humanized | `session_locked` |
| Detail | an optional short English complement | after an em dash | `openrouter_key` |
| Service line | one line of the service's own file | verbatim, never parsed | see the samples |
| Line number | position inside the window read: 1 = the oldest line returned | chronological, kept through the newest-first inversion and through the filter. ⚠️ It is not the file's absolute line number: once the file outgrows the 1000-line window, the numbers move with the window. For NFe Pack it counts error lines only | `214` |
| Count | how many lines the source returned | "N linhas" without a filter, "shown / N linhas" with one; no thousands separator | `214 linhas` · `2 / 214 linhas` |

## States

| State | Trigger | What is drawn |
| --- | --- | --- |
| Empty: Aplicativo | journal absent or empty | "Nada registrado ainda. Os eventos do app aparecem aqui conforme acontecem." |
| Empty: MedLens · Serviço | no folder configured, or the service never rose (no file) | "Sem o serviço ter subido nesta máquina, não há arquivo — o log do uvicorn nasce na primeira subida." |
| Empty: Spook · Serviço | this app never raised the Spook (no file) | "Este app nunca subiu o Spook nesta máquina — um Spook de outra pessoa escreve o log dela, não aqui." |
| Empty: NFe Pack · Serviço | no folder, no file, or no error line in it | "Nenhum erro no log do monitor — o log completo, ciclo a ciclo, vive na tela NFePackMonitor." |
| Empty: Fantasma · Sessão | the robot was never driven from this machine (no file) | "O robô nunca foi dirigido desta máquina — as linhas nascem no primeiro Ligar." |
| Filtered to nothing | the source has lines and the filter matches none | "Nenhuma linha passa no filtro." The count reads `0 / N linhas` and Copiar is disabled |
| Service not configured | the MedLens or monitor folder is blank | the source's empty sentence: the screen does not tell "not configured" from "no file" |
| Before the first read answers | the screen just opened | the current source's empty sentence; there is no "reading…" state (the reads are local files and answer quickly) |
| Following (default) | toggle on | re-read every 2 s; whenever the number of shown lines changes, the panel scrolls back to the top |
| Not following | toggle off | nothing is re-read until "Reler" |
| Copied | the clipboard accepted | "Copiado" for 1.5 s, then "Copiar" again; a refused clipboard changes nothing, and clicking again is the retry |
| Read refused | a read rejects (possible only when the app's own data or configuration folder is unavailable) | no error state is drawn; the previous lines stay |
| Narrow window | — | no breakpoint: the header and toolbar never wrap (the filter is 320 px wide and the five segments do not shrink), so check 1024 px |

## Actions

| Gesture | Where | Precondition | Effect | Feedback |
| --- | --- | --- | --- | --- |
| Choose a source | segmented control | — | reads that source at once and restarts the 2 s follow; the filter text is kept; the URL is not rewritten | the hint line and the pressed segment change |
| Filter | toolbar field | — | keeps the lines whose drawn text contains the typed text, case-insensitive (for Aplicativo the stamp, event and detail are all searchable); lines are not renumbered | the count becomes `shown / N linhas` |
| Clear the filter | the X inside the field | the field holds text | empties it | — |
| `Ctrl ⇧ K` | anywhere on the screen | — | focuses and selects the filter | — |
| Reler | toolbar | — | one re-read of the current source | none (no spinner); the lines just change |
| Seguir / Seguindo | toolbar | — | toggles the 2 s re-read and the pin to the top | label and tint |
| Copiar | toolbar | at least one line shown | copies the shown lines' text: filtered, newest first, one per line, **without** the numbers | "Copiado" for 1.5 s |
| Deep link | the URL's `?source=` with `app`, `medlens`, `spook`, `nfepack` or `fantasmaSession` | — | opens on that source; any other value opens Aplicativo; read once, when the screen opens | — |

No gesture here writes anything, so there is no confirmation and nothing to undo.

## Business rules

1. **The source is part of the datum.** Logs are never stacked into one stream without saying where each line came from: three stacked answers become none (`docs/telas.md` § `/logs`; [02-invariants](../02-invariants.md)).
2. **There is no "Container" source, and that is a finding, not an omission.** Everything the robot does runs through `docker compose exec`, whose output goes to whoever called it. The predecessor app's Container tab answered "nada impresso" for months (header comment of `src/routes/logs/+page.svelte`).
3. **A service's log is never interpreted.** Lines are shown raw, and no state is derived from their text. The NFe Pack cut is a selection by the stamp the monitor's own logger writes (`[ERROR]`/`[CRITICAL]`), never a reading of the sentences (decisions 033 and 034).
4. **NFe Pack shows errors only here** (the operator's call, 26/08/2026), selected before the cap. If the cap were taken first, a thousand INFO lines would push every error out of reach. The full log is on [NFePackMonitor](12-nfe-pack-monitor.md).
5. **The Spook has one file, not two.** The service writes no log of its own; this app redirects its whole output (the rise and every portal read after it) into one file. A Spook that somebody else raised writes its log elsewhere, and the empty sentence says so.
6. **Newest on top, real numbers.** What just happened is where the eye lands (the operator's call, 05/08/2026). The numbers stay chronological under the inversion and under the filter, so "linha 214" means the same line to two people (decision 009).
7. **A journal line says _that_ something happened, never _what with_.** The event is an English `snake_case` identifier (at most 64 characters, starting with a lowercase letter); the detail is at most 500 characters of short English. No e-mail addresses, codes, hashes, secret values or informe file names (a file name carries a patient's name). The only value-like details allowed are stored names from a closed set, such as which wallet entry or which preference fields changed. `journal.rs` enforces this for every writer (decision 009; LGPD).
8. **The journal never fails the act it records**, and an absent or mangled journal reads as fewer lines, never as an error.
9. **A tail, not an archive.** Each read returns at most 1000 lines. The journal files keep 30 days on disk for whoever needs to search deeper outside the app.
10. **Poll only while open and following**: every 2 s, and leaving the screen stops it.
11. **Every notice behind the bell is also a journal line** (`notice_posted — <kind>`): the Avisos screen promises the full story here ([Avisos](17-notices.md)).

## Must survive the redesign

- Five sources, one at a time, with the chosen one always named. No Container source, and no merged stream without per-line provenance.
- Newest first, with the real chronological numbers, including under the filter.
- Lines verbatim: no badges, colours or summaries derived from a service line's text (see Open questions about the logger's own level stamp).
- NFe Pack · Serviço is errors-only here and says that the full log is on the NFePackMonitor screen.
- A distinct empty sentence per source that says why there is nothing: absence is not an error.
- Follow re-reads only while the screen is open and the toggle is on; "Reler" stays as the manual gesture.
- Journal lines never carry values. No redesign may propose showing file names, e-mails, codes or patient names in Aplicativo lines.
- `Ctrl ⇧ K` focuses this screen's filter, and the hint is drawn on the field that owns it.

## Free to redesign

The source picker's form (segments, tabs, a list, a select); where the toolbar sits; the monospace face and size; the gutter and hover styling; where the count sits; how the three structured parts of an Aplicativo line (stamp, event, detail) are laid out, since they are fields, not interpreted text.

## Sample content for mockups

Aplicativo, newest first (the event names are real identifiers; the stamps are fictitious):

```
214  09/09/2026 14:32:07  notice_posted — nfepack_errors
213  09/09/2026 14:05:51  bill_queued
212  09/09/2026 13:58:12  preferences_saved — profile.avatar, profile.avatar_seed
211  09/09/2026 13:40:03  session_unlocked
210  09/09/2026 13:22:40  session_locked
209  09/09/2026 13:22:40  idle_autolock
208  09/09/2026 11:17:19  catalog_collected
207  09/09/2026 08:41:30  secret_saved — openrouter_key
```

Service sources. ⚠️ The exact line formats of the services were not verified; draw them as opaque text:

```
NFe Pack · Serviço   [2026-09-09 14:31:58] [ERROR] Authentication failed
MedLens · Serviço    INFO:     127.0.0.1:51822 - "GET /health HTTP/1.1" 200 OK
```

Counts: `214 linhas`; with the filter `session`: `2 / 214 linhas`.

## Design opportunities

1. **Say when the window is cut.** At the cap the count reads "1000 linhas", which cannot be told apart from a file of exactly 1000 lines. The app's own standard elsewhere is "no silent caps" ([Busca](18-search.md) says "mostrando 80 de N"). Proposal: "últimas 1000 linhas" when the read was cut (the backend would have to report the cut).
2. **Copy with the numbers**, or offer both. The numbers are the shared reference ("linha 214"), and the copy drops them today.
3. **Pause following when the operator scrolls away from the top**, with a "voltar ao mais recente" affordance. Today the pin fires only when the line count changes, and at the 1000-line cap the count stops changing.
4. **Lay the Aplicativo journal out in columns** (stamp · event · detail), and consider a pt-BR gloss beside each event identifier. The identifier itself stays, verbatim.
5. **Tell "no folder configured", "no file yet" and "file without lines" apart** in the empty states (MedLens and NFe Pack collapse them today). The backend would have to say which.
6. **Land on the line** when arriving from Busca (highlight and scroll to it) instead of the top of the source.
7. **A read-failure state.** Today a refused read draws nothing.
8. **Day separators in the Aplicativo journal**: the stamps already carry the date.
9. **A narrow layout.** At 1024 px, five segments and a fixed 320 px filter crowd one row.

## Open questions

- Is highlighting by the logger's **own level stamp** (for example `[ERROR]`) acceptable, as the same "selection by field" that the NFe Pack cut already uses? Or does any colouring count as interpreting the log?
- Should Aplicativo events be worded in pt-BR on screen, while the stored names stay English?
- Should NFe Pack · Serviço also offer the full log here (a "só erros" toggle), or stay errors-only as the operator decided on 26/08/2026?
- Should "not read yet" look different from "empty" on the first paint?
