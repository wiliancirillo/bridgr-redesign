# Primeira montagem — `/welcome`

> Where: no sidebar item and no menu entry — the root route `/` opens it while the first assembly was never finished · Status: built · Code: `src/routes/welcome/+page.svelte`, `src/routes/welcome/+page.ts`, `src/routes/+page.ts`, `src/lib/assembly.ts`, `src/lib/door.ts`, `src/lib/preferences.svelte.ts`, `src/lib/components/service-fields/MedLensFields.svelte`, `MedLensPanel.svelte`, `SpookFields.svelte`, `FantasmaFields.svelte`, `FantasmaPanel.svelte`, `FolderField.svelte`, `SecretField.svelte`, `src/lib/ipc.ts` (`readEmailChallenge`, `confirmEmailCode`), `src-tauri/src/handshake.rs`

## The question

Is this station assembled — who signs what passes through here, and are the
services the conveyor cannot move without connected? The screen's own opening
line: "Três peças de pé e a esteira anda."

## Where it sits in the work

The **door**, walked once, before any other screen. On launch the root route
decides: first assembly never finished → `/welcome`; finished → `/dashboard`.
Finishing records the moment and the wizard stops existing — a typed or stale
link to `/welcome` lands on the Dashboard. It writes the **same** fields that
Configurações (MedLens, Spook, Fantasma sections) and the Perfil edit later —
one truth, two doors. It does **not** set up the Doca, the NFePackMonitor or
the catalog mirrors; those live only in Configurações and Catálogos
([10-settings.md](10-settings.md), [08-catalogs.md](08-catalogs.md)).

The shell around it is reduced: titlebar with "· Primeira montagem" and the
window controls only — no search, bell or user menu; no sidebar; no status
bar. Toasts do appear ([../04-shell.md](../04-shell.md)).

## Anatomy

Two columns filling the window under the titlebar.

### 1. The rail (left, 296 px, a ground quieter than the page)

- Title **"Primeira montagem"** and the intro: "Três peças de pé e a esteira
  anda. O que você preencher aqui passa a morar em Configurações e no Perfil —
  é lá que se edita depois."
- **[functional]** The five steps, top to bottom, joined by a thin vertical
  track behind their discs. Each step is a button: a round disc (the step
  number; the step's icon instead of the number on the current step, with an
  accent border and ink), the label, and a one-line summary under it.

  | #   | Label (verbatim) | Summary (verbatim)                | Icon on the current disc |
  | --- | ---------------- | --------------------------------- | ------------------------ |
  | 1   | Você             | quem assina o que passa aqui      | `ph-user-circle`         |
  | 2   | MedLens          | os olhos — lê os anexos           | `ph-eye`                 |
  | 3   | Spook            | Portais — opcional                | `ph-plugs-connected`     |
  | 4   | Fantasma         | as mãos — lança no Everest        | `ph-robot`               |
  | 5   | Tudo pronto?     | o que a esteira já consegue fazer | `ph-flag-checkered`      |

  A step that cannot be reached yet is drawn at 45 % and disabled, with the
  tooltip "Faltam campos nos passos anteriores.". Completed steps carry no mark
  of their own today — the rail shows only "current", "reachable" and
  "unreachable".

- **Rail foot — development builds only**: "Ver o app sem montar nada"
  (`ph-arrow-right`) and under it "O app abre, mas nada será lido nem
  lançado." In an installed (production) build this block does not exist.

### 2. The step panel (right; content column at most 600 px wide; scrolls)

Top to bottom, the same frame on every step:

1. **Kicker row**: "passo 2 de 5" (drawn in uppercase, accent ink) and the
   step's **state pill** (see Data shown).
2. **Title** (25 px) and a **role** paragraph under it.
3. **Notice box** (hairline border, `ph-info`): one paragraph of context.
4. **The step's content** (below).

**Step 1 — Você.** Title "Sua conta nesta máquina". Role: "O nome e o e-mail
ficam junto de cada correção e de cada conta aprovada — é assim que a equipe
sabe quem conferiu o quê." Notice: "Um e-mail de verdade importa por um motivo
só: é o que amarra cada correção a uma pessoa. Nada de senha aqui — a
confirmação é por código."

- Two fields side by side: **"Seu nome"** (note "Aparece do lado de cada
  correção que você fizer.") and **"Cargo"** (note "Só um rótulo — não muda o
  que você pode fazer.").
- **"E-mail de trabalho"**: a monospace input and, beside it, either the button
  **"Confirmar e-mail"** (`ph-paper-plane-tilt`) or — once confirmed — the
  quiet green line **"e-mail confirmado"** (`ph-check-circle`) in its place.
  Note under the row: "Sem senha: a confirmação é por código de uso único."
- **The confirmation panel** (opens under the e-mail row; hairline box):
  instructions with `ph-info` — "Abra o Thunderbird → Preferências do
  MailBridgr → aba Bridgr, escolha este e-mail e clique em Gerar código. O
  código aparece lá e você digita aqui." — then **one** of:
  - waiting: "Aguardando o código ser gerado no Thunderbird…";
  - a code for another address: "O código gerado no Thunderbird é de outro
    e-mail. Escolha lá o mesmo endereço digitado aqui.";
  - expired: "O código expirou — gere outro no Thunderbird.";
  - ready: the field **"Código"** (monospace, uppercase, placeholder
    `ABCD-EFGH`, up to 9 characters) and the button **"Confirmar"**.

  Under it, the verdict line of the last attempt (danger ink), and a quiet
  **"Cancelar"** link that closes the panel.

**Step 2 — MedLens.** Title "MedLens — os olhos da esteira". Role: "Ele lê os
anexos e devolve as contas. Sem o MedLens, a Triagem e a Revisão não têm o que
mostrar." Notice: "Preencha a chave e os modelos: ficam gravados e prontos.
Ligar o serviço a partir daqui chega com a integração."

- **"Pasta do projeto MedLens"** — a folder field: monospace input (placeholder
  `~/Projects/medlens`) + **"Escolher"** (`ph-folder-open`, the desktop's own
  folder chooser); note "De onde o app sobe o serviço (uv run uvicorn). É a
  pasta com o pyproject.toml do MedLens."
- **"Chave da OpenRouter"** — a secret field (placeholder `sk-or-v1-…` while
  nothing is stored; the masked `••••••••••` and the badge **"gravada"** once
  stored); note "Fica só nesta máquina, no cofre do sistema. Depois de gravada,
  campo em branco significa não mexi — nunca apagar."
- **"Modelo primário"** and **"Modelo secundário"** side by side, monospace,
  placeholders `anthropic/claude-3.7-sonnet` and `google/gemini-2.5-flash`;
  while the MedLens is up with its credential accepted, typing suggests model
  ids from the AI provider's catalogue — any id is still accepted.
- Under the fields, the **MedLens panel**: a service card ("Serviço · {state}",
  version, "uvicorn em 127.0.0.1:8000 — sobe com o app, desce com ele", an
  on/off switch that shows its own spinner while rising) and a credential card
  ("Credencial" + its state line, button "Emitir" / "Emitir outra"). Every
  state of both cards: [10-settings.md](10-settings.md).

**Step 3 — Spook** (the one step that may stay blank). Title "O Spook, para os
portais". Role: "Cotações e notas fora do Everest". Notice: "Pode seguir em
branco: sem Spook o app monta igual, e as telas de portal ficam desligadas até
você configurar. Se preencher, preencha os dois campos — endereço sem token
responde 401 em tudo."

- The Spook's service card ("Serviço · {state}", "O Spook responde onde este
  endereço apontar.", switch) and credential card ("Credencial", button
  "Emitir" / "Reemitir") — the credential is **issued** by that button, never
  typed.
- Fields: **"Pasta do projeto Spook"** (folder field; note "De onde o app sobe
  o serviço (uv run spook serve). É a pasta com o pyproject.toml e
  src/spook."), **"Endereço"** (placeholder `http://127.0.0.1:8765`; note
  "Onde o Spook atende, por exemplo http://127.0.0.1:8765. Vazio: a conexão não
  existe."), and side by side **"Usuário do portal (OPMENEXO)"** and **"Senha do
  portal"** (secret) with the shared note "O login do portal que o Spook usa — o
  que antes ia à mão no .env dele. A senha fica no cofre do sistema e entra no
  serviço quando o app o sobe; em branco significa que você não mexeu."
- The portal's clients editor is not here (Configurações only).

**Step 4 — Fantasma.** Title "Fantasma — as mãos no Everest". Role: "Ele lança
as contas aprovadas, de dentro de um container, na máquina do ERP. Sem ele, a
conta chega até a fila e para." Notice: "Preencha as pastas e as credenciais —
é delas que o .env vai nascer. Ligar a sessão a partir daqui chega com a
integração."

- **"Pasta do projeto"** (placeholder `~/projetos/fantasma`; note "É aqui que
  mora o .env que o docker compose lê.") and **"Pasta-fila"** (placeholder
  `~/Bridgr/fila`; note "Aprovar grava o JSON da conta aqui."), both folder
  fields.
- **"Usuário do RDP"** and **"Usuário do Everest"** side by side; then
  **"Credencial do RDP"** and **"Credencial do Everest"** (secret fields) side
  by side, with the shared note "Em branco significa não mexi — nunca apague."
- Under the fields, the **Fantasma panel**: "Sessão · {state}", version, the
  state's sentence and a switch; switching it off opens an inline confirmation
  ("Deixar de pé" / "Derrubar agora") because a take-down costs a two-minute
  release. Every state: [10-settings.md](10-settings.md).

**Step 5 — Tudo pronto?** Title "A esteira anda até onde der". Role: "Você pode
entrar assim mesmo: o app não esconde o que falta — cada tela diz onde a conta
vai parar." Notice: "Nada aqui é gravado por engano: o que ficou em branco
continua em branco, e o app avisa quando a conta esbarrar na peça que falta."

- A summary: one bordered row per step 1–4 — the step's icon, label, summary
  and its state pill.
- A closing note (`ph-gear-six`): "Daqui em diante, estes campos se editam em
  Configurações → MedLens e → Fantasma, e os seus dados em Perfil. Esta tela
  não volta: feita a montagem, o app abre direto no trabalho."

A wallet refusal under the MedLens or Fantasma fields shows a danger line "O
cofre recusou: {motivo}".

### 3. The footer (bottom of the panel, always visible)

"Voltar" (`ph-arrow-left`; disabled on step 1) · flexible space · the counter
"passo 2 de 5" · **"Avançar"** (`ph-arrow-right`) — or, on step 5, **"Ir
para o app"** (`ph-arrow-right`).

## Data shown

**State pills — stored facts only** (config and the wallet's yes/no), never a
live service state:

| Step         | Condition                                         | Pill (verbatim)         | Tone |
| ------------ | ------------------------------------------------- | ----------------------- | ---- |
| Você         | the e-mail was confirmed by code                  | "e-mail confirmado"     | ok   |
|              | name and e-mail both blank                        | "sem conta"             | idle |
|              | otherwise                                         | "e-mail não confirmado" | idle |
| MedLens      | the wallet holds the OpenRouter key               | "chave gravada"         | ok   |
|              | otherwise                                         | "sem chave"             | idle |
| Spook        | address blank                                     | "em branco"             | idle |
|              | address filled and a Spook credential stored      | "endereço e token"      | ok   |
|              | address filled, no credential                     | "falta o token"         | warn |
| Fantasma     | every Fantasma field and both credentials present | "campos completos"      | ok   |
|              | otherwise                                         | "faltam campos"         | idle |
| Tudo pronto? | always                                            | "montagem parcial"      | warn |

**What each step requires** (one function answers every gate):

| Step     | Complete when                                                                                   |
| -------- | ----------------------------------------------------------------------------------------------- |
| Você     | name, role and e-mail are filled — the **confirmation is not required**                         |
| MedLens  | folder filled, OpenRouter key stored in the wallet, primary and secondary model filled          |
| Spook    | the address and a stored Spook credential are **both present or both absent** — half is refused |
| Fantasma | project folder, queue folder, RDP user, Everest user filled; RDP and Everest credentials stored |
| Finish   | the four above together                                                                         |

While the wallet has not answered, every step that needs it counts as
incomplete — never assumed complete.

**Counter**: "passo n de 5", in the kicker and in the footer.

## States

| State                      | Trigger                                       | What is drawn                                                                                   |
| -------------------------- | --------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| First open                 | nothing stored                                | step 1, pill "sem conta", "Avançar" disabled, steps 2–5 disabled in the rail                    |
| Step incomplete            | a required fact missing                       | "Avançar" disabled, tooltip "Preencha os campos deste passo para avançar."                      |
| Step unreachable           | an earlier step incomplete                    | rail step at 45 %, tooltip "Faltam campos nos passos anteriores."                               |
| Reopened on a later launch | assembly not finished                         | starts at step 1; stored fields are already filled; the rail lets the operator jump ahead       |
| E-mail blank               | —                                             | "Confirmar e-mail" disabled                                                                     |
| Code panel: waiting        | no code generated yet                         | "Aguardando o código ser gerado no Thunderbird…" (re-checked every 2 s while the panel is open) |
| Code panel: other address  | the code was generated for a different e-mail | the other-e-mail sentence; no field                                                             |
| Code panel: expired        | the code's 5 minutes passed                   | "O código expirou — gere outro no Thunderbird."                                                 |
| Code panel: ready          | a valid code is pending                       | "Código" field + "Confirmar" (disabled while blank or while checking)                           |
| Wrong code                 | attempt failed                                | "Código errado — restam 3 tentativas." (the number is the attempts left); the typed code stays  |
| Attempts exhausted         | fifth failure                                 | "Tentativas esgotadas. Gere um código novo no Thunderbird."                                     |
| No pending code            | confirmed with nothing pending                | "Nenhum código pendente — gere um no Thunderbird."                                              |
| Could not check            | the app could not read its own data           | the system's message, verbatim                                                                  |
| Confirmed                  | right code                                    | panel closes; the button becomes "e-mail confirmado"; pill turns green                          |
| Wallet refused             | the system wallet refused a secret            | "O cofre recusou: {motivo}"                                                                     |
| Service cards              | the live services                             | as in [10-settings.md](10-settings.md) (starting, up, foreign, stopped, refused credential…)    |
| Last step                  | steps 1–4 complete                            | summary rows; pill "montagem parcial" (amber) in every case; "Ir para o app" enabled            |
| Development build          | —                                             | the skip block in the rail foot                                                                 |
| Already assembled          | finished before                               | never drawn — `/welcome` redirects to the Dashboard                                             |
| Unreadable configuration   | the settings file cannot be read              | treated as a first run: the wizard opens                                                        |
| Narrow window (1024 px)    | —                                             | same layout: the rail keeps 296 px, field pairs stay side by side                               |

## Actions

| Gesture                          | Where                  | Precondition / disabled reason                                       | Effect                                                                                           | Confirmation         | Reversible                                                                          | Feedback                                                    | Shortcut           |
| -------------------------------- | ---------------------- | -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ | -------------------- | ----------------------------------------------------------------------------------- | ----------------------------------------------------------- | ------------------ |
| Type in a field                  | any step               | —                                                                    | saved when the field is left (or Enter), trimmed; no Save button                                 | —                    | edit again (here, or later in Configurações/Perfil)                                 | the pill and gates update                                   | —                  |
| Change the e-mail                | Você                   | —                                                                    | saves it and **clears** its confirmation                                                         | —                    | confirm again                                                                       | pill back to "e-mail não confirmado"                        | —                  |
| Confirmar e-mail                 | Você                   | e-mail filled; panel not already open                                | opens the code panel and starts watching for a code                                              | —                    | Cancelar                                                                            | panel states                                                | —                  |
| Confirmar (code)                 | code panel             | a code typed; not already checking                                   | verified by the app; on success stamps the confirmation                                          | —                    | —                                                                                   | verdict line, or the panel closes                           | Enter in the field |
| Cancelar                         | code panel             | —                                                                    | closes the panel, stops watching                                                                 | —                    | reopen                                                                              | —                                                           | —                  |
| Type a secret                    | secret fields          | a blank field never writes                                           | stored in the system wallet; the field empties itself                                            | —                    | type a new one                                                                      | badge "gravada"; for the key, toasts about where it reached | —                  |
| Escolher                         | folder fields          | —                                                                    | the desktop's folder chooser, titled with the field's label; cancelling keeps the value          | —                    | —                                                                                   | danger toast with the system's message if it fails          | —                  |
| Service switch, Emitir, Derrubar | service panels         | see [10-settings.md](10-settings.md)                                 | raise/stop the service, issue a credential                                                       | Derrubar asks inline | —                                                                                   | toasts and card states                                      | —                  |
| Step in the rail                 | rail                   | reachable only when every earlier step is complete                   | jumps to that step                                                                               | —                    | —                                                                                   | —                                                           | —                  |
| Voltar / Avançar                 | footer                 | Voltar off on step 1; Avançar only when the current step is complete | previous / next step                                                                             | —                    | —                                                                                   | —                                                           | —                  |
| Ir para o app                    | footer, step 5         | the whole assembly complete (re-checked at the click)                | records the finish, journals it, opens the Dashboard; from now on the app opens on the Dashboard | none                 | **no** — the wizard never returns; fields stay editable in Configurações and Perfil | the Dashboard                                               | —                  |
| Ver o app sem montar nada        | rail foot (dev builds) | —                                                                    | opens the Dashboard for **this session only**; the next launch knocks again                      | —                    | —                                                                                   | —                                                           | —                  |

## Business rules

1. **A door, not a destination.** It opens only while the assembly was never
   finished; no menu links to it; a finished assembly redirects it away. A
   permanent menu item for a one-time act teaches people to look for work
   where there is none (Bridgr Pro decision 006 § 2).
2. **No Catálogos step.** Collecting a mirror is a manual gesture, one at a
   time, that holds the robot's lock for minutes; it belongs on Catálogos, not
   on a screen walked once (decision 006 § 2).
3. **Every field is required — except the Spook, which is blank or whole.**
   MedLens and Fantasma are the conveyor; the Spook answers for portals the
   conveyor never touches, and demanding it would block every assembly until a
   fourth service exists. Half filled is refused: an address without a
   credential answers 401 on every call and reads as a broken service
   (`assembly.ts`; decision 024 § 6).
4. **The gate is one function** — Avançar, the rail and the finish ask the
   same place, so they cannot disagree, and the finish re-checks at the click.
5. **The e-mail is confirmed by a code born in Thunderbird**, not by a message:
   the MailBridgr extension generates an 8-character code (shown as
   `ABCD-EFGH`) valid for **5 minutes**, **5 attempts**, single use; only a
   hash of it is written where the app can read it; the app verifies it (case
   does not matter, and the look-alikes O, I and L are read as 0, 1, 1). It
   proves the address is an identity configured in this machine's Thunderbird
   and that the extension → app channel works — **not** mailbox ownership. So
   the screen says **"confirmado"**, never "verificado" (decision 007).
6. **Green only from a stored fact somebody asked for.** The confirmation stamp
   is written only by the verification and zeroed by any e-mail edit; pills
   read config and the wallet's yes/no, never a service's live state — the
   live states belong to the service cards, which ask their service.
7. **Secrets are write-only**: they go to the system wallet, never come back to
   the screen; blank means "I did not touch it" ([../02-invariants.md](../02-invariants.md) § 12).
8. **The connections are born here.** The MedLens, Spook and Fantasma steps
   carry the same service cards as Configurações: filling and switching on here
   is the whole rise. Whether a service rises again at the next launch is the
   Sobre list's switch in Configurações — on by default for MedLens and Spook,
   **off** for the Fantasma; closing the app never takes the robot down
   (decisions 011, 012, 035; [10-settings.md](10-settings.md)).
9. **The confirmation is not required to finish** — the fields are.
10. **The skip exists only in development builds**, and only for the session:
    a production station without its pieces is exactly what the wizard exists
    to prevent.

## Must survive the redesign

- The door semantics: first launch only, no menu entry, gone after the finish.
- Every gate, with its reason said where the control is disabled; the finish
  re-checks.
- The optional-but-never-half Spook step.
- Pills derived from stored facts only; green only for a confirmed e-mail, a
  stored key, a whole Spook connection, complete Fantasma fields.
- The word **"confirmado"**; the code panel's distinct states — waiting,
  another address, expired, wrong with attempts left, exhausted, nothing
  pending.
- Secrets masked, blank = untouched; the service cards' multi-state switches and
  the inline confirmation before a take-down.
- A visible way to know which step is current, which are reachable and which
  are not.
- No skip in production.

## Free to redesign

- Rail versus a top stepper; the discs, the track, the pills' look.
- The per-step layout, the notice boxes, the summary's form.
- Where the footer buttons sit, as long as Voltar/Avançar/Ir para o app and the
  step counter stay together.

## Sample content for mockups

- Você: "Camila Duarte Rocha" · "Faturista OPME" · `camila.rocha@empresa.example`;
  pill "e-mail não confirmado"; code panel ready with `7KQM-2XRD` typed; after a
  wrong try, "Código errado — restam 4 tentativas.".
- MedLens: folder `~/Projetos/medlens`; key "gravada"; models
  `anthropic/claude-3.7-sonnet` / `google/gemini-2.5-flash`; service card "no
  ar"; pill "chave gravada".
- Spook: left blank — pill "em branco"; or folder `~/Projetos/spook`, address
  `http://127.0.0.1:8765`, portal user `camila.rocha`, credential issued — pill
  "endereço e token".
- Fantasma: `~/Projetos/fantasma`, queue `/srv/fila-fantasma`, users
  `camila.rocha` / `CROCHA`, both credentials "gravada"; pill "campos
  completos".
- Folders and cast: [../06-sample-data.md](../06-sample-data.md).

## Design opportunities

1. **Mark finished steps in the rail.** Today a completed step looks exactly
   like an untouched reachable one. The same stored facts that gate the
   buttons could draw a check per step — never a live service state.
2. **Let the last pill say the truth.** Step 5 always wears the amber
   "montagem parcial", although it can only be reached when every required step
   is complete; amber is reserved for "a human must decide here". A pill that
   distinguishes "complete" from "complete, Spook left blank" would fit the
   rule. Wording is the operator's.
3. **Reword the stale notices (proposal).** Steps 2 and 4 say switching the
   service on "chega com a integração", while the switch is right below them;
   step 5 says "Você pode entrar assim mesmo", while the finish is gated.
4. **Show the code's time left.** The app knows the pending code's expiry; the
   waiting/ready states could count it down instead of only saying "expirou"
   afterwards.
5. **One word for the role.** The wizard calls the field "Cargo" ("Só um
   rótulo — não muda o que você pode fazer."); the Perfil calls the same field
   "Função" ("quem aprova conta neste app"). Proposal: one label and one note.
6. **Make the Spook's pair visible as a pair.** The notice asks for "os dois
   campos", but only the address is a field — the other half is the credential
   issued by the card's button. Drawing address and credential as the two halves
   would match the rule the gate enforces.

## Open questions

- Should the wizard ever learn the Doca or the NFePackMonitor, or do they stay
  Configurações-only?
- What should step 5 say for a station assembled without the Spook — is that
  "pronto" or still "parcial"?
