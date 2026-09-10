# Perfil — `/profile`, and the locked window

> Where: Perfil — titlebar user menu › "Perfil" (also the status bar's "PIN ativo", the two PIN notices, and the lock screen's "Bloqueio com PIN"); the locked window — replaces the whole body whenever the window is locked · Status: built · Code: `src/routes/profile/+page.svelte`, `src/lib/components/PinCard.svelte`, `AvatarPhotoEditor.svelte`, `Avatar.svelte`, `LockScreen.svelte`, `src/lib/avatar.ts`, `src/lib/avatar-photo.svelte.ts`, `src/lib/lock.svelte.ts`, `src/lib/preferences.svelte.ts`, `src/lib/format.ts`, `src/routes/+layout.svelte` (auto-lock), `src-tauri/src/pin.rs`, `src-tauri/src/avatar.rs`

## The question

- **Perfil**: who is at this bench, and how does the app show them? (Its
  section kicker: "Quem assina o que passa por aqui".)
- **The locked window**: who may come back to this bench — and how, when the
  PIN is forgotten?

## Where it sits in the work

The Perfil is off the conveyor: a person is not a station. It holds the
identity that goes beside every correction and approved bill, the face the
chrome draws, two person-shaped preferences (sidebar mode, language), facts
about this station, and the **PIN** that locks the window. The locked window
can appear over any screen: by the auto-lock, by "Encerrar sessão" in the user
menu, by "Trancar agora" on the PIN card, or on launch when the window was left
locked. Everything behind it stays open and untouched.

## Anatomy

### Perfil

1. **Header** (a bottom hairline under it): the face (58 px); the name as the
   page title (23 px) — "Sem nome" in quiet ink when blank; a subtitle "role ·
   e-mail" with whichever of the two exist (absent when both are blank); at the
   right, a toggle button **"Editar dados"** (`ph-pencil-simple`) which reads
   **"Concluir edição"** (`ph-check`, accent outline) while editing.
2. **[functional]** Two columns: a flexible main column and a 320 px aside. At
   1180 px of window width or less they stack (aside under main).
3. **Main › "Quem assina o que passa por aqui"** — an uppercase kicker with a
   fading hairline, then a 2 × 2 grid of tiles separated by hairlines. Each tile:
   small uppercase label, the value (or "—" in quiet ink), sometimes a note.

   | Tile                     | Value style | Note under it (verbatim)                                                          |
   | ------------------------ | ----------- | --------------------------------------------------------------------------------- |
   | Nome                     | text        | —                                                                                 |
   | Função                   | text        | "quem aprova conta neste app"                                                     |
   | E-mail                   | monospace   | only when an e-mail exists: "não confirmado" or "confirmado em 05/09/2026 · 14:32" |
   | Assinatura no informe    | monospace   | "vai no rodapé do que você exporta"                                               |

   In edit mode each value becomes an input in place (e-mail and signature in
   monospace).
4. **Main › "Foto e avatar"** — kicker, then a card; **the whole card accepts a
   dropped image**. Left: the chosen face in a 72 px circle that is itself a
   button (tooltip "Clique para desenhar outra variação" when a drawn style is
   chosen, "Clique para escolher uma foto" otherwise), captioned "arraste um
   arquivo". Right: the hint "Sua foto fica só nesta máquina. Se preferir não
   usar foto, escolha um avatar desenhado — ou fique nas iniciais."; a row of
   seven choices — **"Iniciais"** (the initials face) and six drawn styles, each
   previewed at 30 px from the current seed: **"Traço"**, **"Retrato"**,
   **"Polegar"**, **"Robô"**, **"Formas"**, **"Grade"** — the chosen one
   outlined in the accent. Under the row, only when a name exists: "Os avatares
   vêm do DiceBear, desenhados nesta máquina com a semente `Camila Duarte
   Rocha#3` — mesma pessoa, mesmo desenho em qualquer máquina." There is no
   "photo" choice in the row: the photo mode is entered by picking or dropping a
   picture. A failed save shows a danger line "A foto não foi gravada: {motivo}".
5. **The framing dialog** (modal over the whole window, dimmed backdrop):
   title **"Enquadrar a foto"**; a 240 px circular window onto the picture —
   drag to position it; a **"Zoom"** slider (1× to 3×); **"Cancelar"** and
   **"Usar foto"** (disabled until the picture has loaded). The picture can never
   leave a gap inside the circle; what the circle shows is exactly what is
   saved.
6. **Aside › "Bloqueio com PIN"** card (below).
7. **Aside › "Preferências"** card:
   - a row linking to Aparência: `ph-palette`, "Aparência", the palette in
     force underneath (e.g. "Breeze Dark"), a caret → Configurações › Aparência;
   - **"Barra lateral"**: a segmented control "Automática" · "Sempre aberta" ·
     "Sempre recolhida";
   - **"Idioma"**: a segmented control "Português (Brasil)" · "English" —
     each language written in itself, never translated.
8. **Aside › "Esta estação"** card: three label/value lines, values in
   monospace — "Máquina" (hostname), "Versão do app", "Aberto desde" (when this
   run started, `dd/mm/aaaa · hh:mm`). Unknown values are "—".

**The "Bloqueio com PIN" card** — title, subtitle "Tranca a janela sem fechar
o que está aberto. Não substitui a senha da máquina — evita que outra pessoa
mexa na esteira.", and a switch at the right (tooltip "Ligar/Desligar o
bloqueio com PIN"). Three bodies:

- **Off** (no PIN stored, no form open): nothing below the header; the switch
  is off.
- **Form** (switch just turned on, or "Trocar PIN"): the switch shows on while
  the define form is open. Title "Escolha um PIN de 4 dígitos" or "Trocar o
  PIN"; when changing, a first field **"PIN atual"**; then **"Novo PIN"** and
  **"Repita"** side by side — masked, numeric keyboard hint, 4 characters,
  placeholder `••••`; an error line; **"Gravar PIN"** (`ph-check`) and
  **"Cancelar"**; a note: "4 dígitos, só nesta máquina. Esquecer o PIN não perde
  trabalho: um código gerado no MailBridgr destranca a janela e desliga o PIN."
- **Active** (a PIN is stored, no form open): a line with a filled lock icon in
  green — "PIN de 4 dígitos ativo · gravado em 05/09/2026 · 14:32" (the date
  part only when known); an uppercase label "Trancar sozinho depois de" over
  four chips **"1 min"**, **"5 min"**, **"15 min"**, **"nunca"** (the current one
  in the accent); buttons **"Trocar PIN"** (`ph-pencil-simple`) and **"Trancar
  agora"** (`ph-lock-simple`).

### The locked window

The titlebar stays — mark, name, the context of the screen behind the lock,
the bell and the user button drawn but disabled, no search field, window
controls working ([../04-shell.md](../04-shell.md)). Everything else is
replaced by one centred column:

1. A pill-shaped badge with a filled lock: **"Janela trancada"** (uppercase).
2. The face (66 px), the name (18 px; "Sem nome" when blank), and under it
   **"trancada há 6 min"** (whole minutes since the lock, refreshed every
   30 s; "trancada há 0 min" right after locking).
3. One of three panels:
   - **The PIN pad** — four dots (filled in the accent as digits are typed);
     an error line when there is one; a 3 × 4 keypad (keys about 58 × 44 px):
     `1 2 3 / 4 5 6 / 7 8 9 / ⌫ 0 ✓` — "⌫" is named "Apagar", "✓" is named
     "Destrancar" and is accent-tinted; under it the link **"Esqueci o PIN —
     destrancar por código"**.
   - **The code panel** (after that link): instructions with `ph-info` — "Abra o
     Thunderbird → Preferências do MailBridgr → aba Bridgr, escolha este e-mail
     e o propósito Recuperar PIN, e clique em Gerar código. O código destranca a
     janela e desliga o PIN — você grava um novo depois." — then the same states
     as the wizard's e-mail confirmation (waiting / another address / expired /
     ready with the field `ABCD-EFGH` and **"Confirmar"**), a verdict line, and
     the link **"Voltar para o PIN"**.
   - **No PIN stored** (defensive): "O bloqueio com PIN está desligado nesta
     máquina, então não há nada para destrancar. Ligue o PIN no Perfil se quiser
     que a janela se tranque sozinha." with **"Bloqueio com PIN"**
     (`ph-lock-key`, primary) and **"Voltar ao app"**.

## Data shown

### Perfil

| Field / fact                 | Meaning                                                      | Presentation rule                                                     | Example                                   |
| ---------------------------- | ------------------------------------------------------------ | --------------------------------------------------------------------- | ----------------------------------------- |
| Nome                         | who signs; self-declared, authorizes nothing                 | "—" when blank; header shows "Sem nome"                               | Camila Duarte Rocha                       |
| Função                       | a free-text role                                             | "—" when blank                                                        | Faturista OPME                            |
| E-mail                       | the address confirmed through MailBridgr                     | monospace; confirmation note only when an e-mail exists               | camila.rocha@empresa.example              |
| Confirmation                 | the stamp written by the code verification                   | "não confirmado" / "confirmado em dd/mm/aaaa · hh:mm"                 | confirmado em 05/09/2026 · 14:32          |
| Assinatura no informe        | the string printed in the footer of exports                  | monospace                                                             | C. D. ROCHA · FATURAMENTO OPME            |
| Face                         | photo › drawn style › initials › person glyph                | same face in titlebar, menu, header, lock                             | CR                                        |
| Seed                         | what the drawn styles are generated from                     | the name, or the pinned variation `name#N`                            | Camila Duarte Rocha#3                     |
| Aparência value              | the palette painting the base in force                       | the built-in Breeze of that base when none chosen                     | Breeze Dark                               |
| Máquina                      | this computer's name                                         | "—" when unknown                                                      | estacao-faturamento-01                    |
| Versão do app                | this build                                                   | "—" when unknown                                                      | 0.1.0                                     |
| Aberto desde                 | when this run of the app started                             | `dd/mm/aaaa · hh:mm`, "—" when unknown                                | 10/09/2026 · 08:02                        |
| PIN active line              | a PIN is stored (the wallet's answer) + when it was saved    | date part omitted when unknown                                        | PIN de 4 dígitos ativo · gravado em 05/09/2026 · 14:32 |
| Auto-lock                    | idle minutes before the window locks itself                  | 1 / 5 / 15 / nunca; default 5                                         | 5 min                                     |

### The locked window

| Element             | Meaning                                  | Rule                                        | Example              |
| ------------------- | ---------------------------------------- | ------------------------------------------- | -------------------- |
| Name and face       | who left the bench                       | as above                                    | Camila Duarte Rocha  |
| "trancada há N min" | whole minutes since the lock             | floor; re-read every 30 s; survives restart | trancada há 6 min    |
| Dots                | digits typed so far                      | 0–4                                         | ●●○○                 |

## States

### Perfil

| State                          | Trigger                                   | What is drawn                                                                                       |
| ------------------------------ | ----------------------------------------- | --------------------------------------------------------------------------------------------------- |
| No identity yet                | profile blank                             | "Sem nome", no subtitle, tiles "—", initials face = person glyph, no seed note                     |
| Viewing / editing              | "Editar dados" / "Concluir edição"        | values vs inputs in place                                                                           |
| E-mail unconfirmed / confirmed | the code confirmation                     | tile note "não confirmado" / "confirmado em …"; no note without an e-mail                           |
| Initials / drawn / photo       | the avatar choice                         | the chosen button outlined (none outlined while the photo is in use)                               |
| Framing                        | an image picked or dropped                | the modal; "Usar foto" disabled until loaded                                                        |
| Photo not saved                | the app refused or the disk failed        | "A foto não foi gravada: {motivo}"                                                                  |
| Non-image dropped              | —                                         | nothing happens                                                                                     |
| Station facts unknown          | the answer failed                         | "—"                                                                                                 |
| PIN off / form / active        | the wallet's answer and the switch        | the three card bodies above; until the wallet answers the card shows off                           |
| PIN form errors                | save refused                              | "Digite o PIN atual." · "O PIN tem 4 dígitos." · "Os dois campos não batem." · "O PIN atual não confere." · or the system's message |
| Preference not written         | disk refused                              | the change stays for the session; danger toast (shell)                                             |
| Narrow window                  | ≤ 1180 px                                 | one column; aside under main                                                                        |

### The locked window

| State                   | Trigger                                   | What is drawn                                                                    |
| ----------------------- | ----------------------------------------- | -------------------------------------------------------------------------------- |
| Pad, idle               | locked                                    | four empty dots                                                                  |
| Typing                  | digits                                    | dots fill; a fifth digit is ignored                                              |
| Too short               | ✓ / Enter with fewer than 4               | "O PIN tem 4 dígitos."                                                           |
| Checking                | ✓ / Enter                                 | "✓" disabled until the answer                                                    |
| Wrong PIN               | mismatch                                  | "PIN errado." and the dots empty; no limit on tries                              |
| Could not check         | the app failed to answer                  | the system's message                                                             |
| Code panel states       | "Esqueci o PIN…"                          | waiting, another address, expired, ready, and the verdicts of the wizard's code panel ([15-welcome.md](15-welcome.md)) |
| No PIN stored           | locked while the wallet holds no PIN      | the defensive panel; also drawn until the wallet has answered                    |
| After a restart         | the lock is kept on disk                  | still locked; minutes count from the stamp on disk — the moment of locking, or the moment the X closed the locked window (the X locks again before closing) |

## Actions

### Perfil

| Gesture                       | Where                     | Precondition / disabled reason                         | Effect                                                                               | Confirmation | Reversible                        | Feedback                        | Shortcut |
| ----------------------------- | ------------------------- | ------------------------------------------------------ | ------------------------------------------------------------------------------------ | ------------ | --------------------------------- | ------------------------------- | -------- |
| Editar dados / Concluir edição | header                   | —                                                      | switches the tiles between values and inputs                                         | —            | yes                               | button state                    | —        |
| Edit a tile                   | tiles (edit mode)         | —                                                      | saved when the field is left (or Enter), trimmed; the chrome updates at once          | —            | edit again                        | —                               | —        |
| Change the e-mail             | E-mail tile               | —                                                      | saves and **clears** the confirmation                                                | —            | confirm again (only the wizard confirms today) | note "não confirmado" | —        |
| Pick a style                  | style row                 | —                                                      | "Iniciais" or a drawn style, saved at once                                           | —            | yes                               | the face changes everywhere     | —        |
| Click the big circle          | avatar card               | —                                                      | drawn style chosen: the next variation (`name#2`, `name#3`, …) is pinned; otherwise opens the desktop's picture chooser ("Escolher foto", filter "Imagens": png, jpg, jpeg, webp) | — | clicking walks on; never back | the face changes          | —        |
| Drop a picture                | anywhere on the avatar card | an image file                                        | opens the framing dialog                                                             | —            | Cancelar                          | —                               | —        |
| Frame                         | framing dialog            | —                                                      | drag to place; zoom 1×–3×                                                            | —            | —                                 | live preview                    | arrows move 4 px (Shift: 20 px) |
| Usar foto                     | framing dialog            | picture loaded                                         | saves a 256 × 256 picture on this machine (one file, overwritten), switches to photo | —            | choose another mode               | toast "Foto salva."             | —        |
| Cancelar                      | framing dialog            | —                                                      | closes without saving                                                                | —            | —                                 | —                               | Esc; a click on the backdrop |
| Aparência                     | Preferências card         | —                                                      | opens Configurações › Aparência                                                      | —            | —                                 | —                               | —        |
| Barra lateral                 | Preferências card         | —                                                      | Automática / Sempre aberta / Sempre recolhida, at once                               | —            | yes                               | the sidebar changes             | —        |
| Idioma                        | Preferências card         | —                                                      | switches the whole interface's language, at once                                     | —            | yes                               | every label changes             | —        |
| Switch on                     | PIN card                  | no PIN stored                                          | opens the define form (switching again closes it)                                    | —            | —                                 | —                               | —        |
| Gravar PIN                    | PIN form                  | 4 digits, both fields equal; when changing, the current PIN | stores the new PIN (checked by the app), stamps "gravado em"                     | —            | change or switch off              | toast "PIN gravado." / "PIN trocado." | —   |
| Switch off                    | PIN card                  | a PIN stored                                           | removes the PIN at once — **asks nothing**                                           | none         | define a new one                  | toast "PIN desligado."          | —        |
| Auto-lock chip                | PIN card (active)         | —                                                      | 1 / 5 / 15 min / nunca, saved at once                                                | —            | yes                               | the chip lights                 | —        |
| Trocar PIN                    | PIN card (active)         | —                                                      | opens the change form                                                                | —            | Cancelar                          | —                               | —        |
| Trancar agora                 | PIN card (active)         | —                                                      | locks the window now                                                                 | —            | unlock                            | the locked window               | —        |

### The locked window

| Gesture                               | Where         | Precondition                          | Effect                                                                                         | Confirmation | Reversible            | Feedback                         | Shortcut           |
| ------------------------------------- | ------------- | ------------------------------------- | ---------------------------------------------------------------------------------------------- | ------------ | --------------------- | -------------------------------- | ------------------ |
| Digit                                 | keypad        | fewer than 4 typed                    | adds a digit                                                                                   | —            | ⌫                     | a dot fills                      | `0`–`9`            |
| ⌫ Apagar                              | keypad        | —                                     | removes the last digit                                                                         | —            | —                     | —                                | Backspace          |
| ✓ Destrancar                          | keypad        | not already checking                  | the app checks the PIN; right → the window unlocks where it was                               | —            | —                     | error line on failure            | Enter              |
| Esqueci o PIN — destrancar por código | pad           | —                                     | shows the code panel and starts watching for a code (every 2 s while open)                     | —            | "Voltar para o PIN"   | —                                | —                  |
| Confirmar (code)                      | code panel    | a code typed                          | right code → **unlocks and turns the PIN off** in one act; the bell receives "O PIN foi desligado pelo código" | — | define a new PIN      | the screen behind returns        | Enter in the field |
| Bloqueio com PIN                      | no-PIN panel  | —                                     | unlocks and opens the Perfil                                                                   | —            | —                     | —                                | —                  |
| Voltar ao app                         | no-PIN panel  | —                                     | unlocks                                                                                        | —            | —                     | —                                | —                  |
| Fechar (X)                            | titlebar      | —                                     | locks again (re-stamping the moment) and closes the app                                        | —            | —                     | next launch opens locked         | —                  |

## Business rules

1. **Identity is self-declared and authorizes nothing** — there is no login.
   It is written immediately, like every preference; there is no Save.
2. **The e-mail is confirmed only in the wizard**, by a code generated in
   Thunderbird; the Perfil shows the state read-only, and any e-mail edit clears
   the confirmation — the code confirmed that address, not its replacement
   (Bridgr Pro decision 007).
3. **The face follows the person everywhere** — titlebar button, user menu,
   Perfil header and the locked window draw the same one, photo included; a face
   that changes between the corner and the profile reads as two people
   (decision 006 § 2).
4. **Drawn avatars are generated on this machine**; nothing is fetched. The
   seed starts as the name — same person, same drawing on any machine — and
   clicking the chosen circle walks **deterministic** variations (`name#2`,
   `name#3`…), pinned; a path that can be walked again, not a slot machine
   (decision 006 § 2). Initials use the first and **last** word; with no name,
   a generic glyph — never invented letters. An unknown stored style falls back
   to initials.
5. **The photo stays on this machine**: picked through the desktop's own
   chooser, never the webview's; framed so the circle shows exactly what is
   saved (256 × 256); one file, overwritten each time.
6. **Language names are written in their own language** — a picker that
   names the other language in the current one cannot be used by someone who
   does not read the current one (decision 002).
7. **Sidebar mode and language live here**, because they belong to the person;
   theme, palettes and density live in Configurações › Aparência.
8. **Station facts are real or "—"**, never invented.
9. **No week numbers on the Perfil — not planned** (the operator's decision);
   the design's "O que o seu trabalho ensinou" panel is not built.
10. **The PIN is a courtesy lock**: four digits, this machine only; it stops a
    colleague at an unattended desk, not an attacker. The digits are never
    stored — only a salted hash in the system wallet; whether a PIN exists is
    the wallet's answer, never a flag. Switching it off asks nothing (whoever is
    at the unlocked screen could change it anyway); changing it requires the
    current one, checked by the app (decision 008).
11. **Auto-lock**: 1, 5 or 15 idle minutes, or never (default 5); pointer,
    keyboard and wheel count as activity; armed only while a PIN exists. When it
    fires, the bell receives "A janela se trancou sozinha" — it happened without
    a hand (decision 010).
12. **The lock survives a restart**, and unlocking is decided by the app's core,
    never by the screen; a damaged lock record opens unlocked — a corrupt file
    must not brick the window shut.
13. **The way back without the PIN is a MailBridgr code** (purpose "Recuperar
    PIN" in the extension): same rules as the e-mail confirmation (5 minutes, 5
    attempts, single use, the profile's e-mail must match); it unlocks **and**
    turns the PIN off in one act (decision 008).
14. **Nothing behind the lock is reachable or told**: no sidebar, status bar,
    toasts or search; the menu and bell are disabled. The design's "Acontecendo
    por trás" services card and its "Encerrar sessão nesta máquina" are
    deliberately absent — the first would tell what the lock exists to hide
    (and asserted states nobody checked), the second has no login to fall back
    to (decision 008).

## Must survive the redesign

- The identity fields and their notes; the e-mail's confirmation state, with
  "confirmado" (never "verificado").
- Three avatar modes, the same face everywhere, click-to-walk-variations on the
  drawn style, the seed sentence, initials from first and last word.
- The framing step before a photo is used; "Cancelar" never saves.
- Endonyms in the language control.
- "—" for unknown station facts.
- The PIN card's three bodies; "off" asks nothing; change asks the current PIN;
  the auto-lock choices including "nunca".
- The locked window: nothing of the app visible behind it; name, face and
  minutes since locking; a keypad **and** the physical keys; the code escape
  that says it will turn the PIN off; the defensive no-PIN panel; toasts never
  on top.

## Free to redesign

- The Perfil's layout (header, columns, tiles, cards), the edit-mode pattern,
  the style picker's form, the dialog's look.
- The lock screen's composition, the keypad's look and size, the dots.

## Sample content for mockups

- Header: CR face · "Camila Duarte Rocha" · "Faturista OPME ·
  camila.rocha@empresa.example" · "Editar dados".
- Tiles: Nome "Camila Duarte Rocha" · Função "Faturista OPME" · E-mail
  `camila.rocha@empresa.example` — "confirmado em 05/09/2026 · 14:32" ·
  Assinatura `C. D. ROCHA · FATURAMENTO OPME`.
- Avatar: "Traço" chosen, seed `Camila Duarte Rocha#3`.
- Aside: "Aparência — Breeze Dark"; "Barra lateral: Automática"; "Idioma:
  Português (Brasil)"; Esta estação: `estacao-faturamento-01` · `0.1.0` ·
  `10/09/2026 · 08:02`.
- PIN card, active: "PIN de 4 dígitos ativo · gravado em 05/09/2026 · 14:32",
  chip "5 min" lit.
- Locked window: "Janela trancada" · CR · "Camila Duarte Rocha" · "trancada há
  6 min" · two dots filled; a wrong try shows "PIN errado.". Code panel ready
  with `7KQM-2XRD`.

## Design opportunities

1. **A "Foto" choice in the style row.** Once another mode is chosen, the
   stored photo cannot be chosen again without picking and framing it anew, and
   there is no way to remove it. A "Foto" chip (when a photo exists) and a
   remove gesture would close both gaps — the second needs implementation.
2. **Make the re-roll discoverable.** Walking variations by clicking the big
   circle is announced only by a platform tooltip.
3. **Confirm the e-mail from here.** The tile says "não confirmado" with no
   gesture beside it; the app's own spec calls a "confirm now" action a
   candidate follow-up. Needs implementation.
4. **One word for locking.** The PIN card says "Trancar agora", the user menu
   says "Encerrar sessão" for the same act, and the original design said
   "Trancar a janela". Proposal only.
5. **One label for the role.** "Função" here, "Cargo" in the wizard, with
   notes that disagree ("quem aprova conta neste app" vs "Só um rótulo — não
   muda o que você pode fazer."). Proposal only.
6. **Say what the lock did not do.** The lock only says "trancada há X min";
   the notice for an automatic lock already says "Nada foi fechado nem perdido".
   Carrying that true sentence onto the lock would reassure without asserting any
   service state.
7. **Show the code's time left** in the code panel — the app knows the expiry.
8. **Submit on the fourth digit** — today the PIN needs ✓ or Enter. A proposal
   that changes behaviour; it needs the operator's call.

## Open questions

- Will "O que o seu trabalho ensinou" (the design's learned-corrections panel)
  arrive, and fed by what?
- The design's login screen ("Entrar") is unbuilt; it would arrive with real
  sessions, and the lock screen is not it. Is it still planned?
