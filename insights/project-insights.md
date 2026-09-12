# Project insights — Bridgr Pro, read end to end

> **Developer-facing. Not uploaded to Claude Design.** Findings from reading
> the code of `~/Projects/bridgr` at `8aa7c35` (2026-08-28) to write the
> redesign brief, on 2026-09-10. Nothing was run: not the app, not the tests,
> not the services. Every item says where, why it matters, and a proposal —
> proposals, not decisions.
>
> **Verified** = re-read in the code by the coordinator. **Reported** = found
> by the screen-by-screen reading and plausible, not re-checked line by line.

## Before anything else: the locked window

The session PIN is a courtesy lock (decision 008), not a security boundary
against a determined person — but its one promise is written in the memory
and in `docs/telas.md`: _closing the window is never a way around the PIN_.
Two paths break it, and both put patient data on an unattended screen.

### A. Closing the window from the desktop skips the lock — Verified

- Where: `closeWindow()` in `src/lib/components/Titlebar.svelte` locks before
  closing, but only the app's own X and the menu's "Encerrar o app" call it.
  There is no close-requested handler anywhere (`CloseRequested`,
  `on_window_event`, `onCloseRequested` — none in `src/` or `src-tauri/src/`),
  and `lib.rs` only reacts to `RunEvent::Exit`.
- What: Alt+F4, the taskbar's close, or the compositor closing the window ends
  the app **without writing the lock**; with a PIN active, the next launch
  opens unlocked.
- Proposal: intercept the close in one place (Rust `WindowEvent::CloseRequested`
  or the window's `onCloseRequested`), lock first when a PIN exists, then let
  the close proceed — and drop the Titlebar's special case.

### B. A locked window paints the app before the lock — Verified

- Where: `LockStore` in `src/lib/lock.svelte.ts` starts unlocked
  (`#lockedAt = null`); `lock.start()` runs inside a root-layout `$effect`
  (`src/routes/+layout.svelte`), i.e. after the first paint.
- What: on a restart while locked, the landing screen (with its patient
  names) renders until `sessionLockState()` answers; the lock screen itself
  shows its no-PIN panel until the wallet answers.
- Proposal: a third state, "lock not known yet", during which the layout draws
  nothing but the frame — the house rule _whoever has not asked cannot answer
  "no"_, applied to the lock.

## Keys that act without being asked

Triagem binds its keyboard on the **window** (`src/routes/triage/+page.svelte`,
the `onKey` effect): ↑↓ navigate, `Enter` extracts, `Esc` ignores. The handler
exempts only text inputs, textareas and selects.

### C. `Enter` on a focused button also extracts — Verified

- What: a button that has the focus (a pill, "Ignorar", the adapter menu, a
  ticked checkbox…) gets its native Enter-click **and** the window handler
  runs `extractFocused()`. On a "pronta" piece that is a forced re-extraction:
  it spends API and, once it succeeds, **retires the reviewer's corrections**
  on MedLens (by design, decision 017 § 4 — `retire_corrections` in `lib.rs`),
  with no confirmation.
- Proposal: act only when the event target is the page itself (or the rail),
  never a control; and give "Reextrair" a confirmation that names what it
  discards.

### D. The `Esc` that closes the bell or the user menu also ignores the piece — Verified

- What: `NoticeCenter.svelte` (and the Titlebar's menu) close on their own
  window-level `Esc` without stopping the event, and the Triagem handler, which
  only checks its two adapter menus (not the grouping menu), then runs
  `ignoreFocused()`. Closing a popover on `/triage` sends the focused piece to
  Ignorados — reversible there, but silent.
- Proposal: one keyboard dispatcher per window (popovers first, then the
  screen), or `stopPropagation` in the popovers.

### E. OPMENEXO: `Esc` closes the order subtela from inside a field — Verified

- What: `OrderDialog.svelte` closes on any window-level `Escape`, with no
  check of the target — typing in a field or answering an open inline
  confirmation included (`ConfirmButton` does not take `Esc`). The close
  discards the invoice basket, the typed NF numbers, the quotation prices and
  new lines, the chosen laudo and a message draft.
- Proposal: `Esc` first leaves the focused field or cancels the open
  confirmation; closing with unsent work asks once.

## 1. The screen says something that is not true

These break the app's first rule — _the screen never asserts what nobody
verified_ — which is the rule most of its history was paid for.

### 1.1 Faturamento: a failed gesture shows success, or nothing — Verified

- Where: `src/lib/billing.svelte.ts` (`deliver`, `process`, `recover`,
  `resolve`, `withdraw`) and `src/routes/billing/+page.svelte` (the handlers of
  the same names).
- What: every gesture catches its refusal into `#failure`, then its `finally`
  runs `await this.refresh()`, and a successful list read sets
  `#failure = null` — before the page reads it. So a failed **"Entregar ao
  Fantasma"** shows the ok toast "Conta entregue ao Fantasma."; refused
  Processar / Retirar / Recuperar / Assinar show nothing.
- Why it matters: the operator believes a bill is on the robot's list when it
  is not, or clicks again with no idea why nothing happened.
- Proposal: return the gesture's outcome (or error) from the store method and
  toast from that value; keep the list read's failure in its own field.

### 1.2 Dashboard: a healthy run is drawn as "parou no meio da corrida" — Verified

- Where: `unfinished_run` in `src-tauri/src/lib.rs` (`job_id` set, no outcome,
  no failure); `src/routes/dashboard/+page.svelte` (the asks rows and the
  stalled buckets).
- What: during a live run the record matches exactly that shape. The
  Faturamento hides its note with `!billing.isBusy(bill.id)`; the Dashboard
  does not, so it shows a red "parou no meio da corrida — recupere o desfecho"
  row and marks the stage bucket "parada" while the robot works normally.
- Proposal: apply the same busy guard on the Dashboard (the store is shared),
  or have the store expose "stopped mid-run" already excluding the bill in
  flight.

### 1.3 Revisão: an unread colinha is drawn as a clear gate — Verified

- Where: `sheetLine()` in `src/routes/review/[id]/+page.svelte`.
- What: it returns `null` both when the colinha is complete and when the
  colinhas were never read (or the read failed). The dot turns green and
  Aprovar is enabled with nothing checked. The Rust gate at approval still
  refuses, so no bad file is written — but the screen asserted a check.
- Proposal: three states (unknown / complete / missing), Aprovar disabled and
  the gate line saying "colinha ainda não lida" while unknown.

### 1.4 Revisão: corrections can vanish without a sign — Verified

- Where: `review.correct()` in `src/lib/review.svelte.ts`; the commit handler
  in `src/routes/review/[id]/+page.svelte`.
- What: `correct()` returns `false` while another save is in flight, so a
  second edit typed quickly is dropped silently; the editor is closed
  (`editing = null`) before the POST, so a refused save loses what was typed;
  and nothing marks the cell as saving.
- Proposal: keep the draft until MedLens answers, mark the cell pending, and
  queue (or visibly refuse) a second edit.

### 1.5 Configurações: the colinha toast claims a file it may not have written — Verified

- Where: `save()` in `src/lib/components/service-fields/AdaptersSection.svelte`;
  `write_local` in `src-tauri/src/cheatsheet.rs`.
- What: every successful save toasts "Colinha gravada — o arquivo do Fantasma
  foi regenerado.", but `write_local` returns `Ok(())` **without writing** when
  the Fantasma folder is missing or does not look like the project (by
  design, so a colinha can be assembled first). The toast asserts a
  regeneration nobody did — and the robot's file is what invoices.
- Proposal: have the command answer whether the robot's file was written, and
  word the toast from that ("gravada aqui — o Fantasma ainda não está
  configurado").

### 1.6 A failed preference write can toast "[object Object]" — Verified

- Where: `#write()` in `src/lib/preferences.svelte.ts`.
- What: `cause instanceof Error ? cause.message : String(cause)`, while this
  app's errors cross the bridge as `{ kind, message }` objects — so the strip
  and the toast ("A preferência vale nesta sessão, mas não foi gravada:")
  carry `[object Object]`. `ipcMessage()` exists for exactly this.
- Proposal: use `ipcMessage(cause)`.

### 1.7 A saved Relatório can name a question that did not produce it — Verified

- Where: `saveDraft()` in `src/routes/query/+page.svelte` stores
  `source_sql: query.sql` — the editor's text at the moment of saving.
- What: if the operator edits the SQL after the answer arrived (to prepare the
  next question) and then saves, the sealed sheet carries rows from one query
  and the text of another; "Rodar de novo" will later reload the wrong one.
- Proposal: keep the SQL that was sent with the answer in the query store, and
  save that.

### 1.8 Consulta: the save-scope note and the sort arrows say more than is true — Verified

- The save box says "todas as linhas à vista vão para o relatório", but with
  nothing ticked the save takes **every surviving row**, ignoring the filter
  (`keyedRows` in `saveDraft()`).
- Shift + click on a header adds a second TanStack sort key (multi-sort is
  never disabled in `Grid.svelte`): two headers show arrows, while
  `GridView.inShownOrder` applies only `sorting[0]` — and only that one is
  named in the pager and saved.

### 1.9 Other untrue faces — Reported

- **Spook and NFe Pack stores**: a failed poll stores its message and a later
  good poll does not clear it (`refresh()` in `spook.svelte.ts` — Verified;
  `nfepack.svelte.ts` — Reported), so a red strip outlives the problem until
  the next gesture.
- **Sobre**: the autostart switch reuses the service switch, whose "on" is
  the ok-green of "the service is up" — a stored preference wearing the colour
  reserved for "somebody checked".
- **Spook address note** says "Vazio: a conexão não existe.", but with a
  folder set a blank address means this app's own `127.0.0.1:8765`.
- **Wizard, Spook step** — Verified: the step is complete when
  `base_url !== "" === spook_token` (`src/lib/assembly.ts`), so a Spook set up
  **by folder** (blank address on purpose, decision 024 § 5) with a token
  issued blocks "Avançar", and its pill says "em branco". The sidebar's own
  rule is "folder OR address" (024 § 7).
- **Status bar, MedLens block** — Verified (code path): only
  `credential === "rejected"` warns, worded "sem credencial…"; a credential
  that is simply missing falls through to a green "no ar"
  (`src/lib/service-blocks.ts`). The Spook block keeps the two apart.
- **Wizard, step 5** — Reported: its pill is always amber "montagem parcial"
  and its copy says "Você pode entrar assim mesmo", though the step is only
  reachable with every required step complete; steps 2 and 4 say "Ligar…
  chega com a integração" above a working switch.
- **Stale screen text** — Reported: the MailBridgr block still says "doca a
  caminho" / "a Doca vigia quando existir"; the "PIN ativo" tooltip says the
  window locks itself even with auto-lock "nunca".
- **Busca: an NFe Pack log hit opens a view that does not hold it** —
  Verified: the search reads the monitor's **full** log
  (`readNfepackLog(false)` in `src/lib/search.svelte.ts`), and the hit's door is
  `/logs?source=nfepack`, which shows **error lines only**, numbered among
  themselves — an INFO hit lands on a screen without that line.
- **Logs: a refused read draws nothing** — Reported: the reads have no error
  handling (`logs/+page.svelte`, and the log pane of `nfe-pack-monitor`), so a
  refusal is an unhandled rejection repeated by the 2 s follow, with the old
  lines left in place; switching source can briefly show the previous source's
  lines under the new label.
- **NFe Pack console: "never asked" drawn as absence** — Reported: before the
  first status answer it says "nenhum ciclo registrado ainda" and the metrics
  card says it never exported.
- **The bell lags for notices posted in the background** — Reported: notices
  posted from Rust (a boot failure, the monitor's errors, an extraction
  outcome) reach the badge only at the next re-read (launch, unlock, opening
  the panel or the screen).
- **Catálogos: a refused list of catalogs is invisible on a first visit** —
  Verified: both failure paragraphs sit inside `{#if current !== null}`
  (`routes/catalogs/+page.svelte`), and with no answer there is no current
  catalog — a lost wallet key with mirror files present looks like an empty
  panel, the loss dressed as absence.
- **Listas** — Reported: `/shortlists/[id]` draws "Este catálogo ainda não
  foi lido nesta máquina" before the catalog list answers and when it fails;
  a mirror that refuses to open leaves "Abrindo o espelho…" forever;
  Copiar tudo / Exportar CSV stay active with no mirror and write every code as
  "não está no espelho"; the meta line keeps the pre-save count and stamp; on
  `/shortlists` a failed delete's message is wiped by the refresh and a failed
  pin replaces the whole list; an unreadable list file is skipped with no word.
- **A collection another window started ends without an outcome** — Reported
  (`catalogs.svelte.ts`): its failure is never shown.
- **Ignorados can toast success when nothing happened** — Reported: a stale
  restore or delete, and a watched file whose revision changed, return `Ok`
  (`lib.rs`), so "Peça de volta à doca." / "Peça excluída da doca." appear.
- **Relatórios and Consulta** — Reported: the sheet's title block keeps the
  stamp and row count it loaded after Salvar (the returned `SheetMeta` is
  discarded); the Consulta's "célula editável aqui dentro" promises editing
  that does not exist; removing every row shows the filter-miss sentence with
  an empty filter; a failed delete or pin on `/sheets` vanishes or replaces the
  list (the shared `SavedList` behaviour also seen on Listas).
- **Triagem's "Extrair ({n})" counts every tick** and silently skips the
  pieces without an adapter or hidden by the filter — and does nothing, with no
  word, when none qualifies; "Ignorar" and the batch adapter act on ticked
  pieces the filter hides. Reported.
- **OPMENEXO: the busy sentence can print "na ordem null."** — Verified: the
  condition is any portal write in flight, the text interpolates
  `opmenexo.sendingOrderId`, which only an invoice send sets
  (`OrderDialog.svelte`); during a quotation answer, a laudo upload or a sweep
  of another order the sentence names no order.
- **OPMENEXO describes itself as read-only** — Reported (labels quoted in the
  spec): the badge "fatia 1 · só leitura", "nada aqui muda sozinho — só quando
  você pede. Nenhuma ação sai desta tela." and "somente leitura · o portal
  responde e fatura, não esta tela", while the subtela sends invoices, answers
  quotations, uploads the laudo, replies, marks read — and the operator's
  clock re-reads on its own. `question.opmenexo` still starts with "Estudo:".
- **OPMENEXO, smaller** — Reported: the read stamp never ages while the screen
  stays open (`Date.now()` inside a `$derived`, no timer) and counts only
  minutes ("lido há 1440 min"); the card number means non-hidden rows when rows
  came and the Spook's count when they did not, with the tone taken from the
  portal's total (an all-hidden pile shows "0" in full tone); the seal is green
  for any status the app does not know; the "portal já recebeu" panel reads the
  permanent send memory, so an order back in `FA` keeps it forever; the
  documents column stays "Lendo a ordem no portal…" after a failed read.
- **Dashboard first paint**: every launch lands on it, and before the stores
  answer it shows zeros, "Nada exige você agora.", "nunca" and Fantasma "falta
  configurar". The stores have `checking` flags it does not read.
- **Dashboard**: spend header "US$ 0,00" when nothing could be asked; a query
  trail that failed to load reads "nunca"; spend tooltips and duplicate rows
  print ISO dates.
- **Revisão**: "Todas (n)" counts departed bills the view hides; a failed
  open leaves "Abrindo a extração…" forever; a failed list draws "Nada para
  conferir" under the red strip (the store marks the list as answered in
  `finally`).
- **Faturamento**: a list read that fails with no rows shows "Nada em
  andamento".

## 2. Business-rule gaps — the operator decides

### 2.1 Quantity does not hold the gate — Verified

- Where: `queue::evaluate` in `src-tauri/src/queue.rs`.
- What: the gate demands the lot on every item not struck out; quantity
  "rides the model as a number and cannot be absent", so **0 passes**.
  `AGENTS.md` and `docs/telas.md` say "lot + quantity".
- Proposal: decide whether 0 is a valid quantity; if not, add it to both the
  screen gate and `queue::evaluate` (the rule is re-checked where the file is
  written).

### 2.2 Some bills can never be approved — Reported (gate Verified)

- What: the gate requires a code whenever an insurer or doctor text was read;
  those two texts are not editable on the Revisão, and codes come only from
  the mirror. A doctor missing from the ERP registry, or a catalog never
  collected, leaves "Não enviar" as the only way out — against the inherited
  rule "the mirror suggests and never blocks".
- Proposal: an explicit path (correct/clear the read text, or type a code with
  the "nobody checked" warning the Colinha dialog already has).

### 2.3 A "provável" bill cannot be signed — Verified

- Where: `src/routes/billing/+page.svelte` — "Assinar como resolvida" renders
  only when `bill.last_failure !== null`.
- What: an investigation bill has an outcome and no `last_failure`, so the
  sign-off never appears; Processar is gone and only Retirar is offered.
  Decision 023 § 8 describes the sign-off as closing this case.
- Proposal: offer the sign-off on `needs_investigation` too (the human's word
  is exactly what the investigation asks for).

### 2.4 Reported, smaller

- **Processar is not gated on the robot's state or busy lock**; the refusal
  arrives only after the confirmation. The banner reads "Processando agora"
  for any gesture, and a `foreign` robot shows "Robô pronto · Sessão aberta
  no Everest" although that state is decided before the session is checked.
- **Run stamps use the instant the run started** (`wrote_to_erp_at_ms`,
  `checked_at_ms`): a run crossing midnight counts toward the previous day in
  "Faturadas hoje".
- **`approve_bill` does not check whether the bill is already queued**; today
  the screens load the summaries first, so this is fragile rather than broken.
- **The atendimento fallback differs** between the screen
  (`admission_id ?? medical_record_id`, no fallback for an empty string) and
  the gate (`queue::admission_from`, which trims and falls back).
- **English refusal sentences reach the pt-BR screen** (in `lib.rs`; all of
  `spook_settings.rs`), sometimes with a doubled lead: "O cofre recusou: the
  system wallet refused: …". Decision 002 keeps Portuguese out of Rust by
  crossing keys; these refusals cross as sentences.
- **The OPMENEXO clients dialog drops unsaved TOML on `Esc` or a click
  outside**, even while saving (`SpookClientsDialog.svelte`).
- **The 5-minute floor of the portal clock lives only on the screen and in the
  store** (`opmenexo.svelte.ts` treats under 5 as off); `preferences.rs`
  stores whatever arrives. Decision 032 says the floor is enforced at the
  boundary.
- **Colinha code fields**: non-numeric text typed and left without a pick
  becomes empty (`RegistryCodeField.svelte` + `CheatsheetDialog.svelte`),
  against the component's own comment; the dialog does not close on `Esc`.
- **Closing a locked window with the X re-stamps the lock** (`Titlebar.svelte`
  → `lock.lock()`), so "trancada há X min" restarts from the close. Reported.
- **The profile photo cannot be chosen again once another avatar mode is
  picked, nor deleted** (`profile/+page.svelte`). Reported.
- **The lock screen's two buttons swallow failures silently**
  (`LockScreen.svelte`). Reported.
- **A silenced notice kind can become impossible to un-silence**: the toggle
  lives only on that kind's rows, so once they are archived or older than 30
  days nothing reverses it (`notices/+page.svelte`). Reported.
- **`Esc` on `/search` always leaves the screen**, including the `Esc` that
  only blurs the titlebar field or closes the bell or the user menu
  (`search/+page.svelte`). Verified.
- **The `opmenexo_news` notice stores Portuguese text as its detail**
  (`opmenexo.svelte.ts`), against decision 010's "stored as a kind, worded at
  draw time" — a language switch cannot reword it. Reported.
- **Coletar invites a click that will be refused** when the Fantasma is off,
  unconfigured or busy with a query — the store only knows about collections;
  and the "comece pelo SQL" toast fires for **any** failure on an edited
  catalog, "Fantasma ocupado" included (`catalogs.svelte.ts`). Reported.
- **A code hit from the global search always lands on Materiais**, whatever
  its catalog (`search.svelte.ts` + `catalogs/+page.svelte`). Reported.
- **Deleting an ignored piece that had been extracted may drop its bills from
  the Revisão**: "Excluir" forgets the piece's `links.json` entry, and the
  Revisão lists only linked extractions (`lib.rs`). Inferred from the code,
  not traced end to end.
- **A crop that MedLens fails has already emptied the basket**
  (`dock.svelte.ts`), so retrying means marking every region again, on every
  photo. Reported.
- **A re-extraction moves the piece out of "Já extraídos" and the focus to
  another document** for the duration — `telas.md` asks that it stay in view.
  Reported.
- **No unsaved-changes guard anywhere**: a dirty Relatório or Lista, the
  Consulta's draft (removed rows, ticks, filter — page-local, reset on
  remount) and the portal clients dialog are all dropped on navigation without
  a question. Reported.
- **Consulta trail**: refusals raised before the query leaves (busy,
  unconfigured, session closed) write no trail line; a running query and one
  orphaned by a closed app both read "sem resposta"; the stored refusal text is
  never shown. Reported.
- **OPMENEXO, during a sweep**: a message send, an invoice send and a
  quotation "Confirmar o envio" can be clicked and return without a word
  (`opmenexo.svelte.ts`; `busyElsewhere` ignores `bulkSweeping`), or reach the
  Rust lock and surface as "O envio não respondeu…" with an English cause.
  Reported.
- **OPMENEXO clock after a restart**: the snapshot is loaded only by the page,
  so the first automatic tick ignores the kept reading's age and
  `#ringIfGrew` compares against `null` — no news rings (decision 032 § 4 says
  it compares with the snapshot on disk). Reported.
- **The quotation's DE-PARA failure sentence asks to "sair e voltar ao campo"**,
  but the same reference is never asked twice (`row.asked === reference`); in
  add-only mode the sheet's footer total reads "—" while the confirmation shows
  the real total; reopening the subtela during an answer shows "0 linha(s)".
  Reported.
- **The laudo chooser accepts XML** — Verified: the pick command fixes the
  filter to `pdf` + `xml` (`lib.rs`) and is reused for the laudo, labelled
  "Laudo (PDF)".

## 3. Project memory and documentation hygiene

### 3.1 The memory fails its own policy check — Verified

`~/Projects/agents/check.sh ~/Projects/bridgr` (run 2026-09-10): **FAIL** —
`.agents/PROJECT_MEMORY.md` line 8 ("Current state") is **77,609 characters**
(limit 1,200), plus a feature-inventory warning; and **16 warnings** because
decisions 033–040 write `- **Date:**` / `- **Status:**` instead of the
template's `- Date:` / `- Status:`. The policy's reason is the one biting
here: an entry nobody can edit stops being updated. Proposal: split the
"Current state" paragraph into the decisions it already cites (most of it is
their history) and keep one-line pointers; normalize the eight headers.

### 3.2 Temporary files tracked in git — Verified

- `docs/Para verificar antes aprovar.txt` (committed 2026-08-18): a validation
  checklist that ends with the tail of an assistant session ("✻ Crunched for
  58s", "※ recap: …") and carries real OPMENEXO order numbers and real client
  names. The memory policy forbids transcripts and temporary task state;
  the walk it lists was validated on 20/08.
- `docs/plano-2026-08-27.md`: a plan that says it "dies with the last front";
  its Frente E (the environment set up only through the UI) is not marked
  done.
- Proposal: delete the checklist; close or delete the plan.

### 3.3 Stale statements — Verified unless marked

- `README.md`: "A esteira ainda não se move" — the door (022) and the
  Faturamento (023) are built and validated.
- `.agents/PROJECT_MEMORY.md` § Screens: `/dashboard` "an EmptyScreen";
  Settings "seven sections" (nine exist); § Related repositories points at
  `~/Projetos/…` (absent on this machine, which has `~/Projects`) and omits
  Spook and NFePackMonitor.
- `docs/telas.md`: bilingual (the later sections are Portuguese, against the
  repository's own English rule) and overtaken in places — Revisão "Aprovar
  is drawn disabled"; "document on the left" (the code puts the bills left and
  the document right); Faturamento "three gestures" and a "Faturada" stage (5
  gestures and 4 stages today); "gets out of the way when there is a single
  file" (not built); sidebar counters and an idle Fantasma block (no counters;
  the Fantasma block is live). Reported by the screen readings.
- Decisions (Reported): 022 § 5's queue chip/view/pill replaced by 023 § 6
  without a supersession mark; 018 § 3 (colinha outside the gate, Aprovar
  disabled) overtaken; 037's claim that a withdrawn and re-approved bill counts
  twice is false (withdrawal deletes the record); 040 still "awaiting live
  validation"; a `billing.rs` comment says no gesture writes `resolved_at_ms`,
  and `resolve_bill` does.
- Configurações (Reported): the memory still calls decision 027 "proposed"
  with per-portal tabs (the clients editor was built as a dialog) and the Doca
  "one field" (two); `telas.md` § `/settings` still describes Ligar/Desligar
  buttons (a switch since 19/08), clients "edited in place" (a dialog), and a
  prompt editor and corrections panel under Adapters (not built — yet the
  section's hint still promises "prompt de extração"); 024 § 3 says the token
  is read-only (`spook_service.rs` issues read + write); 035 says the "vale no
  próximo início" hint lasts the session (it lasts the visit); 006 § 2 claims
  a unified console vocabulary (see § 4).
- Consulta and Relatórios (Reported): decision 013 § 6 and the memory say 25
  rows per page (the default is 50; `telas.md` is right); `telas.md`, the
  memory and 006 § 2 say row removal needs "no dialog" (014 § 6a overturned it —
  the code asks in place through `ConfirmButton`); the memory says the danger
  tone comes from `:first-of-type` (now an opt-in `danger` class); 013 § 3
  names a "Reaproveitar" label that does not exist; `telas.md` says the sheet
  filter reaches every column (only visible ones); the lint hints live only in
  the Catálogos SQL dialog, not in the Consulta.
- Triagem and Ignorados (Reported): `telas.md` names the piece states
  `sem_cliente/fila/pronto` (the code: `no_client/queued/ready`, worded
  "aguarda extração" and "pronta") and promises a count on Ignorados (none
  exists); decision 019 § 6 still says the Revisão shows the crop as the
  source (the original is the default since 14/08, and 019 was never amended);
  an early memory sentence says crops never force a fresh read (a failed photo
  forces one — corrected further down, the old sentence left in place); the
  comment saying the grouping "vive na sessão" is wrong (it resets per visit).
- Catálogos and Listas (Reported): `telas.md` says only Materiais collects
  (all nine do since 09/08) and that column widths are not persisted (they are:
  decision 026 superseded 015 § 9's deferral, which the memory still repeats);
  the design deck has no Listas screen and a different Catálogos subtitle,
  neither listed in 006; `StatusBar.svelte`'s header still calls the Fantasma
  block an idle placeholder.
- Logs, NFe Pack, Avisos, Busca (Reported): decision 006 § 1 still lists the
  global search and the Serviços block as absent (both built); `telas.md` §
  `/logs` lists four sources (five, with "NFe Pack · Serviço") and places Logs
  under "Fora da esteira" (it is under "Sistema"); `telas.md` says the monitor
  screen "carrega os dois gestos, nada mais" (it also opens notes); decision
  034 and the notice's own pt-BR text send `nfepack_errors` to Logs · NFe Pack,
  while the code routes it to the console; two comments promise the metrics
  will expand on the dashboard (036 put that on the console); `search.svelte.ts`
  says discarded accounts stay out, and its code lists them as "Revisão · não
  enviar" — which is also what the footer's "contas excluídas ficam de fora"
  leaves ambiguous.
- OPMENEXO (Reported): `telas.md` § `/opmenexo` still says "fatia 1, só
  leitura, token de leitura", that each card shows the filter it used (removed
  on 21/08), that the action column exists only on the notes pile (it is on
  all three), that messages cannot be hidden (they can) and that the stamp ages
  (no timer); decision 025 § 9 and a store comment put the documents in the
  left column (the code and `telas.md` put them on the right); 032 § 3 promises
  the next tick in the header (not drawn).
- Shell and wizard (Reported): `telas.md`, the memory and the wizard's own
  comment say four steps (five, with the Spook); `telas.md` and
  `style-manual.md` still describe sidebar counters and a "Serviços" footer
  block (neither exists) and idle MailBridgr/Fantasma status blocks (five
  live blocks); `style-manual.md` gives "Encerrar sessão" a danger hover (the
  code's is neutral); the deck draws "Trancar a janela · Ctrl L", "Ctrl ,",
  "Atalhos do teclado ?" and a full page per study, none of them built nor
  listed in 006 as departures.
- The Sobre text (`settings` in `pt-BR.ts`) says the app "não emite nota e não
  dirige o robô — enfileira e observa", while "Processar conta" drives the
  robot and the OPMENEXO subtela sends invoices to a portal. Wording for the
  operator to confirm.

### 3.4 Comments in Portuguese — Verified (heuristic count)

The repository rule is English comments; a word heuristic finds about **350
Portuguese comment lines**, concentrated in the OPMENEXO slice:
`OrderDialog.svelte` (~104), `opmenexo.svelte.ts` (~72), `ipc.ts` (~52),
`i18n/messages.ts` (~30), `routes/opmenexo/+page.svelte` (~26). Proposal:
translate when those files are next touched, not as a sweep.

## 4. Design system and consistency

- **`--color-text-secondary` is used and never defined** (`ConfirmButton.svelte`
  ×2, `CatalogSqlDialog.svelte`) — Verified; recorded as a latent finding in
  the memory since 19/08 and still open. The redesign's token set is the
  natural moment to define or repoint it.
- **Custom themes do not derive the accent ramp** — Verified: a theme paints
  six aliases (`theme.svelte.ts`), and `--color-accent-200…900` stay Breeze
  values, so a custom accent keeps Breeze-blue sorted headers and numeric
  cells.
- **No bundled typeface** — Verified: `--font-body: Inter, system-ui,
sans-serif` with no font file in the app (CSP allows only local fonts), so
  the face depends on the machine.
- **Two byte formatters disagree** — Verified: `OrderDialog.shownSize` ("kB",
  dot) vs `format.shownBytes` ("KB", comma).
- **One service, three names** — Verified: "NFePackMonitor" (sidebar),
  "NFe Pack" (Configurações, status bar, Logs), "NFe Pack Monitor" (one
  OPMENEXO sentence); and "Opmenexo" (sidebar) vs "OPMENEXO" (Configurações).
- **The service consoles do not share one vocabulary** — Reported, despite
  006 § 2 saying they were unified: the foreign state is "de fora do app" in
  one card and "no ar — subiu fora deste app" in another; a silent Spook is
  "não respondeu" on its card and "sem resposta" in the status bar; reissuing
  is "Emitir outra" (MedLens) and "Reemitir" (Spook); starting is amber on
  MedLens and accent on the Fantasma; only MedLens and Fantasma toast on
  switching.
- **One busy flag drives every gesture of a console** — Reported: issuing a
  credential spins the service switch ("Desligando…"), switching shows
  "Emitindo…" (`MedLensPanel.svelte`, `SpookFields.svelte`).
- **Disabled switches with no reason**, against `ServiceToggle`'s own rule —
  Reported (`MedLensPanel.svelte`, `SpookFields.svelte`, `NfepackFields.svelte`).
- **Dead i18n keys** — Verified with a binary-safe search (`rg -a`, outside
  `src/lib/i18n/`): `welcome.otpConfirmed`, `app.tagline`,
  `billing.notDelivered`, `notices.pageEmpty`, `medlens.spendSub` (the
  dashboard's `spendSub` is a local variable, not the key),
  `spook.toastChecked`, `spook.tokenNote`, `spook.tokenLabel`,
  `spook.serviceVersionBefore`, and the services' `turnOn/turnOff/takeDown`.
  Reported: `SpookStore.check()`, `shortlists.removedAfter` (the
  `removedAfter` in use belongs to `sheets` and `query`), `triage.clearFilter`,
  the OPMENEXO keys `copyOrder`, `copied`, `docsPull`, `docsPulling`,
  `docsOpen`, `colMat.manufacturer`, `colMat.unit`; unused CSS rules (`.tools`
  in `routes/catalogs/+page.svelte`, `.tr` in `routes/opmenexo/+page.svelte`);
  the sweeps' failed/forbidden states, stored and never drawn.
- **Text that bypasses the i18n layer** — Reported: the crop composition
  errors reach a pt-BR toast in English (`crop.svelte.ts`); Rust hard-codes
  "desconhecido" and "(sem assunto)" (`delivery.rs`); a crop refusal lists
  internal piece ids instead of file names (`lib.rs`).
- **Listas speaks Relatórios' nouns** — Verified (labels quoted in the
  Listas spec): "Excluir relatório", "Renomear o relatório", "Voltar aos
  relatórios", "Ordenar relatórios", "Nenhum relatório fixado", through keys
  shared with `SavedList`. And the seal's refusals say "a chave **do** lista"
  (`seal.rs` fills a masculine template with `subject: "lista"` from
  `shortlists.rs`).
- **Small promises the code does not keep** — Reported: the catalogs' filter
  placeholder mentions "referência" (no catalog draws that column); the sheet's
  "Filtrar em todas as colunas…" skips hidden columns by rule; the SQL
  dialog's "Ctrl+Enter" placeholder is bound to nothing; the pager and the
  Listas counts have no thousands separator (Catálogos has); the mirror stamp
  is drawn in two formats (dashboard vs Catálogos).
- **The SQL editor paints strings in `--st-ok` and numbers in `--st-warn`**
  (`SqlEditor.svelte`) — state roles used as decoration, which the style
  manual forbids (the red comment is deliberate: it is an error). Reported.
- **Two confirmation drawings** — the in-place `ConfirmButton` and a
  hand-drawn amber strip (SavedList, the sheet, Ignorados, the Revisão's
  Aprovar) — although `ConfirmButton.svelte` presents itself as the one idiom.
  Reported.
- **One field, two names; one gesture, two names** — Reported: "Cargo" (wizard)
  and "Função" (profile) edit the same `profile.role`, with notes that
  contradict each other; "Preferências" names two different places; "Encerrar
  sessão" and "Trancar agora" do the same thing.
- **The status bar's height is a literal 25 px** (`StatusBar.svelte`), outside
  the chrome tokens density scales (`app.css`), and the toast stack sits over
  it — Reported.
- **Tiled windows keep their rounded corners in CSS**: the page only knows
  "maximized" (`window.svelte.ts`), while GTK squares the corners when tiled
  too (`frame.rs`) — Reported.
- **Built consoles under "Ideias futuras"** — Verified: NFePackMonitor and
  Opmenexo lost their `study` badge but stay under that kicker in `nav.ts`.
- **Hand-built parts where shared ones exist** — Reported: the Revisão's items
  table is a plain `<table>`, not the one `Grid` decision 001 requires; its
  Aprovar confirmation is hand-built, not `ConfirmButton`.
- **Missing singulars** — Reported: "1 itens", "1 arquivos", "1 itens sem
  lote", "1 ocorrências", "+1 mensagens"; **two `ago()` helpers** with different
  words and thresholds (dashboard, billing); **"ontem" means two things** —
  24–48 h ago in the bell's panel, the calendar day on the Avisos screen;
  **one counter, two formats** — the heartbeat line prints "1482" where the
  Ciclos line prints "1.482".
- **The NFe Pack "falhas" tile stays amber for a lifetime count**
  (`NfepackMetrics.svelte`), against 036's "only a current failure warns" —
  Reported.
- **`Titlebar.svelte` hard-codes the "Ctrl K" hint** outside the i18n layer
  (decision 002 allows no literal) — Reported.

## 5. Code structure

- **Very large components** — Verified (`wc -l`): `OrderDialog.svelte` 3,710 ·
  `ipc.ts` 2,631 · `routes/triage` 2,563 · `routes/review/[id]` 2,410 ·
  `routes/dashboard` 1,929 · `routes/notices` 1,617. The redesign will touch
  all of them; splitting along the regions the brief already names (e.g. the
  bill card, the matched field, the gate box) would make the new look cheaper
  to apply.
- **The Dashboard runs a full `dock.refresh()` every 30 s** (spool sweep plus
  the MedLens sync) — Reported.
- **The NFe Pack notes grid re-streams the whole history on every good
  cycle** — Verified: an `$effect` keyed on the heartbeat's cycle count calls
  `loadNotes()` (`routes/nfe-pack-monitor/+page.svelte`), which restarts the
  500-row keyset walk from the top. At the monitor's default pause (10 s) that
  is a full re-read every few seconds of work, and the operator's filtered or
  paged view shrinks to the newest 500 rows and regrows each time. Proposal:
  fetch only rows newer than the top id and prepend them.
- **A tooling trap for whoever reads this code base** — Verified on this
  machine: in the agents' shell, `grep` is a shell function over ugrep, and it
  silently skips `src/routes/dashboard/+page.svelte` (valid UTF-8, no control
  bytes, yet `file` reports it as `data`). Two screen readings tripped on it.
  Any "unused" claim about this repository should be checked with `rg -a` or
  `grep -a`.
