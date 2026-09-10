# Dashboard — `/dashboard`

> Where: sidebar, first item — alone above the group kickers, label "Dashboard" (gauge icon); also the app's landing route once the first assembly is complete · Status: built (Pro decisions 037 + 040; 040 still awaits the operator's live validation) · Code: `src/routes/dashboard/+page.svelte`, `src/lib/service-blocks.ts`, the stores `src/lib/{billing,review,dock,catalogs,query,medlens,fantasma,spook,nfepack}.svelte.ts`, `src/lib/format.ts`, `src/lib/assembly.ts`

## The question

"O que precisa da minha atenção agora?" — the app's own wording (i18n `question.dashboard`). It is **not drawn** on the built screen (only the unbuilt-screen placeholder prints a question); `docs/telas.md` titles the screen "como está o dia?". Both name one job: in a single look, where the day's work is waiting, what only a human can unblock, where bills stopped, whether the services answer, and what today produced.

## Where it sits in the work

- **Outside the conveyor.** It reads; it never moves a bill, never writes, never starts or stops a service. Every clickable element is a door to the screen where that subject lives.
- **Landing route**: the root redirects here once the first assembly is complete (or was skipped for this session); the wizard exits here ([Primeira montagem](15-welcome.md)); the search's Esc falls back here when it has no origin ([Busca](18-search.md)).
- It reads the stores the conveyor screens own — dock ([Triagem](02-triage.md)), extraction summaries ([Revisão](03-review.md)), billing records ([Faturamento](04-billing.md)), catalog summaries ([Catálogos](08-catalogs.md)), the query trail ([Consulta](06-query.md)) — plus the service statuses the status bar polls ([shell](../04-shell.md)).

| Element | Door |
|---|---|
| "Começar pela doca" | `/triage` |
| Tile "Na doca" | `/triage` |
| Tile "A conferir" | `/review` |
| Tiles "Esperando o robô" and "Faturadas hoje" | `/billing` |
| "Só anda com você" — dock rows | `/triage` |
| "Só anda com você" — bill rows | `/billing` (the list; the bill is not pre-selected) |
| Duplicate row | `/review/{id}` of the first file in the group |
| "Duplicidades ↗" | `/duplicates` — an empty study ([future studies](14-future-studies.md)) |
| Serviços rows | per state, see § Anatomy 3e |
| Catálogos row "Abrir" | `/catalogs` |

## Anatomy

One vertically scrolling page. Reading order:

### 1. Header
- **Greeting** (the page's only heading): "Bom dia" before 12:00, "Boa tarde" before 18:00, "Boa noite" from 18:00 — followed by ", {first name}" when the profile has a name (first word of the profile name): "Boa tarde, Camila". Re-evaluated on the 30 s clock.
- **Sub line**: `{long local date} · {day summary}`. Date in the locale's long form: "quinta-feira, 10 de setembro de 2026". Day summary: "nenhuma peça nova desde ontem" / "1 peça nova desde ontem" / "N peças novas desde ontem"; when MedLens is not up, " · nada é lido enquanto o MedLens estiver fora" is appended.
- **Button** at the right end: "Começar pela doca" + arrow → `/triage`. The only filled (accent) button on the screen.

### 2. Failure strip — conditional **[functional]**
Directly under the header: one red line per source whose last read was refused, in this order — dock, review, billing, catalogs, MedLens spend. Each line is the refusing side's own sentence, verbatim, with no prefix naming the source. Absent when nothing failed. The cards below keep their last good answer (or zeros if that source never answered).

### 3. Two columns
A wide left column and a 336 px right column; below a 1180 px window width they stack into one column (left stack first). Cards are bordered surfaces; the funnel tiles are buttons.

**Left column**

**3a. Funnel — four tiles in work order [functional: dock → review → robot → billed].** Four across; 2 × 2 below 900 px. Each tile: icon + uppercase stage label; large number + unit; optional note line. Hover outlines the tile in accent.

| Tile | Label | Unit | Note | Tone | Door |
|---|---|---|---|---|---|
| Dock (tray icon) | "Na doca" | "peça" / "peças" | "N precisa de decisão" / "N precisam de decisão" when N > 0; otherwise none | amber border, number and note when N > 0 | `/triage` |
| Review (checklist icon) | "A conferir" | "conta" / "contas" | "em N arquivo" / "em N arquivos" when the count > 0 | plain | `/review` |
| Robot (receipt icon) | "Esperando o robô" | "conta" / "contas" | "na fila do Fantasma"; "Fantasma falta configurar" when the Fantasma setup is incomplete | amber border, number and note only when unconfigured | `/billing` |
| Billed (check-circle icon) | "Faturadas hoje" | "conta" / "contas" | none | number always green, zero included | `/billing` |

**3b. Card "Só anda com você".** Header: hand icon (amber), title "Só anda com você", count pill (amber, number of rows — not of pieces) when there are rows, kicker at the right "o robô não decide nenhuma destas". Body: one full-width button per row — status icon (amber warning or red warning-circle), a "what" sentence, a "where" line under it, an optional age at the right, a trailing arrow. Empty: "Nada exige você agora."

**3c. Card "Duplicidade de contas".** Header: copy icon (amber), title, count pill "N suspeita" / "N suspeitas", kicker "a mesma conta entrando duas vezes". Row (button): patient name + "uso em {date}"; under it one chip per arrival "{file name} · {client id}" (tooltip: the file name); at the right a pill — red "já no Faturamento" or amber "nenhuma faturada" — and an arrow. Empty: "Nenhuma suspeita agora — a comparação olha paciente e data de uso nos arquivos em revisão." Footer (always): info icon + "Comparação crua: mesmo paciente e mesma data de uso, em arquivos diferentes. Nada é bloqueado nem descartado sozinho — nem existe gesto para isso ainda." + small outlined door "Duplicidades ↗".

**3d. Card "API do MedLens".** Header (wraps when narrow): title "API do MedLens"; the 14-day total in mono accent ("US$ 3,42"); kicker "últimos 14 dias" + " · pico em dd/mm" when anything was spent; at the right a two-segment tab control "Por dia" | "Por modelo" (default "Por dia", not remembered between visits). Body by tab and state (§ States). Footer (always): info icon + "Só chamadas do MedLens: a Consulta e o Fantasma não gastam API. O gasto vem da OpenRouter, com alguns minutos de atraso, e só da chave configurada agora."
- *Por dia*: 14 vertical bars, oldest at the left, today at the right, each labelled with its day of month. Height proportional to that day's cost against the window's highest day (minimum 4 %); a day with nothing is a 3 % stub in the empty-track color; the peak day in full accent, the others in a darker accent. Tooltip per bar: "{aaaa-mm-dd} · US$ 0,42" or "{aaaa-mm-dd} · nada lido" (ISO date in the current build).
- *Por modelo*: one row per model id, most expensive first — model id (mono, truncated), "N leitura" / "N leituras" under it, a horizontal bar scaled to the most expensive model (minimum 3 %), the cost, and the share of the window total rounded to a whole "%".

**Right column**

**3e. Card "Serviços".** Title only. Six rows: a status dot, the name, a state line under it, and one small outlined door button whose word and target follow the state. The five service rows come from the same function the status bar uses, so the words and tones are identical to the bar's.

| Row | State line → dot | Door when green (Fantasma: also when busy) | Door otherwise |
|---|---|---|---|
| "MailBridgr" | "canal confirmado · doca a caminho" → green (the profile e-mail was confirmed — a stored fact); "doca ainda não vigiada" → grey | "Abrir" → `/triage` | "Configurar" → Configurações · Doca |
| "MedLens" | "…" grey (never answered); "subindo…" accent; "sem credencial — 401 em tudo" amber (running but credential rejected); "no ar" green; "no ar · de fora do app" green (a MedLens this app did not start); "parado" grey | "Logs" → Logs · MedLens | "Configurar" → Configurações · MedLens |
| "Fantasma" | "falta configurar" amber (setup incomplete — checked before any live state); "…" grey; "ocupado" accent (the robot's lock is held — overrides every other word); "pronto" green; "sem sessão" amber (container up, Everest not logged in); "subindo…" / "desligando…" accent; "de fora do app" grey; "parado" grey | "Logs" → Logs · Fantasma · Sessão | "Configurar" → Configurações · Fantasma |
| "Spook" | "…" grey; "sem endereço" grey; "parado" grey (local, not answering); "sem resposta" amber (remote, not answering); "credencial recusada" red; "sem credencial" amber; "no ar" green | "Logs" → Logs · Spook | "Configurar" → Configurações · Spook |
| "NFe Pack" | "…" grey; "sem pasta" grey; "baixando" green (+ " · de fora do app" when another process runs it); "parado" grey | "Abrir" → `/nfe-pack-monitor` | "Configurar" → Configurações · NFe Pack |
| "Catálogos" | "espelho de {dd/mm/aaaa · hh:mm} · N de M catálogos" green (newest collection; how many catalogs hold a mirror, of all known); "sem espelho — colete quando o Fantasma estiver pronto" grey | "Abrir" → `/catalogs` (always) | — |

Dot colors: green ok · amber needs attention · red refused · accent in motion/busy · grey idle or unknown.

**3f. Card "Hoje".** Title "Hoje"; three lines, each a mono value and a label:
1. `{N}` "contas faturadas — pedido criado no Everest" — value green when > 0.
2. `{N}` "contas aprovadas e entregues ao robô".
3. `{US$ …}` "gasto do MedLens hoje" — or "—" (muted) with "gasto do MedLens hoje" when MedLens is up but the spend has not answered, or "—" with "gasto do MedLens — serviço parado, sem como perguntar" when MedLens is not up.

**3g. Card "Últimas leituras".** Title; four rows: icon, what, stamp at the right (mono). Stamps are `dd/mm/aaaa · hh:mm`, local.

| Row | Stamp | "nunca" (amber) when | "—" (muted) when |
|---|---|---|---|
| "Catálogos do ERP" | newest catalog collection | no catalog was ever collected | never |
| "Consulta ao Everest" | newest entry of the query trail | the trail is empty (also when it could not be read) | never |
| "Leitura do MedLens" | newest spend row (any age) | the spend answered with no rows at all | the spend has not answered during this visit |
| "Ciclo do NFe Pack" | the monitor's last heartbeat | never | no heartbeat / status not answered |

**3h. Card "Onde as contas estão paradas".** Title; sub line "nenhuma conta em voo" or "N conta em voo · estágio no robô" / "N contas em voo · estágio no robô". Five rows: label (the Faturamento's stage words) · horizontal track with a bar · optional red pill "parada" · count (mono). Rows: "No Faturamento", "Conferência", "Encerrada", "Pedido criado", "Faturadas hoje". Bars scale to the largest row; accent normally, red when the bucket is marked "parada", green on the "Faturadas hoje" row.

No shared component from [05-components](../05-components.md) is used: empty sentences are plain inline lines, not `EmptyState`.

## Data shown

Every number below is computed on the front from a store that already exists; nothing here has a source of its own.

| Number / field | Counts exactly | Presentation rule | Example |
|---|---|---|---|
| Day summary | dock pieces not ignored whose `since_ms` — the instant the piece **entered its current state** — is at or after yesterday's local midnight. A piece extracted today counts, however old its arrival. | 0 → "nenhuma peça nova desde ontem"; 1 → "1 peça nova…"; N → "N peças novas…" | "4 peças novas desde ontem" |
| Na doca | pieces of every shipment in state no client, extraction refused, waiting for extraction (has adapter) or extracting — everything not extracted and not ignored | number + unit | 5 peças |
| … "precisa(m) de decisão" | the subset with no client or with a refused extraction — the two states the Triagem says require a human | amber only when > 0 | "2 precisam de decisão" |
| A conferir | accounts (bills inside an extraction) on the review list that are not marked "não enviar", whose file is not waiting in the Fantasma's queue folder, and that carry no billing outcome word. The review list holds only extractions that came through this app's dock and did not end in error. | number + unit | 7 contas |
| … "em N arquivos" | distinct extraction files holding at least one such account | shown only when the count > 0 | "em 3 arquivos" |
| Esperando o robô | billing records not finished — not yet delivered, delivered and waiting, running, failed, needing investigation, or stopped mid-run all count | number + unit | 4 contas |
| Faturadas hoje | billing records that are finished **and** carry an order number **and** whose point-of-no-return stamp is at or after today's local midnight. A Rule 1 ending (the Everest already had the bill) is finished with no order and is excluded. | green number; same value in three places (funnel, Hoje line 1, stalled row 5) | 2 contas |
| Aprovadas hoje (Hoje line 2) | billing records whose approval stamp is today (local). Counts approvals whether or not the delivery to the robot succeeded; a bill withdrawn afterwards leaves the count, because withdrawal deletes its record. | plain number | 5 |
| Gasto hoje (Hoje line 3) | sum of MedLens spend rows whose date falls on today's local date | US$ format; "—" when unknown | US$ 0,38 |
| Asks count pill | rows in "Só anda com você" (one row for all no-client pieces, one for all refused pieces, one per bill) | amber pill; absent at 0 | 4 |
| Ask age | now minus the oldest `since_ms` of the pieces in that row | "agora há pouco" (< 1 min), "há N min", "há N h" (< 24 h), "há N d" | "há 3 h" |
| Duplicate group | accounts on the review list, "não enviar" excluded, grouped by trimmed patient name (case-insensitive) + surgery date as read; a group shows only when its accounts span **two or more different files**. Queued and worked accounts stay in. | patient as first read; date as stored; one chip per account | 2 suspeitas |
| Duplicate pill | red "já no Faturamento" when any account of the group is queued or worked; otherwise amber "nenhuma faturada" | — | — |
| Spend total | sum of spend rows dated within the last 14 local days, today included | "US$ 1,23"; below one cent four decimals ("US$ 0,0042"); zero "US$ 0,00" | US$ 3,42 |
| Peak | the day with the highest sum in the window | " · pico em dd/mm" only when total > 0 | " · pico em 07/09" |
| Per-model readings | spend rows in the window for that model id (one row per extraction) | "N leitura(s)" | 38 leituras |
| Per-model share | model cost ÷ window total, rounded | whole percent | 79% |
| Stalled buckets | unfinished billing records grouped by stage — the stored stage, or the live phase when it is ahead — capped at "Pedido criado" | count + bar | No Faturamento 2 |
| "parada" mark | the bucket holds at least one bill that failed, stopped mid-run, or needs investigation | red pill + red bar | — |
| In flight (stalled sub) | same set as "Esperando o robô" | "N conta(s) em voo · estágio no robô" | 4 contas em voo |

## States

| State | Trigger | What is drawn |
|---|---|---|
| First paint | the stores have not answered yet (every launch lands here first) | No loading face in the current build: every count reads 0, "Nada exige você agora.", the no-duplicates sentence, "nenhuma conta em voo", Catálogos "sem espelho…" and "nunca", Consulta "nunca"; service rows read "…" until the status bar's polls answer; until the wallet status answers, the robot tile and the Fantasma row read "falta configurar" (amber). Numbers fill in as each store answers. |
| Quiet refresh | every 30 s while the screen is open | Nothing signals it; numbers change in place. Billing, review, dock, catalogs and the query trail are re-read on this clock; the service statuses ride the status bar's own polls (MedLens 5 s, Fantasma 20 s, Spook 30 s, NFe Pack 30 s). |
| One source refused | a store's read failed | Red line in the failure strip with that side's sentence; that source's cards keep the last good data (zeros if it never answered). Other cards are unaffected. |
| MedLens not up | status is not running/foreign | Day summary gains " · nada é lido enquanto o MedLens estiver fora"; spend card body "MedLens parado — o gasto não tem como ser perguntado." (only when nothing was loaded during this visit; otherwise the last bars stay) while its header still shows "US$ 0,00 · últimos 14 dias"; Hoje line 3 "—" + down label; "Leitura do MedLens" "—" unless loaded earlier; MedLens service row per its state. The review list usually refuses too, which adds a failure line. |
| MedLens up, spend refused | the spend call failed | Failure line; "Por dia" shows 14 empty stubs and total "US$ 0,00"; "Por modelo" "nada lido nos últimos 14 dias"; Hoje "—" with the normal label; "Leitura do MedLens" "—". |
| Nothing spent in 14 days | spend answered, window empty | 14 stubs, "US$ 0,00", kicker without the peak; "Por modelo": "nada lido nos últimos 14 dias". |
| Spend freshness | — | The spend is asked once when MedLens is seen rising and once per visit; the 30 s clock does not re-ask it. |
| Fantasma unconfigured | the Fantasma setup (folders, users, both passwords) is incomplete | Robot tile amber with "Fantasma falta configurar"; Serviços "falta configurar" amber + "Configurar". |
| Fantasma busy | its single-flight lock is held (a run, a catalog collection, a query) | Serviços "ocupado" in accent + "Logs". |
| Nothing needs a human | no rows | "Nada exige você agora." — no count pill. |
| No duplicates | no group spans two files | "Nenhuma suspeita agora — a comparação olha paciente e data de uso nos arquivos em revisão." — footer stays. |
| No bill in flight | every record finished | "nenhuma conta em voo"; four zero rows + the billed row. |
| No catalog ever collected | — | Serviços "sem espelho — colete quando o Fantasma estiver pronto" (grey); readings "Catálogos do ERP · nunca" (amber). |
| Evening / morning | local hour | The greeting word changes; nothing else depends on the hour. |
| Narrow window | < 1180 px / < 900 px | One column / funnel 2 × 2. Service and asks rows wrap their text. |

## Actions

Every gesture is navigation; none writes, none needs confirmation, all are reversible by going back.

| Gesture | Where | Precondition | Effect | Feedback |
|---|---|---|---|---|
| Click "Começar pela doca" | header | none | opens `/triage` | route change |
| Click a funnel tile | funnel | none | opens its door (table in § Where it sits) | route change |
| Click an asks row | "Só anda com você" | none | dock rows → `/triage`; bill rows → `/billing` | route change |
| Click a duplicate row | "Duplicidade de contas" | none | opens the first file of the group in the Revisão | route change |
| Click "Duplicidades ↗" | duplicates footer | none | opens the empty study | route change |
| Switch "Por dia" / "Por modelo" | spend card | none | swaps the body; not persisted (back to "Por dia" on the next visit) | active tab outlined in accent |
| Click a service door | Serviços | none | "Abrir" / "Logs" / "Configurar" per the state table | route change |
| Hover a spend bar / a duplicate chip | — | — | tooltip (date + amount / file name) | tooltip |

No keyboard shortcut belongs to this screen; the global ones are the shell's ([shell](../04-shell.md)). All interactive elements are native buttons (Tab / Enter).

## Business rules

1. **Every number traces to an existing store, and a number with no source is not drawn.** "Itens conferidos por você" (in the original drawing) stays undrawn: corrections live in MedLens and nothing here keeps a per-day count (Pro decisions 037, 040; 006 § 2).
2. **Each source is asked on its own, and a refusal is named at the top, never dressed as an empty card.** A MedLens refusal must not hide the dock (the inherited "never mix a local call and a MedLens call" rule; 040).
3. **"Hoje" is the operator's local day, never UTC's** — the cut is made on the front, at local midnight (037).
4. **"Faturadas hoje" requires the order number.** A Rule 1 ending is finished but created nothing; counting it would claim an invoice this app never made (037).
5. **The token model holds here.** A bill that left the Revisão (queued, delivered, worked) counts only on the Faturamento side — never in "A conferir" too (023 § 6; 040).
6. **Amber means a human is required.** On the dock tile only the no-client + refused subset earns it; the robot tile turns amber only when the Fantasma is unconfigured (style manual; 040).
7. **Bill rows carry no age.** The billing record keeps no failure stamp; an invented age would lie. Only piece rows, which have a real stamp, say "há N h" (040).
8. **Service problems are not repeated in "Só anda com você".** The Serviços card beside it carries them; two places asserting the same failure is an inherited lesson (040).
9. **The duplicate card is a raw comparison, never a verdict.** It blocks nothing, discards nothing, offers no gesture, and says so in its footer. "Não enviar" accounts are out (already decided); queued/worked ones stay in, because a pair where one side already went to the robot is the dangerous one (040).
10. **The spend shows only the cuts its data holds** — per day and per model; per client and per outcome are not answerable from the spend rows (006 § 2). It is scoped to the key configured now, comes from OpenRouter minutes late, and the footer says both (011 caveat).
11. **A number nobody could ask is "—", never 0**; "nunca" (amber) is a source that exists and never ran; "—" (muted) is a stamp nobody could ask right now (037, 040).
12. **One truth for service states.** The Serviços rows and the status bar draw from the same function; a bar saying "no ar" beside a card saying "parado" would assert two worlds (040).
13. **Stored facts are not live states.** MailBridgr's row speaks from the stored e-mail confirmation; the Catálogos row is a stored stamp. Neither pretends to be a health check.
14. **Fantasma "ocupado" overrides every other word**, and an unconfigured Fantasma warns before any live state is consulted (012; telas.md).
15. **The stage words are the Faturamento's** ("No Faturamento · Conferência · Encerrada · Pedido criado") — one vocabulary on both screens.
16. **Nothing here moves or writes.** No button processes, approves, withdraws or starts anything; the robot's rule "one bill, one click, with confirmation" lives on the Faturamento only.

## Must survive the redesign

- The counting definitions of § Data shown, including what is excluded (Rule 1 endings from "faturadas", queued/worked accounts from "A conferir", discarded accounts from duplicates).
- Failures at the top, one per source, verbatim; the rest of the screen stays readable when one source fails.
- The three faces of "unknown": "—" (nobody could ask), "nunca" (never ran), and a real 0 — never merged.
- The tone grammar: amber only for what needs a human (decision-pending pieces, pieces without client, failed bills, unconfigured robot), red for refusals, stopped runs, investigations, stuck buckets and the "já no Faturamento" duplicate pill, green only for what really happened (billed, services up).
- The funnel's work order (dock → review → robot → billed) and that each tile is a door.
- "Só anda com você" lists work facts only, one row per bill, no age on bill rows, and its kicker "o robô não decide nenhuma destas".
- The duplicate card's footer sentence (raw comparison, nothing blocked or discarded) and the "já no Faturamento" vs "nenhuma faturada" distinction.
- The spend card's scope note and its two cuts; the "—" with the reason when MedLens is down.
- Service rows with the status bar's exact words and tones; the door follows the state (up → Logs/Abrir, otherwise → Configurar).
- Local-day cuts for "hoje".
- No gesture that writes; no number without a source.

## Free to redesign

- The layout: the two-column split, card order (except failures on top), card vs. list treatments, widths, breakpoints.
- The funnel tiles' look, icons, sizes; whether the number or the label leads.
- The greeting's styling and whether the start button stays in the header.
- Chart forms: the day bars, the per-model bars, the stalled-bucket bars (any honest proportional form works).
- Where "Hoje" lives, and whether "Faturadas hoje" keeps appearing three times.
- Tabs vs. toggle for the spend cuts; the chips' shape in the duplicate rows.
- Typography, spacing and the dot/pill shapes, within the tone grammar above.

## Sample content for mockups

Cast from [06-sample-data](../06-sample-data.md). Values are illustrative.

- Header: "Boa tarde, Camila" · "quinta-feira, 10 de setembro de 2026 · 4 peças novas desde ontem".
- Funnel: Na doca **5** peças — "2 precisam de decisão" (amber) · A conferir **7** contas — "em 3 arquivos" · Esperando o robô **4** contas — "na fila do Fantasma" · Faturadas hoje **2** contas.
- Só anda com você (pill 5):
  - amber — "2 peças da doca não têm cliente — sem adapter, o MedLens não sabe com que prompt ler" · "Triagem" · "há 3 h"
  - red — "1 peça teve a extração recusada — o motivo está escrito na peça" · "Triagem" · "há 25 min"
  - red — "A conta de JOÃO CARLOS PRADO parou no meio da corrida — recupere o desfecho antes de qualquer novo gesto" · "Faturamento · atend. 2026004733"
  - red — "A conta de CARLOS EDUARDO LIMA pede investigação — refazer do zero pode duplicar a nota" · "Faturamento · atend. 2026004715"
  - amber — "A conta de PEDRO HENRIQUE SALLES falhou no robô — o motivo está na linha do Faturamento" · "Faturamento · atend. 2026004701"
- Duplicidade (pill "2 suspeitas"):
  - "LUÍSA ANDRADE REIS" "uso em 2026-09-03" (the date as stored) — chips "informe_vale_verde_2026-09-02.pdf · vale_verde", "etiquetas_sala3.jpg · vale_verde" — amber "nenhuma faturada"
  - "MARTA LEMOS FERREIRA" "uso em 2026-09-01" — two chips — red "já no Faturamento"
- API do MedLens: "US$ 3,42" · "últimos 14 dias · pico em 07/09"; bars labelled 28 29 30 31 1 2 3 4 5 6 7 8 9 10 (28/08 → 10/09), three empty days. Por modelo: "exemplo/visao-rapida-2" — "38 leituras" — US$ 2,71 — 79% · "exemplo/visao-precisa-1" — "6 leituras" — US$ 0,71 — 21% (model ids fictitious).
- Serviços: MailBridgr ● "canal confirmado · doca a caminho" [Abrir] · MedLens ● "no ar" [Logs] · Fantasma ● "pronto" [Logs] · Spook ● "sem endereço" [Configurar] · NFe Pack ● "baixando" [Abrir] · Catálogos ● "espelho de 09/09/2026 · 18:42 · 5 de 6 catálogos" [Abrir].
- Hoje: 2 "contas faturadas — pedido criado no Everest" · 5 "contas aprovadas e entregues ao robô" · US$ 0,38 "gasto do MedLens hoje".
- Últimas leituras: Catálogos do ERP 09/09/2026 · 18:42 · Consulta ao Everest 10/09/2026 · 14:05 · Leitura do MedLens 10/09/2026 · 15:31 · Ciclo do NFe Pack 10/09/2026 · 15:30.
- Onde as contas estão paradas: "4 contas em voo · estágio no robô" — No Faturamento 3 ("parada") · Conferência 0 · Encerrada 1 ("parada") · Pedido criado 0 · Faturadas hoje 2.
- Failure line example (verbatim shape of a service refusal; wording illustrative): "MedLens não respondeu — conexão recusada".

## Design opportunities

1. **Say which source failed.** Failure lines are bare sentences; a failure from the review list and one from the spend read look alike. Prefixing the source (Revisão, Faturamento, Doca, Catálogos, Gasto do MedLens) would tell the operator which card is stale.
2. **A "not asked yet" face.** On launch the screen shows zeros and "nothing to do" sentences for a moment before the stores answer — the app's own rule is that whoever has not asked cannot answer "no". A skeleton or "…" per card until its source answers would honor it.
3. **Deep-link bill rows.** Bill rows in "Só anda com você" (and the stalled buckets) open the Faturamento list, which then selects its top row, not the bill named. A selection carried in the route would land on the right bill.
4. **One home for "Faturadas hoje".** The same number is drawn three times (funnel tile, Hoje line 1, stalled row 5).
5. **The spend header asserts "US$ 0,00" when nothing could be asked.** With MedLens down or the spend refused, the body says so but the header total still reads zero; "—" there would match rule 11.
6. **Dates on screen in dd/mm/aaaa.** The bar tooltips print ISO days and the duplicate rows print the surgery date as stored (ISO), while every other date in the app reads dd/mm/aaaa.
7. **"Peças novas desde ontem" counts state changes, not arrivals.** A piece extracted today counts as "nova". Either the sentence or the count should move (the shipments carry a real arrival stamp).
8. **Show freshness.** The 30 s clock is invisible; a quiet "lido há N s" per card or in the header would tell a stale screen from a quiet day.
9. **Split "Esperando o robô".** It mixes bills not yet delivered, waiting, running, failed and under investigation under "na fila do Fantasma"; a breakdown (or the stalled card beside it) could carry what the tile flattens.
10. **A green zero.** The "Faturadas hoje" tile paints 0 in green, while the Hoje card paints 0 plain; green should mean something happened.
11. **Point where the reason actually is.** "…falhou no robô — o motivo está na linha do Faturamento": on the Faturamento the reason sits in the detail panel, not in the row.
12. **The Fantasma "de fora do app" row offers "Configurar"**, although nothing is wrong with a foreign robot (the Faturamento treats it as ready); "Logs" may fit better.

## Open questions

- Should the screen draw its question ("O que precisa da minha atenção agora?")? Today no built screen does.
- While a bill is being processed, its record reads as "stopped mid-run" until the outcome arrives; the Faturamento hides that note for the bill it is running, but this screen does not, so a red "parou no meio da corrida" row (and a "parada" bucket) can appear during a healthy run. Should the design give "running now" its own face here? (Engineering has the details.)
- Should the MedLens spend be re-asked on the 30 s clock, or is "once per rise / per visit" the intended freshness?
- The "Duplicidades" door leads to an empty study; keep it, or hide it until the study exists?
- unverified: the exact number of catalogs the "N de M catálogos" line reports (the count comes from the catalog list, not from this screen).
