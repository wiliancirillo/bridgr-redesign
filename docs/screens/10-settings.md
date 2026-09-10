# Configurações — `/settings`

> Where: sidebar group **Sistema › Configurações** (gear icon); also the titlebar user menu, the status bar, the bell and the portal screens — see [Deep links](#deep-links) · Status: built — eight of nine sections; **Exportação** is an empty room · Code: `src/routes/settings/+page.svelte`; `src/lib/components/service-fields/*.svelte` (14 files); `src/lib/components/CheatsheetDialog.svelte`, `RegistryCodeField.svelte`, `SpookClientsDialog.svelte`, `TomlEditor.svelte`; stores `src/lib/preferences.svelte.ts`, `theme.svelte.ts`, `medlens.svelte.ts`, `fantasma.svelte.ts`, `spook.svelte.ts`, `nfepack.svelte.ts`, `secret-status.svelte.ts`, `opmenexo.svelte.ts` (the auto-read clock); labels `src/lib/i18n/locales/pt-BR.ts`; rules enforced on the Rust side in `src-tauri/src/secrets.rs`, `cheatsheet.rs`, `spook_settings.rs`, `preferences.rs`, `lib.rs`

How this file is organised: the template headings at level 2 describe the
screen as a whole — its own navigation, how saving works, the secret rule.
Then comes one part per section, in the order of the section rail (Sections 1–9, the
template headings at level 3). The last part, [Shared field
primitives](#shared-field-primitives), describes the fields and cards the
sections share with the [Primeira montagem](15-welcome.md) wizard.

## The question

How is this station set up: which programs does it talk to, is each one
actually up, with which folders and credentials, and how does the app look?
This screen is the console of the ecosystem ([01-product.md](../01-product.md)).
A new machine is set up here, and a fallen service is revived here.

The app carries its own phrasing in i18n, "O que já é real neste app, e como
ajustá-lo." (`question.settings`), but this page does not draw it; the line
under the title shows the current section's hint instead.

## Where it sits in the work

Off the conveyor: nothing here is daily work. The connections to MedLens,
Spook and Fantasma are **born** in the first-assembly wizard
([Primeira montagem](15-welcome.md)), which mounts the same panels and fields;
this screen **maintains** them (check, tweak, reissue, revive). It also holds
everything the wizard never asks: appearance, the Doca folders, the
NFePackMonitor, the adapters' colinhas, the OPMENEXO clients, the auto-read
clock and the autostart switches. The operator usually lands here from a
signal elsewhere: a status-bar block, a bell notice after a failed boot rise,
a portal screen pointing at the Spook, or the user menu's "Preferências".

## Anatomy

1. **Header** (full width, divider under it)
   - Title "Configurações".
   - Sub-line: the **current section's hint** (verbatim in the table below). It
     changes with the section.
2. **Body**, two columns:
   - **Section rail**: the screen's own navigation, separate from the app
     sidebar. It is a fixed column of about 198 px with its own divider and its
     own scroll. It holds nine items (icon + label) in the fixed order below.
     The current item is drawn as an accent tint plus accent ink, never a solid
     flood. Every item is a link to `/settings?section=<id>`.
   - **Content column** (scrolls; content held in a left-aligned column of
     max ~660 px). From the top:
     - **Save-failure strip**, only after a preference write failed (see
       States). It is the first thing in every section.
     - The **section's own content** (Sections 1–9).

| # | Rail label | `?section=` | Icon (Phosphor) | Header hint (verbatim) | Built |
|---|---|---|---|---|---|
| 1 | Sobre | `about` | info | "Nove assuntos — o que já é real neste app é a porta de entrada" | yes (the entry point) |
| 2 | Aparência | `appearance` | palette | "Tema, galerias e densidade" | yes |
| 3 | Doca | `dock` | tray | "De onde os informes chegam" | yes |
| 4 | MedLens | `medlens` | eye | "Serviço, credencial, IA e gasto" | yes |
| 5 | Fantasma | `fantasma` | robot | "Pastas, credenciais e sessão" | yes |
| 6 | Spook | `spook` | plugs-connected | "Endereço e credencial do serviço de portais" | yes |
| 7 | NFe Pack | `nfepack` | receipt | "Serviço e pasta do monitor de NF-e" | yes |
| 8 | Adapters | `adapters` | plugs | "Clientes, prompt de extração e colinha" | yes (no prompt editor, see Section 8) |
| 9 | Exportação | `export` | export | "Formato e planilha de registro" | **no** (empty room) |

## Data shown

At screen level there are only the section list and the save-failure strip. Each
section's data is in its part. What every section shares:

- **Preferences** come from `config.json`, read when the app starts and held
  in memory. The screen always shows the in-memory value, which is also the one
  in force.
- **Secret fields never show a value**, only whether one is stored (badge
  "gravada" and a masked placeholder), see [SecretField](#secretfield).
- **Service states are live, and only while their section is on screen**:
  MedLens is polled every 3 s; Fantasma every 10 s, and every 2 s during a
  transition or the release countdown; Spook every 5 s; NFePackMonitor every
  5 s. Leaving the section stops its poll.

### Deep links

| From | Lands on |
|---|---|
| Sidebar › Sistema › "Configurações" | Sobre |
| Titlebar user menu › "Configurações" | Sobre |
| Titlebar user menu › "Preferências" | Aparência |
| Perfil › card "Preferências" › shortcut "Aparência" | Aparência |
| Status bar: MailBridgr/Doca block | Doca |
| Status bar: MedLens block | MedLens |
| Status bar: Fantasma block | Fantasma |
| Status bar: Spook block | Spook |
| Status bar: NFePackMonitor block | NFe Pack |
| Status bar: version chip `v0.1.0` (tooltip "Sobre este app") | Sobre |
| Bell notices `medlens_autostart_failed` · `fantasma_autostart_failed` · `nfepack_autostart_failed` | MedLens · Fantasma · NFe Pack |
| `/opmenexo` and its order dialog, wherever they point at the Spook | Spook |

A missing or unknown `?section=` value opens **Sobre**, never an error.
Changing section is ordinary navigation, so it creates history entries.

## States

| State | Trigger | What is drawn |
|---|---|---|
| Entry | `/settings` with no or an unknown `section` | Sobre; rail item "Sobre" current |
| Preference write failed | the disk refused a `config.json` write | The setting **still applies for this session**. A danger strip at the top of the content reads "A preferência vale nesta sessão, mas não foi gravada:" followed by the refusal text, and a danger toast carries the same sentence (the toast reaches whoever changed a setting from any other screen). The next write that works clears it. |
| Section left | picking another section | its polls stop; only the service and secret stores keep their last answers |
| Narrow window | near the 1024 × 768 minimum | Nothing adapts. The rail keeps ~198 px and the content column shrinks. The two-column pairs (user + password, primary + secondary model, notes folder + pause) stay side by side and compress. The palette cards (~200 px each), the Gasto header and the metrics tiles wrap. Dialogs cap at 92 % of the window width and 84 % of its height. |

Refusals are shown as the other side wrote them: often an English sentence,
sometimes with a file path. See Business rules 7.

## Actions

| Gesture | Where | Effect | Feedback |
|---|---|---|---|
| Pick a section | rail | opens it; URL becomes `?section=<id>` | the item is marked current |
| Change a preference (text, switch, segment, palette card, checkbox) | any section | **written immediately; this screen has no global "Salvar".** Text fields commit on `change` (leaving the field, or Enter); switches, segments, cards and checkboxes commit on click. | Silent on success, because the screen already shows the effect; on failure, the strip and the toast |
| Type a secret | secret fields | stored in the system wallet when the field is left non-empty; the field then empties itself | badge "gravada"; wallet refusals inline |
| The only two explicit saves | Colinha dialog "Gravar" · OPMENEXO clients dialog "Salvar" | Both edit files that another program reads, so they validate and write only on the button. | toasts, in their parts |

No section has a filter field, so `Ctrl ⇧ K` does nothing here. In-section
keys: the registry-code fields (↑ ↓ Enter Esc) and the TOML editor (`Ctrl+S`).

## Business rules

1. **Every change writes immediately, and the screen changes before the disk.** A
   button would ask the operator to agree to what they can already see. If
   the write fails, the value still applies now and the failure is said out loud.
   The reverse order would be a click that seems to do nothing.
   (`preferences.svelte.ts`)
2. **A preference that did not persist is announced**, inline and as a toast.
   Otherwise it comes back wrong at the next launch with nothing to connect
   it to.
3. **Passwords never come back to the screen, and blank means "I did not touch
   it", never "erase it".** Without that rule, saving the queue folder would
   erase the two credentials beside it. The Rust side also refuses an empty
   secret. ([02-invariants.md](../02-invariants.md) § 12; `secrets.rs`)
4. **The secrets are a closed set.** Only five can be typed: the OpenRouter
   key, the RDP and Everest credentials, and the two portal passwords
   (OPMENEXO, ACOM). The MedLens and Spook credentials are **issued** by the
   app through each service's own tool, and nobody can type them.
5. **Polls live only while their section is shown**, so a closed settings
   page costs the services nothing.
6. **A service somebody else started can be used, but never stopped from here**
   ("de fora do app"). The app also never raises a service it could not use
   (the Spook without a portal login).
7. **The other side's words are shown verbatim.** Service, TOML and wallet
   refusals arrive as the Rust side wrote them, mostly in English. The design
   must leave room for a long sentence it did not write.
8. **An unknown section is the entry point, never an error.**

## Must survive the redesign

- The secret-field contract: no value on screen, ever; a masked placeholder
  when stored; blank = untouched; and no fake "clear" gesture
  ([SecretField](#secretfield)).
- Write-immediately, with the "applies now, not saved" failure path. Never a
  silent failure, and never a "Salvar?" guarding a preference.
- Each service's full set of states, drawn apart
  ([02-invariants.md](../02-invariants.md) § 15), with the sign of life on the
  control itself while a gesture is in flight.
- The nine `?section=` ids: the status bar, the bell, the user menu and the
  portal screens link to them.
- Every refusal verbatim, in place, next to the thing refused.

## Free to redesign

- The inner rail as a vertical list: any navigation works if it keeps nine
  addressable sections.
- Card shapes, the content column's width, the hint's placement.
- Grouping the sections differently (for example a "Serviços" group), as long
  as each one keeps its deep link.

## Sample content for mockups

Per section, in each part. App version `0.1.0`; folders `~/Projetos/medlens`,
`~/Projetos/fantasma`, `~/Projetos/spook`, `~/Projetos/nfepackmonitor`, queue
`/srv/fila-fantasma` ([06-sample-data.md](../06-sample-data.md)).

## Design opportunities

1. **Group the rail by kind.** Four sections are consoles of a running
   service (MedLens, Fantasma, Spook, NFe Pack) and share one grammar. Doca and
   Adapters feed the conveyor, Aparência is a preference, and Sobre lists the
   programs. A grouped rail would show that the screen is a console.
2. **A live state mark beside each service item in the rail**, so the broken
   service can be picked without opening each one. It must obey "green only
   after somebody asked": draw nothing before the first answer.
3. **MedLens as its own screen.** Bridgr's own screen inventory already names
   it the first candidate: it alone carries service, credential, AI settings
   and a spend panel with a chart and three tabs.
4. **Every disabled control says why.** The switch supports a hover reason,
   but several disabled switches and buttons carry none today (noted in each
   section).
5. **One state vocabulary across the consoles.** The same situation is worded
   differently per service today (quoted verbatim in each section, for example
   the foreign state). Any new wording is a proposal.
6. **Room for long verbatim refusals**: a failure strip that can hold two
   lines and lets the text be selected and copied.
7. **Say once that changes apply immediately.** Nothing on screen says there
   is no Save; a quiet line would stop the operator looking for one. This is
   new text, so it is a proposal.

## Open questions

- Exportação: what it will hold beyond its hint ("Formato e planilha de
  registro") is defined nowhere in the code or the records.
- Should the rail show which sections need attention (a service down, a
  colinha incomplete)? Not decided.

---

## Section 1 — Sobre (`?section=about`)

### The question

Which programs does this station talk to, who does what, and which of them
rise when the app starts?

### Where it sits in the work

The entry point of the screen, reached from the sidebar, from the user menu's
"Configurações" and from the status bar's version chip.

### Anatomy

1. **About card**: title "O que já é real neste app", then the paragraph
   (verbatim): "O Bridgr Pro é a estação humana da esteira: MailBridgr entrega
   os anexos, o MedLens lê, você tria e confere, o Fantasma lança no Everest.
   Ele não emite nota e não dirige o robô — enfileira e observa. O Spook fala
   com os portais e o NFePackMonitor baixa as notas; o interruptor de cada
   linha diz quem sobe junto com o app."
2. **Six program rows**, stacked. Each row holds an accent icon, what the
   program does, its proper name (mono, faint) and, only where allowed, the
   **autostart pair**: a small label plus a switch. **[functional]** The switch
   sits on the row of the program it speaks for.

   | Icon | What it does (verbatim) | Program | Switch |
   |---|---|---|---|
   | tray | "Doca vigiada, anexo por anexo" | MailBridgr | never |
   | eye | "Leitura das contas e aprendizado por correção" | MedLens | when the MedLens folder is set |
   | check-square-offset | "Conferência com o documento ao lado e portão de completude" | Bridgr Pro | never |
   | robot | "Lançamento no Everest, uma conta por vez" | Fantasma | when the Fantasma project folder is set |
   | plugs-connected | "Portais lidos e respondidos daqui" | Spook | when the Spook **folder** is set (an address alone does not count) |
   | receipt | "Baixa de NF-e e DANFE, ciclo a ciclo" | NFePackMonitor | when the NFePackMonitor folder is set |

   Label: "Sobe com o app". After the operator flips that row's switch during
   this visit, the label reads "Vale no próximo início do app". The switch's
   accessible name is "Sobe com o app — <program>".
3. **Identity line** at the foot, after a hairline, in faint mono:
   `Bridgr Pro · 0.1.0 · br.alphamedical.bridgrpro` (name · version · bundle
   identifier, as the running binary knows them).

### Data shown

- Program names are proper nouns, identical in every locale.
- Switch position = the stored choice (`services.<service>.autostart`), **not**
  whether the service is up now. Defaults: MedLens on, Spook on, NFePackMonitor
  on, **Fantasma off**.

### States

- Service not configured: its row shows no label and no switch.
- Flipped during this visit: that row's label reads "Vale no próximo início do
  app" until the operator leaves the section. The flag is not stored.
- Write failed: the screen's save-failure strip and toast.

### Actions

| Gesture | Precondition | Effect | Confirmation | Reversible | Feedback |
|---|---|---|---|---|---|
| Flip "Sobe com o app" | that service's folder is set | stores the choice; it governs only the **next** app start | none | yes, flip back | the label changes; no toast |

Nothing is raised or stopped now. The switch on each service's own card stays
the one way to raise a service by hand.

### Business rules

1. **A switch only where there is something to rise** (Bridgr decision 035).
   MailBridgr is a Thunderbird extension the app never starts, and Bridgr Pro
   cannot raise itself, so their rows never carry one. An unconfigured
   service's row shows none: a switch that cannot act claims a choice nobody
   can make.
2. **The switch governs only the boot rise.** With the switch off, a service
   already running is still adopted at boot, because recognising what exists is not
   starting it.
3. **The Fantasma defaults to off**: raising it takes over the ERP remote session,
   which a colleague may be using during the transition.
4. **The Spook's anchor is its folder.** With only an address, somebody else
   raises it and this app has nothing to rise.

### Must survive the redesign

The six rows and the two that never carry a switch; the switch **hidden**, not
disabled, for an unconfigured service; the "next start" wording after a flip;
the Fantasma's default.

### Free to redesign

Card vs list, icon treatment, where the identity line sits.

### Sample content for mockups

All four startable services configured: MedLens on, Fantasma off, Spook on,
NFePackMonitor on and just flipped ("Vale no próximo início do app").

### Design opportunities

1. **Give "rises with the app" its own look.** Today this switch is the service
   switch, whose "on" is the green that means "the service is up". So a stored
   preference wears the colour reserved for "somebody checked"
   ([02-invariants.md](../02-invariants.md) § 3). A neutral or accent "on"
   here would keep green for facts.
2. Show the service's live state on its row (or link to its section): today
   the row says what rises, not what is up.

### Open questions

- Should the "Vale no próximo início do app" hint last the whole session?
  Bridgr's decision 035 says so; the code keeps it for the visit only.

---

## Section 2 — Aparência (`?section=appearance`)

### The question

Which look, in which light, and how tight?

### Where it sits in the work

A personal preference, also reached from the user menu's "Preferências" and
from the Perfil shortcut. The sidebar mode and the language are **not** here:
they live on [Perfil](16-profile-and-lock.md).

### Anatomy

1. **"Tema"**: label plus a segmented control of three: "Sistema" · "Claro" ·
   "Escuro". Exactly one is pressed.
2. **"Paleta clara"**: label plus a wrapping gallery of cards, one per
   light-based theme.
3. **"Paleta escura"**: the same for dark-based themes. **[functional]** There
   is one gallery per base, and each base keeps its own choice.

   Each card (~200 px) shows:
   - a row of **four swatches** painted in that theme's **own** colours:
     ground, surface, accent, ink;
   - the theme's name, with a filled check icon when chosen;
   - a mono line with two hexes: ground · accent.

   The chosen card has an accent outline and a faint accent tint, with its name
   in accent.
4. **Density checkbox** (a bordered row): "Densidade compacta", with the hint
   "Aperta a interface inteira — listas, tabelas, barra lateral e barra de
   título."
5. **Config door**, set apart below everything by a divider:
   - the hint (verbatim) "Novos temas entram em config.json: copie um dos
     modelos gravados lá, dê outro nome e repinte os tokens — cada tema escolhe
     uma base e pinta por cima dela.";
   - an outlined accent button with a file-code icon: "Abrir o config.json";
   - a failure line under the button when opening fails.

### Data shown

| Theme (built-in) | Base | ground | surface | accent | ink | Hexes on the card |
|---|---|---|---|---|---|---|
| Breeze Light | light | #eff0f1 | #fcfcfc | #3daee9 | #232629 | `#eff0f1 · #3daee9` |
| GitHub Light | light | #ffffff | #f6f8fa | #0969da | #1f2328 | `#ffffff · #0969da` |
| Breeze Dark | dark | #232629 | #31363b | #3daee9 | #eff0f1 | `#232629 · #3daee9` |
| GitHub Dark | dark | #0d1117 | #161b22 | #2f81f7 | #e6edf3 | `#0d1117 · #2f81f7` |

- Breeze is the palette the app was drawn against; the GitHub pair are alternatives.
- In each gallery the built-ins come first, then the custom themes from
  `config.json`, labelled by their key name (never translated).
- On a custom theme card, a missing colour draws its swatch transparent and
  its hex as "—".
- The marked card is the name stored for that base. When nothing is stored, it
  is that base's Breeze card (chosen by name), because the bare base is Breeze.

### States

- First run: "Sistema" pressed; Breeze Light and Breeze Dark marked.
- Normal: one marked card per gallery (for the exception, see Open questions).
- Opening `config.json` failed: the refusal text under the button, in the
  danger tone.
- Write failed: the screen's strip and toast.

### Actions

| Gesture | Effect | Feedback |
|---|---|---|
| "Sistema" / "Claro" / "Escuro" | the mode is stored and applied at once; "Sistema" follows the desktop's light/dark live | the app repaints |
| Click a palette card | stores that base's palette; the app repaints at once if that base is in force; **the mode is not touched** | the card is marked |
| "Densidade compacta" | the whole interface, chrome included, switches between standard and compact spacing | immediate |
| "Abrir o config.json" | if the file's `themes` list is empty, first writes the four built-ins into it as models; then opens the file with the desktop's handler | the file opens outside the app; a failure appears under the button |

### Business rules

1. **One gallery per base, and picking a palette never changes the mode.**
   This is a deliberate departure from the original design (Bridgr decision 006
   § 2). With four palettes, "which look" and "when" are different questions.
   Tying them would make every palette click override "follow the system".
2. **Swatches paint the theme's own colours**, not the ones in force: a
   gallery that looks like the applied theme previews nothing.
3. **A theme paints over a base; it never replaces it.** A theme may set only
   six tokens: ground (`bg`), surface, ink (`text`), divider, accent and desk.
   Any other key is ignored silently. Two groups are **excluded on purpose**:
   - the state roles: amber, green and red carry meaning, and a theme that
     repaints danger green is wrong, not ugly;
   - **paper** and its ink: the digitised document must look like paper in
     both themes.

   (`theme.svelte.ts`)
4. A theme without a valid base (`light`/`dark`) is ignored. A file entry
   named like a built-in is shadowed, so the seeded models do not double the
   gallery. To make a theme from a model, copy it under another name.
5. This density is the whole app. The grid's row density is a separate control
   over each grid ([05-components.md](../05-components.md)).

### Must survive the redesign

Mode and palette as separate choices; each card previewing its palette in its
own colours; the roles no theme can repaint (state, paper); custom themes
limited to the six tokens; the door to `config.json` for hand-written themes.

### Free to redesign

Segmented control or radios; card size, swatch count and shape; whether the hexes show; where
the config door sits.

### Sample content for mockups

- Mode "Sistema".
- Paleta clara: Breeze Light (marked), GitHub Light.
- Paleta escura: GitHub Dark (marked), Breeze Dark, and a fictitious custom
  theme "noite-vale" (ground `#1b1f24`, surface `#242a31`, accent `#4fb3a9`,
  ink `#e8eaed`).
- Density unticked.

### Design opportunities

1. **Preview beyond four swatches**: a small fragment per card (a row, one pill
   in each state role, a button). It would also show that the state roles do
   not change with the palette.
2. **Show where custom themes come from** ("do config.json"), so the operator
   knows where to edit them.
3. **Say when a custom theme was ignored.** Today a malformed entry silently
   disappears from the gallery. This is a proposal and needs code.

### Open questions

- If the stored palette no longer exists in `config.json` (renamed or removed),
  the app paints the bare Breeze but **no card is marked** in that gallery.
  Should the gallery mark Breeze, or say the choice was lost?

---

## Section 3 — Doca (`?section=dock`)

### The question

Where do the informes arrive from?

### Where it sits in the work

It feeds [Triagem](02-triage.md) through two entrances: the MailBridgr
delivery folder (e-mail) and the watched folder (loose files). The status
bar's MailBridgr block opens this section.

### Anatomy

Two [FolderField](#folderfield)s, stacked:

1. "Pasta de entregas (inbox do spool)"
   - placeholder "padrão: a pasta de dados do próprio app";
   - note "Em branco usa o padrão — o mesmo caminho que o MailBridgr resolve
     sozinho. Só preencha se o spool morar em outro lugar."
2. "Pasta vigiada"
   - placeholder "em branco: sem pasta vigiada";
   - note "Arquivo copiado aqui com o app aberto aparece na doca em até 4 s —
     PDF e foto, sem e-mail no meio. Em branco, a entrada não existe."

### Data shown

Two paths, free text (`~` allowed). Empty is the **common case** for both.

### States

- Both empty (default): the dock reads the app's own spool, and there is no
  second entrance.
- Filled: the path as typed or picked.
- The folder picker failed: a danger toast with the refusal.

### Actions

Type a path, or use "Escolher" to open the system folder dialog (titled with
the field's label). Either way the path is written immediately. Cancelling the
dialog keeps what is there. This section validates nothing: a folder that does
not exist is an empty entrance.

### Business rules

1. An empty spool folder means the default, the path both sides find with
   nobody configuring it (MailBridgr resolves the same one).
2. The watched folder is a second entrance for loose files (Bridgr
   decision 020):
   - it reads top-level PDF, PNG, JPG, JPEG and WEBP files: no subfolders, no
     `.eml`;
   - a file copied over an old one is a new document;
   - a folder holding a delivery `manifest.json` reads empty, because pointing
     it inside the spool would list files twice.
3. The app never moves or copies a watched file.

### Must survive the redesign

What blank means is said on each field: blank = the default spool; blank = no watched
entrance. Free text stays beside the picker.

### Free to redesign

Everything visual.

### Sample content for mockups

Spool field empty (placeholder showing); watched folder `~/Documentos/Informes
soltos`.

### Design opportunities

1. Show what each entrance sees now (for example "3 arquivos na pasta
   vigiada"). This needs a read, so it is a proposal.
2. **Several named watched folders** are an operator request recorded in
   Bridgr's memory and not built; the memory asks for the design to draw it
   first. The design could draw a list of named folders (name + path) built
   from the same field.

### Open questions

- The shape and naming of multiple watched folders (above) are not decided.

---

## Section 4 — MedLens (`?section=medlens`)

### The question

Is the reading service up and accepting this app? Which AI models does it use,
with which key, and how much has it spent?

### Where it sits in the work

MedLens reads every informe ([Triagem](02-triage.md) → [Revisão](03-review.md)).
Its connection is born in the wizard's MedLens step and maintained here. The
status bar's MedLens block and the `medlens_autostart_failed` notice open this
section.

### Anatomy

Three blocks, top to bottom.

**A. Panel** (shared with the wizard)

1. **Service card**:
   - icon eye: neutral grey when idle, green when up, amber while starting;
   - title "Serviço" + " · <state>" + version chip `v<version>` (mono, faint,
     only when known);
   - sub-line "uvicorn em 127.0.0.1:8000 — sobe com o app, desce com ele";
   - conditional amber line "Este MedLens não tem: <capabilities>";
   - conditional sub-line when foreign: "Este MedLens foi ligado por outro
     programa — usar, sim; derrubar daqui, não.";
   - the [service switch](#servicetoggle) at the right edge.
2. **Credential card**: icon key; title "Credencial"; the state sentence; a
   button at the right reading "Emitir", "Emitir outra" or "Emitindo…" (with a
   spinner, disabled).
3. **Failure strip**: the last gesture's refusal.

**B. Fields** (shared with the wizard)

1. FolderField "Pasta do projeto MedLens"
   - placeholder `~/Projects/medlens`;
   - note "De onde o app sobe o serviço (uv run uvicorn). É a pasta com o
     pyproject.toml do MedLens."
2. SecretField "Chave da OpenRouter"
   - format placeholder `sk-or-v1-…` while no key is stored;
   - note "Fica só nesta máquina, no cofre do sistema. Depois de gravada,
     campo em branco significa não mexi — nunca apagar."
3. A side-by-side pair of mono text inputs, each with a suggestion list (the
   OpenRouter catalogue: id plus the model's name):
   - "Modelo primário", placeholder `anthropic/claude-3.7-sonnet`;
   - "Modelo secundário", placeholder `google/gemini-2.5-flash`.
4. Wallet failure strip: "O cofre recusou: <reason>".

**C. Gasto card**

1. **Header row**, which wraps:
   - title "Gasto";
   - total `US$ 12,48` (mono, accent);
   - sub "<n> extrações · US$ <x> nos últimos 7 dias";
   - the tab group "Por dia" · "Por modelo" · "Extrações" (only once data
     arrived);
   - an icon button with the tooltip "Atualizar", spinning while it asks.
2. **Partial-history line** (amber, conditional):
   - one row: "<n> extração ficou de fora — foi paga por outra chave de API;
     o registro continua no MedLens.";
   - several: "<n> extrações ficaram de fora — foram pagas por outra chave de
     API; o registro continua no MedLens."
3. **Body**, by tab:
   - **Por dia**:
     - 14 bars for the last 14 calendar days ending today, oldest on the left,
       each labelled with its day of month (mono);
     - height is relative to the peak day, with a 4 % minimum; the peak bar is
       full accent and the others a darker accent;
     - a day without spend is a short stub in the quiet track colour, with a
       fainter label;
     - bar tooltip "<day> · US$ <x>", or "<day> · nada lido".
   - **Por modelo**: one row per model, largest spend first:
     - model id (mono, truncated; "—" when empty), with the note "<n>
       extração / extrações · média US$ <x>";
     - a horizontal bar relative to the largest (min 3 %);
     - the spend (mono) and the share of the total in % (mono, faint).
   - **Extrações**:
     - a table with a sticky header: "Quando" · "Arquivo" · "Modelo" ·
       "Tokens" · "Custo" (the last two right-aligned);
     - up to 100 rows, in a height-capped box that scrolls;
     - when the history is longer, the line "Mostrando as 100 mais recentes
       de <N>." underneath.
4. **Footer note**, always shown, with an info icon: "Só chamadas do MedLens:
   a Consulta e o Fantasma não gastam API. O gasto é o que a OpenRouter cobrou,
   registrado a cada leitura."

### Data shown

| Field | Meaning | Presentation |
|---|---|---|
| Service state | the `/health` answer | "no ar" · "subindo…" · "no ar — subiu fora deste app" · "parado" · "…" (never asked) |
| Version | from `/health` | chip `v…`, only when known |
| Missing capabilities | what this build needs that `/health` did not list | amber line, ids separated by commas, e.g. "Este MedLens não tem: adapters.prompt, adapters.learning" |
| Credential | the app's own console credential, checked by the service | "aceita pelo serviço" (green) · "nenhuma emitida ainda — o app emite a que ele mesmo usa" · "recusada (401). No ar sem credencial dá 401 em tudo com cara de serviço fora — reemita antes de culpar a rede." (danger border and text) · "sem como conferir com o serviço parado" |
| Total | spend of the key configured **now** | `US$` with a decimal comma; 2 decimals, or 4 when the value is below one cent |
| Extractions | spend rows of this key | pt-BR thousands ("1.234") |
| Last 7 days | sum over the last 7 × 24 h | `US$` |
| Other keys | rows paid by other API keys, not in the total | the amber sentence |
| Por dia | sum per local calendar day | 14 bars, zero days included |
| Por modelo | total, count, average, share | sorted by total |
| Extrações | when (`dd/mm/aaaa hh:mm:ss`, local), file name, model id, input + output tokens, cost | numbers and model in mono |

⚠️ In real use, the "Arquivo" column shows informe file names, which can carry
a patient's name. Mockups use only the cast's file names.

### States

**Service card**

- "…": never asked; idle.
- "parado": idle.
- "subindo…": amber icon.
- "no ar": green icon.
- "no ar — subiu fora deste app": green icon, the extra sub-line, the switch
  on and locked with the foreign sentence on hover.
- The switch is disabled when the service is down **and** no folder is set,
  with no hover reason.
- The switch spins only for this window's own gesture. During a rise started
  by the app's boot task ("subindo…"), it stays off and movable.

**Credential button**

- Disabled while the service is not up (running or foreign), with no reason
  shown.
- While **any** MedLens gesture is in flight (switch or issue), it reads
  "Emitindo…" with a spinner, and the switch spins too.

**Gasto card**

| State | What is drawn |
|---|---|
| Not asked yet (service state unknown) | only "Gasto" and the refresh button; no sentence |
| Service known to be down, nothing loaded | "Sem dados — o MedLens precisa estar no ar."; refresh disabled |
| Asking | the refresh icon spins, disabled |
| Loaded, zero rows | total `US$ 0,00`, "0 extrações · …", the tabs, and the body "Nenhum gasto registrado com esta chave ainda." (plus the partial-history line if other keys paid something) |
| Loaded | the current tab's body |
| Failed | a danger box with the refusal replaces the body; the header keeps any earlier values |

The card loads once each time the service comes up (it follows the panel's
poll). After that, only "Atualizar" reloads it.

**Fields**

The model suggestions appear once the service has been up with an accepted
credential. Before that, the inputs take free text with no list.

### Actions

| Gesture | Where | Precondition | Effect | Feedback |
|---|---|---|---|---|
| Switch on | service card | folder set; service down | raises MedLens: starts it, waits for `/health`, ensures the credential, pushes the AI settings | spinner (accessible name "Ligando…"); toast "MedLens no ar."; refusal in the strip |
| Switch off | service card | up and started by this app | stops it | spinner ("Desligando…"); toast "MedLens desligado." |
| "Emitir" / "Emitir outra" | credential card | service up | issues a new console credential through MedLens's own CLI and stores it in the wallet | "Emitindo…"; toast "Credencial emitida." |
| Edit the folder | field | — | stored | silent |
| Paste the OpenRouter key | secret field | non-empty | wallet, then (if connected) pushed to MedLens, then read back | the toast sequence below |
| Edit a model | field | — | stored; pushed to MedLens at once when connected (quietly), otherwise at the next rise | silent |
| Change tab | Gasto | data loaded | changes the view; not stored | — |
| "Atualizar" | Gasto | service up, not already asking | reloads the spend | spinning icon |

None of these gestures asks for confirmation.

The key paste is answered step by step, in toasts:

1. "Chave gravada no cofre." once the wallet accepted it.
2. If MedLens is not up with an accepted credential: "O MedLens está fora do
   ar — a chave será aplicada quando ele subir."
3. Otherwise the key is pushed and read back, and one of these follows:
   - "Chave aplicada no MedLens (<hint>). Se ela vale na OpenRouter, só a
     próxima leitura prova." The <hint> is the service's own masked hint of the
     key; its format was not verified.
   - danger: "O envio passou, mas o MedLens diz que está sem chave — reaplique
     em cima do serviço no ar."
   - danger: "Gravada no cofre, mas o MedLens não aceitou o envio:" followed
     by the refusal.

Refusals the strip can show (verbatim, from the Rust side):

- "MedLens: the MedLens folder is not configured"
- "MedLens: <folder> does not look like the MedLens project (no pyproject.toml)"
- "MedLens: the service did not answer /health in time"
- "MedLens: this MedLens was not started by this app — it stays up"

### Business rules

1. **The service follows the app**: raised at boot when configured and allowed
   by the Sobre switch, and stopped at exit (only the instance this app
   raised). This card is the manual fallback, not the routine. The PIN lock
   never touches it. (Bridgr decision 011)
2. **Green only after `/health` answered**, never a guess.
3. **A 401 means "refused, reissue", never "network"**: a service that is up
   without a credential answers 401 to everything and looks like a service that
   is down.
4. **The credential is issued, never typed**, and it never crosses to the
   screen.
5. **Models are free text** backed by the catalogue: an id outside the list is
   a just-released model, not a typo.
6. **The key's journey is said step by step, including its honest limit**:
   MedLens cannot validate a key, so only the next real reading proves that
   OpenRouter accepts it.
7. **Spend belongs to the key that paid it.** Rotating the key zeroes the
   card, so the partial-history line is load-bearing: a zeroed total must never
   read as "nothing was ever spent".
8. **Only the cuts the data answers**: by day, by model and the extraction
   log. The original design's by-adapter and by-outcome cuts are left out on
   purpose, because MedLens does not record those facts per spend row (Bridgr
   decision 006 § 2).
9. The log shows at most 100 rows; the whole history lives in MedLens.

### Must survive the redesign

- The four service states, plus "never asked".
- The credential's four sentences; "refused" is never drawn as a network
  problem.
- The step-by-step key toasts, including "only the next reading proves".
- The partial-history warning.
- No invented spend cuts.
- Money as `US$` with a decimal comma.
- A foreign instance shown on and locked.

### Free to redesign

Card composition; the chart style (a line or area chart works if zero days still appear
as days); the tab control; table styling.

### Sample content for mockups

- Service: "Serviço · no ar" `v1.8.0` (fictitious); credential "aceita pelo
  serviço" with "Emitir outra".
- Folder `~/Projetos/medlens`; key stored (dots and "gravada"); models
  `anthropic/claude-3.7-sonnet` and `google/gemini-2.5-flash`.
- Gasto: `US$ 12,48` · "342 extrações · US$ 3,17 nos últimos 7 dias", with
  "57 extrações ficaram de fora — foram pagas por outra chave de API; o
  registro continua no MedLens."
- Por modelo:

  | Model | Note | Spend | Share |
  |---|---|---|---|
  | `anthropic/claude-3.7-sonnet` | "298 extrações · média US$ 0,03" | US$ 9,02 | 72% |
  | `google/gemini-2.5-flash` | "44 extrações · média US$ 0,08" | US$ 3,46 | 28% |

- Extrações row: `09/09/2026 14:02:11` · `informe_vale_verde_2026-09-02.pdf` ·
  `anthropic/claude-3.7-sonnet` · 18.432 · US$ 0,03.
- Key-applied toast: "Chave aplicada no MedLens (…9f3c). Se ela vale na
  OpenRouter, só a próxima leitura prova." (fictitious hint).
- Missing-capability variant: "Este MedLens não tem: adapters.prompt".

### Design opportunities

1. **Say why the switch is disabled** when no folder is set: the fix is the
   empty "Pasta do projeto MedLens" just below.
2. **Give each gesture its own sign of life.** Today one in-flight state drives
   both the switch spinner and the "Emitindo…" button.
3. **A read-only "Auditoria de manuscritos" row.** MedLens has a third model
   the app does not configure (`model_audit`); Bridgr's memory lists a
   read-only row as backlog (not built). A quiet line under the model pair
   would keep it from surprising the operator.
4. **The Gasto on its own screen**, or on the dashboard's richer spend panel.

### Open questions

- The Extrações rows follow the order MedLens returns; whether that is newest
  first was not verified.

---

## Section 5 — Fantasma (`?section=fantasma`)

### The question

Is the robot's session up and logged into the ERP, is it working right now,
and with which folders and credentials?

### Where it sits in the work

The Fantasma types approved bills into the Everest ([Faturamento](04-billing.md))
and answers the read-only SQL ([Consulta](06-query.md),
[Catálogos](08-catalogs.md)). It is born in the wizard's Fantasma step and
maintained here. The status bar's Fantasma block and the
`fantasma_autostart_failed` notice open this section.

### Anatomy

**A. Panel** (shared with the wizard)

1. **Session card**:
   - icon robot;
   - title "Sessão" + " · <state>" + version chip `v<version>` (the project's
     version, read from its own files);
   - one sub-line: the state sentence, the busy line or the release countdown;
   - the switch at the right.
2. **Take-down confirmation**: an inline box under the card with an amber
   border, shown only after the switch was turned off. It holds the text and
   two right-aligned buttons, "Deixar de pé" · "Derrubar agora" (danger on
   hover). **[functional]** It sits in place, under the control that asked
   for it.
3. **Failure strip.**

**B. Fields** (shared with the wizard)

1. FolderField "Pasta do projeto"
   - placeholder `~/projetos/fantasma`;
   - note "É aqui que mora o .env que o docker compose lê."
2. FolderField "Pasta-fila"
   - placeholder `~/Bridgr/fila`;
   - note "Aprovar grava o JSON da conta aqui."
3. Pair: "Usuário do RDP" · "Usuário do Everest" (mono text, no placeholder).
4. Pair of SecretFields: "Credencial do RDP" · "Credencial do Everest" (no
   hint, no note of their own), with **one shared note under the pair**: "Em
   branco significa não mexi — nunca apague."
5. Wallet failure strip "O cofre recusou: <reason>".

### Data shown

Session label, sub-line and tone (busy overrides the label):

| Condition | Label | Tone | Sub-line |
|---|---|---|---|
| never asked | "…" | idle | (empty) |
| off, not fully configured | "parado" | idle | "Preencha pastas, usuários e senhas para poder ligar." |
| off | "parado" | idle | "O robô não está rodando nesta máquina. Ligar sobe o container e entra no Everest." |
| starting | "subindo…" | accent icon | "Subindo o container e abrindo a sessão — pode levar um tempo." |
| up: container running, ERP not logged in | "sem sessão" | amber border and icon | "Container de pé, mas o Everest não está logado — cada comando pediria login." |
| ready | "pronto" | green icon | "Container de pé e Everest logado. As mãos estão prontas." |
| shutting down | "desligando…" | accent icon | "Derrubando o container…" |
| foreign | "de fora do app" | idle | "Este Fantasma foi ligado por outro programa — usar, sim; derrubar daqui, não." |
| busy (in any state) | "ocupado" | accent icon | "Trabalhando agora — nenhum gesto passa até terminar." |
| release window after a take-down | (the state's label) | — | "O Everest ainda segura a sessão anterior — <n>s para liberar." (counts down; seconds rounded up) |

- Sub-line priority: countdown, then busy, then the state sentence.
- "Fully configured" = both folders, both user names and both passwords stored.

### States

The switch:

- is on for ready, up and foreign;
- spins while starting or stopping, whether this window started the gesture or
  the state says so (a rise started by the boot task also spins);
- is disabled when off and not startable (not configured, inside the release
  window, or busy), and when on but not stoppable (foreign, or busy);
- shows a hover reason only when foreign (the foreign sentence) or
  unconfigured ("Preencha pastas, usuários e senhas para poder ligar."). The
  busy and release reasons live in the sub-line.

While the confirmation is open, the switch **stays on** until it is answered.
Polling runs every 10 s, and every 2 s while starting, shutting down or
counting the release window.

### Actions

| Gesture | Precondition | Effect | Confirmation | Reversible | Feedback |
|---|---|---|---|---|---|
| Switch on | fully configured, off, release window over, not busy | writes the robot's `.env`, raises the container, claims it, logs into the ERP, verifies the remote-desktop window | none | only through Derrubar (costs 2 min) | spinner ("Ligando…"); toast "Fantasma pronto."; refusal in the strip |
| Switch off | ready or up, started by this app, not busy | opens the inline confirmation; **nothing stops yet** | — | — | the box |
| "Deixar de pé" | confirmation open | closes the box | — | — | — |
| "Derrubar agora" | confirmation open | takes the container down and starts the 2-minute release window | this button is the confirmation | no: the ERP holds the session about 2 min | spinner ("Derrubando…"); toast "Fantasma derrubado."; then the countdown |
| Edit folders or user names | — | stored | — | yes | silent |
| Type a password | non-empty | wallet | — | overwrite only | badge "gravada" |

Confirmation text (verbatim): "Derrubar mata a sessão do Everest, e ele leva
cerca de 2 minutos para soltá-la — religar antes falha com “sessão em uso”.
Nada que já foi lançado se desfaz."

Refusals the strip can show (verbatim; the service mixes languages):

- "Fantasma: the Fantasma is not fully configured"
- "Fantasma: <folder> does not look like the Fantasma project (no compose.yaml)"
- "Fantasma: the RDP and Everest passwords must be in the wallet first"
- "Fantasma: o Fantasma está ocupado — aguarde o gesto atual terminar"
- "Fantasma: este Fantasma foi ligado por outro programa — derrubá-lo mataria a
  sessão do Everest de quem o iniciou"
- "Fantasma: login ran but the FreeRDP window never appeared"

### Business rules

1. **Asymmetric lifecycle** (Bridgr decision 012). The app may raise the
   robot (at boot only if the Sobre switch allows it, off by default), but only
   the operator's explicit Derrubar takes it down. Closing the app never does.
2. **Derrubar names its cost.** The ERP releases the session about 120 s after
   the container dies, and starting inside that window fails with a symptom that
   never names the cause. So the window is stored and counted down, and the
   switch cannot turn on until it ends.
3. **Busy overrides the label**: "pronto" and "trabalhando agora" are different
   facts, and no lifecycle gesture passes while the robot works.
4. **A foreign container can be used, never stopped**: taking it down would kill
   somebody else's ERP session.
5. **"Logged in" is verified, never remembered.** The "sem sessão" state exists
   because the session can drop on its own.
6. User names are configuration. Passwords live in the wallet and are written
   into the robot's `.env` only when the app raises it.

### Must survive the redesign

The inline two-minute confirmation (never a one-click stop); the countdown;
busy overriding the label; the middle states (up without a session, foreign);
the switch staying on until the confirmation is answered; the sign of life on
the control.

### Free to redesign

Card layout; the countdown's shape. The confirmation must stay inline to the
gesture ([02-invariants.md](../02-invariants.md) § 13).

### Sample content for mockups

- "Sessão · pronto" `v2.3.0` (fictitious).
- Busy variant: "Sessão · ocupado" with "Trabalhando agora — nenhum gesto passa
  até terminar."
- Release variant: "O Everest ainda segura a sessão anterior — 87s para
  liberar."
- Folders `~/Projetos/fantasma` and `/srv/fila-fantasma`; users `camila.rocha`
  and `camila.rocha`; both credentials stored.

### Design opportunities

1. **Use the house confirmation idiom** ([05-components.md](../05-components.md),
   ConfirmButton) instead of this one-off amber box; Bridgr decision 006
   prefers one confirmation idiom app-wide.
2. **Put the countdown on the switch itself** (for example a draining ring), so
   the reason it will not move is on the control.
3. **One "busy" tone for all consoles**: today starting is amber on MedLens and
   accent on the Fantasma.

### Open questions

- In "sem sessão" (container up, ERP not logged in) the only gesture offered is
  Derrubar; there is no "log in again". Whether that is intended was not
  verified.

---

## Section 6 — Spook (`?section=spook`)

### The question

Is the portal service reachable, does it accept this app, and can it **write**
(send invoices)? With which portal login, for which OPMENEXO clients, and
re-read how often?

### Where it sits in the work

The Spook automates the hospitals' purchasing portal, OPMENEXO.
[OPMENEXO](13-opmenexo.md) and its order dialog depend on it, and point here
when it is missing. Its wizard step may be left blank, because the portals are
outside the conveyor. The clients block and the auto-read clock exist **only
here**.

### Anatomy

**A. Panel** (shared with the wizard's Spook step)

1. **Service card**: icon plugs-connected; "Serviço · <state>" + version chip;
   sub-line "O Spook responde onde este endereço apontar."; when reachable, a
   second sub-line "<n> capacidades"; the switch at the right.
2. **Credential card**: icon key; "Credencial" and the state sentence; a
   button reading "Emitir", "Reemitir" or "Emitindo…".
3. **Failure strip**: the last gesture's refusal.

**B. Fields**

1. FolderField "Pasta do projeto Spook" (no placeholder), note "De onde o app
   sobe o serviço (uv run spook serve). É a pasta com o pyproject.toml e
   src/spook."
2. Text field "Endereço"
   - placeholder `http://127.0.0.1:8765`;
   - note "Onde o Spook atende, por exemplo http://127.0.0.1:8765. Vazio: a
     conexão não existe."
3. Pair: "Usuário do portal (OPMENEXO)" (text) · SecretField "Senha do
   portal", with one shared note under the pair: "O login do portal que o Spook
   usa — o que antes ia à mão no .env dele. A senha fica no cofre do sistema e
   entra no serviço quando o app o sobe; em branco significa que você não
   mexeu."
4. Failure strip (wallet refusals, shown without a prefix).

**C. Clients card**, set apart by a divider and extra space:

- icon users-three; title "OPMENEXO · Clientes e filtros";
- sub-line "O bloco de clientes do settings.toml do Spook, editado daqui —
  comentários incluídos.", or without a folder "Configure a pasta do projeto
  Spook acima para editar os clientes.";
- button "Editar", disabled without a folder.

**D. Clients dialog** (modal, about 760 px wide, at most 84 % of the window
height), opened by "Editar":

1. Header: title "OPMENEXO · Clientes e filtros", the card's sub-line, a close
   X ("Fechar").
2. **Auto-read clock**, a section above the editor with a divider under it:
   - the kicker "Releitura automática" (drawn uppercase, mono);
   - the label "A cada quantos minutos" with a short numeric input
     (placeholder "0"; empty when off);
   - the hint "A cada tantos minutos o app relê o portal pelo Spook e, se algo
     cresceu, o sino avisa. 0 desliga; o mínimo é 5 — cada leitura percorre as
     filas de todos os clientes. Pausa enquanto o app está trancado e nunca
     durante um envio."
3. Body, by state:
   - while reading: "Lendo o arquivo…";
   - after a failed read: the refusal and a footer with only "Recarregar";
   - once loaded: an optional line "O arquivo ainda não tem o bloco de
     clientes; o primeiro salvar cria os marcadores no fim dele.", then **the
     TOML editor**, then a refusal strip. **[functional]** The editor has line
     numbers, because refusals name lines. It has syntax colours, is ~340 px
     tall and can be pulled taller.
4. Footer, once loaded:
   - the note "Salvar valida e injeta o bloco no arquivo, entre os
     marcadores; o resto do settings.toml nunca é tocado. Vale na próxima
     leitura do portal — sem reiniciar o serviço.";
   - "Recarregar" (counter-clockwise arrow);
   - "Salvar" (floppy icon, accent), disabled until the text differs from the
     file's; while saving it reads "Salvando…" with a spinner.

Editor placeholder when empty (verbatim, except the flow id: the app's
placeholder carries an example id, replaced here by a fictitious one):

```
[default.portals.opmenexo.clients.hospital-exemplo]
company_id = 12345
# statuses = ["FA"]
# invoice_flows = [9999]
```

Editor colours:

- keys: accent;
- table headers: strong;
- strings: the ok role;
- numbers: the warn role;
- **comments: calm grey** (an annotation, welcome);
- invalid tokens: the danger role.

### Data shown

| Field | Meaning | Presentation |
|---|---|---|
| Service label | configured? reachable? can this app raise it? | "…" (never asked) · "sem endereço configurado" · "no ar" · "parado" (local, off) · "não respondeu" (remote and silent) |
| Tone | same order | idle · idle · green · idle · amber |
| Capabilities | the count the service reports | "<n> capacidades" |
| Credential | this app's token, checked by the service | "aceita pelo serviço" (green) · "aceita, mas só de leitura — emitida antes de o app enviar nota. Emita outra para poder enviar." (amber) · "nenhuma guardada" · "recusada pelo serviço" (danger) · "não deu para perguntar" |
| Auto-read minutes | an app preference | empty = 0 = off; 1 to 4 become 5; 5 and above kept as typed |
| Clients block | the raw text between the app's markers in the Spook's `settings.toml` | TOML, verbatim, comments included |

- "Configured" = a folder **or** an address is set.
- A blank address with a folder set means this app's own local Spook
  (`127.0.0.1:8765`).

### States

**The switch**

- On = reachable.
- On but not raised by this app: disabled, hover "de fora do app".
- Off and not raisable from here (no folder, or the address points
  elsewhere): disabled, no hover reason.
- Raisable but the portal login is missing: disabled, hover "Preencha o
  usuário e a senha do portal antes de ligar — o app sobe o Spook com esse
  login."
- Spinner while any Spook gesture is in flight.

**Credential button**: disabled unless the service can be raised from here.
During any Spook gesture it reads "Emitindo…" with a spinner.

**Clients card**: held without a folder.

**Dialog**

| State | What is drawn |
|---|---|
| Loading | "Lendo o arquivo…" |
| Read failed | the refusal **replaces** the editor; only "Recarregar" is offered |
| Block absent | the "ainda não tem o bloco" line above the editor |
| Loaded, unchanged | "Salvar" disabled |
| Edited | "Salvar" enabled |
| Saving | "Salvando…" with a spinner |
| Refused | the refusal strip; the typed text is kept |

### Actions

| Gesture | Precondition | Effect | Feedback |
|---|---|---|---|
| Switch on | local folder, portal user and stored portal password | raises the Spook with the portal login injected (never written to disk) | spinner; no toast; refusal in the strip |
| Switch off | raised by this app | stops it | spinner; no toast |
| "Emitir" / "Reemitir" | can be raised from here | issues this app's API token through the Spook's CLI, with portal read **and** write scopes; stores it in the wallet | "Emitindo…"; toast "Credencial emitida e guardada no cofre." |
| Edit folder, address or user | — | stored; the next poll re-reads them | silent |
| Type the portal password | non-empty | wallet | badge |
| "Editar" | folder set | opens the dialog and reads the block from the file **now** | "Lendo o arquivo…" |
| Minutes | dialog | stored immediately, independent of "Salvar" | silent |
| Type in the editor | loaded | local text; "Salvar" enables when it differs | — |
| "Salvar" or `Ctrl+S` | loaded (`Ctrl+S` saves even when nothing changed) | validated on the Rust side; the block is written between the markers, atomically | "Salvando…"; toast "Clientes salvos no settings.toml."; refusal verbatim in the strip |
| "Recarregar" | not saving | re-reads the file, discarding edits | — |
| Close (X, `Esc`, click outside) | always | closes; unsaved edits are dropped without warning | — |

Refusals the dialog can show (verbatim from the Rust side, in English):

- "the clients block was refused: " followed by the TOML parser's message, which
  names the line.
- "the clients block was refused: the block may only define
  [default.portals.opmenexo.clients.*] tables — found `default.api`"
- "the clients block was refused: client `santa-aurora`: `company_id` is
  required — the Spook silently skips a profile without it, and that client's
  work becomes invisible"
- The same lead, then one of:
  - "client `<name>`: `company_id` must be an integer"
  - "… `flows` must be an array of integers"
  - "… `statuses` must be an array of strings"
  - "… unknown field `invoice_flow` — the Spook reads company_id, flows,
    statuses and invoice_flows"
- "the clients block was refused: the marker lines belong to the file, not to
  the block"
- "Spook: the clients block markers in settings.toml are unbalanced — fix the
  file by hand before editing it from here" (on reading as well as saving)
- a filesystem refusal naming `settings.toml` when the file cannot be read.

Service refusals in the panel's strip:

- "Spook: the OPMENEXO portal credentials are not set — Settings · Spook"
- "Spook: the configured address is not this machine's — nothing to raise"
- "Spook: this Spook was not started by this app — it stays up"
- "Spook: the folder does not look like a Spook project (pyproject.toml +
  src/spook)"

### Business rules

1. **Four facts, four sentences.** "Not configured", "not answering",
   "credential missing" and "credential refused" are different problems. An
   unreachable service is never drawn as a broken credential. A local Spook
   that is off is a calm "parado"; only a remote one that does not answer
   warns.
2. **An accepted credential may still be read-only.** The scope is minted into
   the token, so a token issued before the app could send invoices can only
   ever read. The cure is reissuing, and that button sits beside the sentence.
   Empty scopes mean "unknown", and then the screen asserts nothing.
3. **The app never raises a Spook it cannot sign in with**: up but unable to
   reach the portal is worse than not up. A Spook raised elsewhere is exempt; it
   reads its own settings.
4. **The portal password is injected at start-up and never written to disk.**
5. **The truth stays in the file** (Bridgr decision 027):
   - the dialog never shows a kept copy;
   - a failed read replaces the editor, so a save cannot overwrite a block the
     app could not show;
   - the app edits only the region between its markers, and the rest of
     `settings.toml` (hand-written lessons) is never touched.
6. **The warning is not the defence.** The block is refused before a byte is
   written in these cases:
   - bad TOML;
   - a table outside the clients subtree;
   - a client without an integer `company_id` (the Spook silently skips such
     a client);
   - an unknown field (the Spook ignores misspellings);
   - a filter of the wrong shape.

   An empty block is valid.
7. **Saved means in force on the next portal read**, with no restart.
8. **The auto-read clock is an app preference: off by default, 5 minutes at
   least** (Bridgr decision 032). Below that, the portal is hammered for no
   news a human can act on. It pauses while the app is locked and never runs
   during a send.

### Must survive the redesign

- The four-way service vocabulary.
- The read-only credential warning, with the reissue gesture beside it.
- The portal-login gate on raising, with its hover reason.
- The dialog reading the file when it opens; a failed read replacing the
  editor.
- Refusals verbatim, with the typed text kept.
- "Salvar" held until the text differs.
- The clock's 5-minute floor and "0 desliga".

### Free to redesign

Editor chrome and dialog size. The clock's placement is free too, but keep it
with the portal, not beside the address: it already moved once at the
operator's request.

### Sample content for mockups

- Service "no ar" `v0.4.2` (fictitious), "15 capacidades"; credential "aceita
  pelo serviço" with "Reemitir"; also draw the read-only variant.
- Folder `~/Projetos/spook`; address empty; user `camila.rocha`; password
  stored. Clock: 15.
- Editor (fictitious ids):

```
# Hospital Vale Verde — só notas aguardando NF
[default.portals.opmenexo.clients.vale-verde]
company_id = 71001
statuses = ["FA", "SP"]
invoice_flows = [5120]

[default.portals.opmenexo.clients.santa-aurora]
company_id = 71002
flows = [41]
```

- Refused variant: remove `company_id` from `santa-aurora`; the strip shows
  the `company_id` sentence above.

### Design opportunities

1. **Ask before discarding edits.** `Esc` and a click outside close the dialog
   and silently drop unsaved TOML. An inline "descartar alterações?" when the
   text differs would protect the work ([02-invariants.md](../02-invariants.md)
   § 13 prefers inline confirmation).
2. **Frame the refusals in pt-BR**: a Portuguese lead with the original text
   kept verbatim beneath it. This needs code.
3. **Correct the address note.** "Vazio: a conexão não existe." is only true
   without a folder; with a folder, blank means the Spook this app raises at
   `127.0.0.1:8765`. Any new wording is a proposal.
4. **Summarise the block on the card** (for example the number of clients
   configured). This needs a parse, so it is a proposal.

### Open questions

- Should closing a dialog with unsaved edits ask first? Not decided; today it
  does not.

---

## Section 7 — NFe Pack (`?section=nfepack`)

### The question

Is the NF-e monitor running and actually completing cycles, and with which
folders, pause and portal login?

### Where it sits in the work

The NFePackMonitor downloads the distributor's NF-e and DANFEs from the ACOM
portal. [NFePackMonitor](12-nfe-pack-monitor.md) is its console (watch, plus
the on/off switch); this section configures it. The status bar's
NFePackMonitor block and the `nfepack_autostart_failed` notice open this
section.

### Anatomy

**A. Service card**:

- icon receipt; "Serviço · <state>" (no version chip);
- sub-line "As NF-e do portal ACOM — sobe com o app e fica no ar; só
  Desligar derruba";
- sub-line "Último ciclo: <heartbeat line>";
- the switch at the right;
- the failure strip under the card.

**B. Fields**

1. FolderField "Pasta do projeto NFePackMonitor", note "De onde o app sobe o
   serviço (uv run scrape). É a pasta com o pyproject.toml e
   src/nfepackmonitor."
2. One row, **2/3 + 1/3** (the operator's own layout):
   - FolderField "Pasta das notas", note "Onde as NF-e e DANFEs baixadas
     ficam salvas. Em branco: o padrão do monitor (storage/ dentro do
     projeto). O log não tem campo — o app lê o padrão e mostra em Logs.";
   - number field "Intervalo entre ciclos (s)", empty when 0, note "Pausa
     entre um ciclo e o próximo. Em branco: o padrão do settings.toml do
     monitor (hoje 10 s)."
3. Pair: "Usuário do portal (ACOM)" · SecretField "Senha do portal", with the
   shared note "O login do webmonitor que o NFePackMonitor usa. A senha fica
   no cofre do sistema e entra no serviço quando o app o sobe; em branco
   significa que você não mexeu — e campos vazios deixam o .env do próprio
   monitor valendo."
4. Failure strip (wallet refusals).

**C. Métricas card** (the same card appears on the console screen):

- title "Métricas", with the stamp "exportadas em dd/mm/aaaa · hh:mm" when
  known;
- either "Nenhuma exportação ainda — o monitor grava as métricas a cada 10
  ciclos bons." or six tiles, each a mono number with its label under it:
  - "ciclos"
  - "bem-sucedidos"
  - "falhas" (the number in amber when above 0)
  - "notas baixadas"
  - "duração do último ciclo" (shown as "<n> s")
  - "notas no último ciclo"

### Data shown

| Field | Meaning | Presentation |
|---|---|---|
| State | read from the process table (the monitor has no port) | "…" · "sem pasta configurada" · "baixando" (started by this app, green) · "no ar — subiu fora deste app" (foreign, green) · "parado" (idle) |
| Último ciclo | the monitor's heartbeat, written after each successful cycle | "nenhum ciclo registrado ainda", or "10/09/2026 · 14:31 · 3 notas no ciclo · 1284 ciclos no total" (each piece only when present; numbers without a thousands separator) |
| Metrics | counters the monitor exports every 10 good cycles | pt-BR thousands; a counter missing from the file is "—", never 0 |
| Stamp | when that export was written | "exportadas em …" |

### States

The switch:

- is on while the monitor runs;
- when running but foreign: disabled, hover "de fora do app — quem subiu foi
  outro programa; usar, sim; derrubar daqui, não";
- when off and not raisable (for example no folder): disabled, no hover
  reason;
- spins during a gesture.

Metrics: never exported; partial (dashes); full.

### Actions

| Gesture | Precondition | Effect | Feedback |
|---|---|---|---|
| Switch on | folder set, not running | raises the monitor with the typed values injected (notes folder, pause, user, the password from the wallet) | spinner; no toast; refusal in the strip |
| Switch off | started by this app | stops it (the only way down: otherwise it keeps downloading after the app closes) | spinner; no toast |
| Edit fields | — | stored; they take effect at the next rise | silent |
| Pause field | — | blank, 0 or anything not numeric is stored as 0, meaning the monitor's default | silent |

Refusals the strip can show:

- "NFePackMonitor: no NFePackMonitor project folder configured"
- "NFePackMonitor: the folder does not look like an NFePackMonitor project
  (pyproject.toml + src/nfepackmonitor)"
- "NFePackMonitor: this monitor was not started by this app — it stays up"

### Business rules

1. **Asymmetric lifecycle** (Bridgr decision 033):
   - it rises with the app when configured, and does **not** stop when the app
     closes;
   - the switch is the only way down;
   - a foreign instance is used, never stopped, and nothing is started beside
     it, because two scrapers would race one portal and one storage.
2. **A live process is not a working portal.** A failed login leaves the
   process up and the cycles empty. That is why the heartbeat line (which stops
   moving) is shown. The screen never interprets the monitor's log.
3. **Blank fields leave the monitor's own settings in charge**, so nothing here
   gates the switch on the credentials (unlike the Spook).
4. **A typo can never become a zero-second loop** against the portal.
5. **The export stamp is part of the answer**, and a counter missing from the
   file is a dash.
6. **The logs folder has no field** on purpose; [Logs](11-logs.md) reads the
   default.

### Must survive the redesign

Foreign shown on and locked; the heartbeat line; the stamp on the metrics;
dashes for unknown counters; "blank = the monitor's own default" said on each
field.

### Free to redesign

Tiles or a table. The row proportion was the operator's call: keep the long
folder and the short number together.

### Sample content for mockups

- "Serviço · baixando"; "Último ciclo: 10/09/2026 · 14:31 · 3 notas no ciclo ·
  1284 ciclos no total".
- Folder `~/Projetos/nfepackmonitor`; notes folder and pause empty; user
  `camila.rocha`; password stored.
- Metrics, "exportadas em 10/09/2026 · 14:20": ciclos 1.284 · bem-sucedidos
  1.279 · falhas 5 · notas baixadas 3.912 · duração do último ciclo 42 s · notas
  no último ciclo 3.

### Design opportunities

1. **Show the heartbeat's age** ("há 3 min") and warn when the last good cycle
   is much older than the pause. That makes the trap of a live process with a
   failing login visible, without reading the log.
2. **One number format**: the heartbeat line shows "1284" where the tiles show
   "1.284".

### Open questions

None beyond the above.

---

## Section 8 — Adapters (`?section=adapters`)

### The question

Which clients appear in Triagem's menu, and does each client have a complete
colinha, the order header the robot needs?

### Where it sits in the work

- The list comes from MedLens: the adapters it can process.
- Which of them appear filters [Triagem](02-triage.md)'s client menus.
- The colinha feeds the robot and the Revisão's completeness gate:
  [Revisão](03-review.md) names this section when a colinha is missing.

### Anatomy

1. Intro paragraph (verbatim): "O catálogo é do MedLens — esta lista é o que
   ele diz saber processar. A caixa escolhe quais aparecem no menu da Triagem;
   desligar não apaga extração: o que já foi extraído continua na Revisão."
2. The empty sentence, or the list.
3. **Adapter rows**, stacked. Each row holds:
   - a checkbox: checked = shown in Triagem; accessible name "Mostrar <cliente>";
   - the adapter's name (its label, or its id when there is none);
   - a meta line under the name: the id in mono, plus " · colinha: falta
     <fields>" only when the colinha is incomplete;
   - a **pill**;
   - a "Colinha" button with a clipboard icon.
4. **Colinha dialog** (modal, about 640 px wide):
   - header "Colinha · <cliente>";
   - sub-line "O que o Fantasma precisa saber para trabalhar uma conta deste
     cliente. É o padrão do adapter — a conta pode divergir dele sem mudá-lo.";
   - the id in mono;
   - body: **eight registry-code fields in the ERP's own layout**, then the
     explanation paragraph, then the notices. **[functional]** Each field is a
     right-aligned label column, one short code field, and the registry name
     beside the code. The operator knows these fields from the ERP's screen,
     and a second grammar would force a translation at every filling;
   - footer: "Cancelar" · "Gravar" (floppy icon).

| Field (label) | Catalogue searched | Placeholder (catalogue collected) |
|---|---|---|
| Depósito | Depósitos | "código ou nome…" |
| Cliente | Locais de uso | "código ou nome…" |
| Comissionado | Comissionados | "código ou nome…" |
| Tipo Pedido | Tipos de pedido | "código ou nome…" |
| Tabela | Tabelas de preço | `015` |
| Depósito do pedido | Depósitos | `2` |
| Cond. de Venda | Condições de venda | "código ou nome…" |
| Local de Uso | Locais de uso | "código ou nome…" |

When a field's catalogue was never collected, its placeholder reads "colete o
catálogo: <Catálogo>" (for example "colete o catálogo: Comissionados").

Explanation (verbatim): "O depósito é a ponte: o app conhece o adapter, o
Fantasma pensa em depósito. O cliente é para quem a conta é faturada; o local
de uso é onde o material foi usado, normalmente o próprio depósito. O depósito
do pedido não é nenhum dos dois: é o estoque consignado, 2 em todos os
conhecidos. O portador não aparece aqui porque é sempre 100 (carteira) — está
fixo no Fantasma."

### Data shown

Pill, computed from the colinha file over **all eight** fields:

| Pill | Condition | Tone |
|---|---|---|
| "colinha completa" | all eight filled | green |
| "colinha incompleta" | some filled, some missing | amber |
| "colinha não montada" | none filled, or no colinha at all | quiet grey: absence, never a warning |

The meta line lists the missing fields in the order of the table above, and
only for "incompleta".

Registry-code field:

- Out of focus, it shows `<code> · <registry name>` (for example `4 ·
  CONSIGNADO VALE VERDE`). When the catalogue has no name for the code, it
  shows the raw code in mono. The full text is also the hover tooltip.
- In focus, it shows the raw code, selected. Typing searches code, name and
  alias, with up to 8 hits in a floating list (code in accent mono, then the
  name).

### States

- MedLens has not answered yet: only the intro.
- No adapters (MedLens down, it answered none, or the request failed):
  "Nenhum adapter — o MedLens precisa estar no ar para responder a lista.
  Ligue-o em MedLens → Serviço." The list is requested once, when the section
  opens.
- List loaded: an adapter whose colinha is missing on disk reads "não
  montada".

Each dialog field is one of:

- empty;
- filled and known;
- **unknown code** (catalogue collected, code not in it): amber border, and
  the line "não existe no catálogo de <Catálogo>" under the field;
- filled with the catalogue never collected: it saves, but nobody checked it.

Dialog notices, in this order:

1. If any code is unknown, a danger notice: "<fields>: código que não existe
   no catálogo. Isto vira cabeçalho de pedido no ERP — um código inventado não
   falha na hora, fatura errado, e o erro aparece longe. Corrija ou apague antes
   de gravar." (the field names in bold).
2. Otherwise, if any filled field's catalogue was never collected, an amber
   notice: "<fields>: preenchido, mas sem catálogo coletado — ninguém conferiu
   esses códigos. Colete em Catálogos para o app conferir por você."
3. Whenever Depósito is empty, an amber notice: "Sem o depósito esta colinha
   fica gravada aqui, mas não vai para o Fantasma: é por ele que o robô liga o
   informe ao cliente."
4. After a failed write, a danger notice with the refusal; the typed values are
   kept.

While saving, "Gravar" reads "Gravando…" and "Cancelar" is disabled.

### Actions

| Gesture | Precondition | Effect | Feedback |
|---|---|---|---|
| Tick or untick a row | — | shows or hides the adapter in Triagem's menus, stored immediately; extractions are untouched | silent |
| "Colinha" | — | opens the dialog on a local copy of the stored colinha | — |
| Type or pick a code | dialog | ↑ ↓ move through the hits, Enter picks, Esc gives up and restores the value, leaving the field commits what was typed | the name appears beside the code, or the unknown warning |
| "Gravar" | no unknown code | stores the colinha and regenerates the file the robot reads | "Gravando…"; toast "Colinha gravada — o arquivo do Fantasma foi regenerado."; the dialog closes |
| "Gravar" disabled | an unknown code | tooltip "Há código que não existe no catálogo — enviar errado é pior que não enviar" (when enabled: "Grava a colinha e regenera o arquivo que o Fantasma lê") | — |
| "Cancelar", or a click outside | — | closes and drops the edits | — |

### Business rules

1. **Codes come from the catalogues, not from memory**: pick by registry
   name, and the code comes along. What is stored is always the code.
2. **An unknown code blocks the save.** What leaves here becomes an order
   header in the ERP, and an invented code does not fail at the time: it
   invoices wrongly, far away. **Only what could be checked counts**: with no
   collected catalogue, the field saves and the screen says nobody checked.
3. **The price table is text**, because `015` is not `15`. Every other field
   is a number.
4. **Empty means "unknown", never zero.** The robot falls back to its own
   default for a missing field.
5. **No deposit, no entry for the robot.** The colinha is kept in the app but
   not handed to the robot, because the deposit is the bridge between the
   adapter and the robot's world. The robot's file is also written only when
   the Fantasma project folder is set and looks like the robot's project;
   otherwise that write is skipped silently (`cheatsheet.rs`).
6. **Insurer and doctor are not here.** They are matched in Revisão, with the
   bill in front.
7. **The bearer (portador) is not a field**: it is always 100, fixed in the
   robot.
8. **A write failure keeps what was typed**: the values are discovery work.
9. **Hiding an adapter only filters the choice.** The stored list is the
   hidden ones, so a new adapter shows up on its own.
10. **The list and the colinhas are read separately**, so a MedLens that is
    down never stops the colinhas on disk from drawing.

### Must survive the redesign

- The ERP-shaped field layout (label · code · name).
- The blocking unknown-code rule and its sentence, plus the no-catalogue path
  that saves with the "ninguém conferiu" warning.
- `015` kept as text.
- The three pills, with "não montada" drawn as absence.
- The no-deposit warning.
- The typed values kept after a failure.

### Free to redesign

Row composition, pill shapes, dialog size.

### Sample content for mockups

Rows:

| Name | Id and meta line | Pill |
|---|---|---|
| Hospital Vale Verde | `vale_verde` | "colinha completa" |
| Hospital Santa Aurora | `santa_aurora` · "colinha: falta Comissionado, Cond. de Venda" | "colinha incompleta" |
| Instituto Ortopédico Horizonte | `horizonte` | "colinha não montada" |
| Hospital Infantil Sol Nascente (unticked) | `sol_nascente` | "colinha não montada" |

Colinha of Vale Verde, with fictitious registry rows. The colinha's numeric
fields hold plain numbers, so the cast's deposits `01`/`04` appear here as `1`
and `4`; only Tabela keeps its leading zero.

| Field | Shown |
|---|---|
| Depósito | `4 · CONSIGNADO VALE VERDE` |
| Cliente | `1204 · HOSPITAL VALE VERDE` |
| Comissionado | `37 · COMERCIAL EXEMPLO` |
| Tipo Pedido | `7 · CONSIGNAÇÃO OPME` |
| Tabela | `015 · TABELA OPME` |
| Depósito do pedido | `2 · CONSIGNADO` |
| Cond. de Venda | `30 · 30 DIAS` |
| Local de Uso | `1204 · HOSPITAL VALE VERDE` |

Unknown variant: Comissionado `9999`, with "não existe no catálogo de
Comissionados" and the blocking notice.

### Design opportunities

1. **Show where the colinha goes**, for example in the dialog footer: to the
   Fantasma folder, or kept in the app only when the Fantasma is not
   configured. Today the destination is invisible.
2. **Do not draw the missing pieces as if they existed.** The original design
   has a per-adapter Prompt editor and a "corrections → prompt" panel; both wait
   on MedLens data (Bridgr decision 021 § 4). The header hint still mentions
   "prompt de extração".
3. **`Esc` to close** the Colinha dialog. Today it closes only with Cancelar or
   a click outside; `Esc` inside a field only cancels the search.
4. **Refresh the list when MedLens comes up.** Today it is requested once, when
   the section opens.

### Open questions

- If the operator types text that is not a number into a numeric field and
  leaves without picking a hit, the field ends up empty. Should the text stay,
  marked instead?
- The pill counts all eight fields. What the Revisão gate requires of a
  colinha may be a subset: see [Revisão](03-review.md).

---

## Section 9 — Exportação (`?section=export`)

### The question

Not answered yet. The section exists because the subject exists; its hint
promises "Formato e planilha de registro".

### Where it sits in the work

Nowhere yet.

### Anatomy

An empty room, centred in the content column:

- the section's icon (export), large and faint;
- the hint "Formato e planilha de registro";
- a quiet line: "Esta tela ainda não foi construída."

It is drawn by the page itself, not by the shared empty-state component.

### Data shown

None.

### States

One state: the empty room.

### Actions

None.

### Business rules

An empty room that says so beats a form that saves nothing: a form that saves
nothing teaches that saving does nothing.

### Must survive the redesign

The honesty: it must not look configurable.

### Free to redesign

Everything.

### Sample content for mockups

None.

### Design opportunities

1. Draw it like the app's other unbuilt rooms (the shared empty-state
   treatment in [05-components.md](../05-components.md); the future studies in
   [14-future-studies.md](14-future-studies.md)), so all of them read alike.

### Open questions

- What "formato" and "planilha de registro" mean for this app. No code or
  decision defines it; Bridgr's screen inventory only repeats it. Whether it is
  meant to configure the grid's existing CSV export was not verified.

---

## Shared field primitives

The four service sections here and the [Primeira montagem](15-welcome.md)
wizard draw the **same components**, so what one writes the other shows.

| Component | In Settings | In the wizard |
|---|---|---|
| FolderField | Doca (2), MedLens, Fantasma (2), Spook, NFe Pack (2) | inside the MedLens, Fantasma and Spook fields |
| SecretField | MedLens (1), Fantasma (2), Spook (1), NFe Pack (1) | MedLens, Fantasma, Spook |
| ServiceToggle | Sobre (autostart, up to 4), and the MedLens, Fantasma, Spook and NFe Pack cards | the MedLens, Spook and Fantasma cards; also the [NFePackMonitor](12-nfe-pack-monitor.md) console |
| MedLens panel + fields | panel **above** fields | MedLens step: fields **above** panel |
| Fantasma panel + fields | panel above fields | Fantasma step: fields above panel |
| Spook panel + fields | Spook section, plus the clients card (Settings only) | Spook step (the connection only) |

### FolderField

**Anatomy.** A label; a row with a mono text input (placeholder optional) and
a button "Escolher" (folder-open icon); a note under it (always present).

**Behaviour.**

- Free text stays possible: a `~` path, or a path meant for another machine.
- Leaving the field, or Enter, writes the trimmed text immediately.
- "Escolher" opens the system's folder chooser, **titled with the field's
  label**. A picked path is written. Cancelling keeps what is there and never
  writes an empty value.
- A chooser failure shows a danger toast with the refusal.

**States.** Empty (placeholder showing); filled; hover; focus (accent border).

**Rules.**

- Every folder field in the app is this one component, so the shape cannot
  drift.
- The picker exists so a path cannot be mistyped, but typing must remain
  possible.
- The field validates nothing. Each service checks its folder when asked to
  rise, and a wrong folder is refused in the card's failure strip:

  | Service | The folder must contain |
  |---|---|
  | MedLens | `pyproject.toml` |
  | Spook | `pyproject.toml` and `src/spook` |
  | NFePackMonitor | `pyproject.toml` and `src/nfepackmonitor` |
  | Fantasma | `compose.yaml` or `docker-compose.yml` |

### SecretField

**Anatomy.** A label, plus a small quiet-accent badge "gravada" when a secret
is stored; a password input (mono, never autocompleted); an optional note.

| State | The input shows |
|---|---|
| nothing stored | empty; the format hint as placeholder if the field has one (only the OpenRouter key: `sk-or-v1-…`) |
| stored | empty, with `••••••••••` as a **placeholder** in firmer ink and letter spacing: it reads like a masked password but is not a value |
| typing | the typed characters, masked |
| just saved | empties itself; the badge appears or stays |
| wallet refused | the section's failure strip (below) |

**Behaviour.**

- It commits when the operator leaves the field or presses Enter. An empty
  field writes nothing, ever.
- The value goes to the system wallet and is cleared from the input as soon as
  the wallet accepts it.
- Whether a secret is stored is known app-wide: a secret typed in the wizard
  shows "gravada" here at once.

**Rules.**

- No value ever comes back to the screen, and blank means "não mexi".
- The dots are a placeholder, so nothing dummy can be saved.
- There is **no erase gesture**: a stored secret can only be overwritten.
- Only five secrets are typeable (OpenRouter key, RDP, Everest, OPMENEXO
  password, ACOM password).

The failure wording differs by section:

- MedLens and Fantasma show "O cofre recusou: <reason>", which also covers
  "the wallet could not even be read".
- Spook and NFe Pack show the bare reason.
- The reason is the Rust side's text, which itself begins "the system wallet
  refused: …".

### ServiceToggle

**Anatomy.** A switch (track about 38 × 22 px, round thumb). On = the track in
the **ok (green)** role, the same green as the "no ar" state. Off = a neutral
track.

| State | Drawn |
|---|---|
| off / on | thumb left / right |
| busy | the thumb becomes a spinning notch; locked; the accessible name becomes the busy label ("Ligando…", "Desligando…", "Derrubando…") |
| disabled | half opacity; the reason as a hover tooltip **when one is given** |
| focus | accent ring |

**Behaviour.** The switch is **controlled**: it shows the service's state, not
the click. Flipping it only sends the request and snaps back; the switch moves
when the service actually moves. For the Fantasma, "off" opens the
confirmation, and the switch stays on meanwhile.

**Rules.**

- The sign of life is on the control itself, before the command answers.
- A foreign service shows on and locked, with the reason on hover.
- The switch replaced the original design's Ligar/Desligar buttons at the
  operator's request (Bridgr decision 006).
- On Sobre, the same switch shows a **stored preference** (autostart), not a
  service state (Section 1, design opportunity 1).

### Service and credential cards

One card layout for every console: a single row of icon · words · gesture.
The words are the title ("Serviço", "Sessão" or "Credencial") plus " · <state>"
in regular weight, an optional version chip, and one or two sub-lines. The
gesture at the right is a switch or a button.

| Tone | Icon | Card border |
|---|---|---|
| idle (stopped, absent, never asked) | neutral grey | normal |
| ok (up, accepted) | green | normal |
| starting | amber on MedLens; accent on Fantasma | normal |
| warn (Spook remote and silent; Spook read-only credential; Fantasma "sem sessão") | amber | amber only on the Fantasma's "sem sessão" |
| danger (credential refused) | red | red-tinted |

- Absence is drawn calm (grey), never in accent.
- Green appears only after the service answered. "Never asked" shows "…" and
  asserts nothing.
- **Failure strip**: under the cards, a small danger-tinted strip with the last
  gesture's refusal, verbatim. It clears when the next gesture starts.
- NFe Pack has no version chip.
