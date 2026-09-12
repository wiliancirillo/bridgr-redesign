# The shell — everything around the screens

> Where: every route (reduced in the first-assembly wizard, replaced under the lock) · Status: built · Code: `src/routes/+layout.svelte`, `src/routes/+layout.ts`, `src/routes/+page.ts`, `src/lib/components/Titlebar.svelte`, `Sidebar.svelte`, `StatusBar.svelte`, `Toasts.svelte`, `Logo.svelte`, `Avatar.svelte`, `NoticeCenter.svelte` (placement only), `src/lib/nav.ts`, `src/lib/service-blocks.ts`, `src/lib/toast.svelte.ts`, `src/lib/window.svelte.ts`, `src/lib/lock.svelte.ts`, `src/lib/preferences.svelte.ts`, `src/lib/theme.svelte.ts`, `src/app.css`, `src-tauri/src/frame.rs`, `src-tauri/tauri.conf.json`

## The question

Without entering any screen, the frame answers three things: **where am I**
(the one lit sidebar line and the titlebar's "· context"), **who is at the
bench** (avatar and first name), and **what are the services doing right now**
(the status bar). It also carries the app's two voices: the bell — subjects
waiting for a human — and toasts — feedback of an act just done
([02-invariants.md](02-invariants.md) § 14).

## Where it sits in the work

Present on every route. Two situations reduce it:

| Piece                                              | Normal | Wizard `/welcome`                 | Locked window                            |
| -------------------------------------------------- | ------ | --------------------------------- | ---------------------------------------- |
| Titlebar: logo, name, "· context", window controls | ✓      | ✓ (context "· Primeira montagem") | ✓ (context of the route behind the lock) |
| Search field                                       | ✓      | absent                            | absent                                   |
| Bell                                               | ✓      | absent                            | drawn, disabled                          |
| User button and menu                               | ✓      | absent                            | drawn, disabled                          |
| Sidebar                                            | ✓      | absent                            | absent (the whole body is replaced)      |
| Status bar                                         | ✓      | absent                            | absent                                   |
| Toasts                                             | ✓      | ✓                                 | absent                                   |

The wizard is in [screens/15-welcome.md](screens/15-welcome.md); the locked
window in [screens/16-profile-and-lock.md](screens/16-profile-and-lock.md).

## Anatomy

Reading order: window frame → titlebar → sidebar | screen → status bar, with
toasts floating over the bottom-right corner.

### 1. The window frame

- No system title bar: the app draws its own titlebar in HTML. The desktop
  (GTK, client-side decorations) draws the window's shadow — never the app
  ([03-platform.md](03-platform.md) § The window).
- The app's root surface paints the window: `--color-bg` ground, a 1 px
  `--color-divider` hairline all around, corners rounded 8 px
  (`--radius-md`). **Maximized**, border and radius go to zero — the window is
  flush with the screen edges. GTK squares its own decoration when maximized,
  tiled or fullscreen; the page itself is told only "maximized".
- Default window 1440 × 900, minimum 1024 × 768, resizable.
- **[functional]** The screen area scrolls vertically only. Wide content (a
  grid with dozens of columns) scrolls inside its own box; the page never shows
  a horizontal scrollbar.

### 2. The titlebar (38 px; 32 px in compact density)

One row, bottom hairline, left to right:

1. **Brand** — the mark (22 × 24 px: an arch over a base line, stroked in the
   accent, with a four-point sparkle at the top right in `--brand-spark`, an
   orange that stays the same in both themes — brand, not state); the name
   **"Bridgr Pro"** (13.5 px, weight 500); then **"· {context}"** in muted ink
   (11 px, 45 %), ellipsized. The context is the label of the destination the
   route belongs to — the longest matching route wins: "· Revisão" on
   `/review/…`, "· Relatórios" on `/sheets/…`, "· Configurações" on every
   Settings section (the section is not named), "· Perfil", "· Avisos",
   "· Busca", "· Primeira montagem". An unknown route shows no context. The
   window title follows the same rule: "Bridgr Pro · Revisão", or just
   "Bridgr Pro".
2. **Centre slot** — flexible and centred: the global **search field**, at most
   430 px wide: magnifier icon, placeholder "Buscar paciente, atendimento,
   arquivo ou código do ERP…", and a `kbd` hint **"Ctrl K"** that disappears
   while the field has focus (the border turns accent-tinted). Behaviour and the
   results screen: [screens/18-search.md](screens/18-search.md).
3. **Bell** — a 27 × 25 ghost button (`ph-bell`), tooltip and name "Central de
   avisos"; a small red pill with the unread count, ringed in the bar's ground,
   when there is anything unread. It opens a 352 px panel anchored under it,
   right-aligned. The panel, what the badge counts and `/notices`:
   [screens/17-notices.md](screens/17-notices.md).
4. **User button** — avatar (19 px) + the profile's **first name** (first word;
   about 12 characters, then ellipsis) + a caret (down; up while open). No name
   yet → "Sem nome" in quiet ink and a generic person glyph. Accessible name
   "Menu do usuário".
   **User menu** (250 px, surface, large elevation, anchored under the button,
   right-aligned):
   - Header: avatar (30 px), full name (or "Sem nome"), the role underneath when
     one is set.
   - Destinations (icon + label): **"Perfil"** (`ph-user-circle`) → `/profile`;
     **"Preferências"** (`ph-sliders-horizontal`) → Configurações › Aparência;
     **"Configurações"** (`ph-gear-six`) → Configurações (opens on Sobre).
   - A separator, then the way out (`ph-sign-out`): **"Encerrar sessão"** while
     a PIN is stored — it locks the window and stays; **"Encerrar o app"** when
     no PIN exists — it closes the app.
5. **Window controls** — three 26 × 22 ghost buttons: "Minimizar" (`ph-minus`);
   "Maximizar" / "Restaurar" (`ph-square` / `ph-corners-in` — icon and name say
   where the click goes, not what the button is); "Fechar" (`ph-x`, the only
   control that hovers into the danger role).

- **[functional]** The bar is the only drag handle for the window: the bar
  itself and every non-interactive piece of it (mark, name, context, the empty
  space of the centre slot) move the window. The field, bell, user button and
  controls are not handles.

**The face** (`Avatar.svelte`, drawn at every size above and on the Perfil and
the locked window): the operator's **photo** when the avatar mode is "photo"
and a photo is stored; a **drawn DiceBear style** when one is chosen; the
**initials** otherwise (first and last word of the name — "Camila Duarte
Rocha" → CR); a generic person glyph when there is no name. Never invented
letters. Today: a circle on `--color-accent-900` with `--color-accent-200`
bold letters. The modes are chosen on the Perfil.

### 3. The sidebar (224 px; 200 px compact) — or the rail (48 px; 42 px compact)

A column at the left of the screen with a right hairline; it scrolls when the
window is short. Groups in this order; each group after the first opens with
an uppercase **kicker** (9.5 px, wide tracking, 40 % ink) followed by a
hairline that fades to transparent over 48 px. The first group has no kicker.

| Kicker              | Label (verbatim) | Route                             | Icon                     | Extras                                                           |
| ------------------- | ---------------- | --------------------------------- | ------------------------ | ---------------------------------------------------------------- |
| —                   | Dashboard        | `/dashboard`                      | `ph-gauge`               |                                                                  |
| **Esteira**         | Triagem          | `/triage`                         | `ph-tray-arrow-down`     |                                                                  |
|                     | Revisão          | `/review`, `/review/[id]`         | `ph-check-square-offset` |                                                                  |
|                     | Faturamento      | `/billing`                        | `ph-receipt`             |                                                                  |
| **Fora da esteira** | Ignorados        | `/discarded`                      | `ph-archive`             |                                                                  |
|                     | Consulta         | `/query`                          | `ph-brackets-curly`      | parent, fold caret                                               |
|                     | └ Relatórios     | `/sheets`, `/sheets/[id]`         | `ph-chart-bar`           | sub-item of Consulta                                             |
|                     | Catálogos        | `/catalogs`                       | `ph-table`               | parent, fold caret                                               |
|                     | └ Listas         | `/shortlists`, `/shortlists/[id]` | `ph-list-checks`         | sub-item of Catálogos                                            |
| **Sistema**         | Configurações    | `/settings`                       | `ph-gear-six`            |                                                                  |
|                     | Logs             | `/logs`                           | `ph-terminal-window`     |                                                                  |
| **Ideias futuras**  | NFePackMonitor   | `/nfe-pack-monitor`               | `ph-receipt-x`           | needs the monitor's folder (disabled otherwise)                  |
|                     | Inpart Saúde     | `/inpart`                         | `ph-first-aid-kit`       | badge "estudo"                                                   |
|                     | Opmenexo         | `/opmenexo`                       | `ph-plugs-connected`     | needs the Spook: a folder **or** an address (disabled otherwise) |
|                     | Duplicidades     | `/duplicates`                     | `ph-copy-simple`         | badge "estudo"                                                   |
|                     | Costantini       | `/costantini`                     | `ph-buildings`           | badge "estudo"                                                   |

Destinations with no sidebar item: `/profile` (user menu), `/welcome` (the
first-assembly door), `/notices` (the bell), `/search` (the titlebar field).

- **Item**: 15 px icon + label (12.5 px, 72 % ink); hover tints the row 7 %.
- **[functional]** **The current item** is marked by the accent as a tint
  (13 %) **and** a 2 px accent bar down the row's leading edge, with
  `--color-accent-200` ink. **Exactly one line is lit.** A route below an item
  keeps it lit (`/review/123` lights Revisão).
- **Sub-items** (Relatórios, Listas): indented behind a 1 px vertical hairline;
  14 px icon, 12 px label, quieter ink (56 %). Current = tint + ink, **no**
  leading bar (beside the hairline it would read as a second, thicker line).
  The parent does not light up for a visible current child.
- **Fold**: each parent carries a small caret button at the right end of its
  row (`ph-caret-down`, turned −90° when folded; 40 % opacity, 80 % on hover).
  The row itself navigates; only the caret folds. Sub-lists are **born
  folded**, and a fold lasts for the session only. While folded, a current
  child hands its mark to the parent — the one exception to "the parent never
  lights for a child", because no lit line at all is worse.
- **Study badge**: "estudo" in a quiet pill (mono 10.5 px, `--fill-quiet`,
  55 % ink) on the three study items
  ([screens/14-future-studies.md](screens/14-future-studies.md)).
- **Disabled item — the service was never configured.** Opmenexo when the
  Spook has neither a folder nor an address; NFePackMonitor when the monitor
  has no folder. Drawn at 45 % opacity, not clickable, not reachable by Tab, no
  hover tint, and the platform tooltip says why: "Precisa do Spook configurado
  — Configurações · Spook" / "Precisa do NFePackMonitor configurado —
  Configurações · NFe Pack". **Never hidden.**
- **Dimmed icon — configured, but not running.** Only the icon drops to 40 %;
  the item stays clickable; tooltip "Spook parado — Configurações · Spook" /
  "Monitor parado — Configurações · NFe Pack". Read from the status the status
  bar already polls; nothing is dimmed before the first answer.
- Every other item's tooltip is its own label.
- **Counters: none.** No item carries a number today.
- **The rail** (folded sidebar): icons only, centred; labels, badges and
  kicker words are clipped away (still announced to screen readers), the kicker
  hairlines become solid full-width separators, and sub-items are always shown
  as plain icons — no indent, no hairline, no caret; the fold is ignored,
  because in the rail the icons are the only access. Tooltips carry the names.
- **When it folds** — the "Barra lateral" preference, set on the Perfil:
  "Automática" (the rail when the window is 1180 px wide or narrower, the full
  sidebar above), "Sempre aberta", "Sempre recolhida".

### 4. The screen area

Between the titlebar and the status bar, right of the sidebar. It belongs to
each screen; it scrolls vertically only.

### 5. The status bar (25 px, bottom)

A top hairline, a ground slightly lifted from the page (text at 3 % over
`--color-bg`), 10.5 px text. **Left: five service blocks**, each a button:
icon (12 px, coloured by tone) + name (78 % ink, weight 500) + a sentence
(52 % ink; 40 % when idle). Hover tints 7 %. A **warn** or **danger** tone
washes the whole block (the state colour at 11 %). Clicking a block opens that
service's section of Configurações ([screens/10-settings.md](screens/10-settings.md)).

| Block      | Icon                                       | Opens                    | Tooltip (verbatim)                                                           |
| ---------- | ------------------------------------------ | ------------------------ | ---------------------------------------------------------------------------- |
| MailBridgr | `ph-tray`                                  | Configurações › Doca     | "MailBridgr entrega; a Doca vigia quando existir. Abre Configurações · Doca" |
| MedLens    | `ph-eye` when ok, `ph-eye-slash` otherwise | Configurações › MedLens  | "MedLens — os olhos da esteira. Abre Configurações · MedLens"                |
| Fantasma   | `ph-robot`                                 | Configurações › Fantasma | "Fantasma — as mãos no Everest. Abre Configurações · Fantasma"               |
| Spook      | `ph-plugs-connected`                       | Configurações › Spook    | "Spook — os portais. Abre Configurações · Spook"                             |
| NFe Pack   | `ph-receipt`                               | Configurações › NFe Pack | "NFePackMonitor — as notas do portal ACOM. Abre Configurações · NFe Pack"    |

**The sentences, and the tone each one takes** (the single source is
`service-blocks.ts`; the Dashboard's Serviços card draws the same ones —
[screens/01-dashboard.md](screens/01-dashboard.md)):

| Block      | Condition                                                  | Sentence                                 | Tone   |
| ---------- | ---------------------------------------------------------- | ---------------------------------------- | ------ |
| MailBridgr | the profile's e-mail was confirmed by code                 | "canal confirmado · doca a caminho"      | ok     |
|            | otherwise                                                  | "doca ainda não vigiada"                 | idle   |
| MedLens    | no answer yet                                              | "…"                                      | idle   |
|            | starting                                                   | "subindo…"                               | busy   |
|            | up, credential refused                                     | "sem credencial — 401 em tudo"           | warn   |
|            | up (ours)                                                  | "no ar"                                  | ok     |
|            | up, raised outside the app                                 | "no ar · de fora do app"                 | ok     |
|            | stopped                                                    | "parado"                                 | idle   |
| Fantasma   | folders, user names or credentials missing (checked first) | "falta configurar"                       | warn   |
|            | no answer yet                                              | "…"                                      | idle   |
|            | working right now (overrides every state below)            | "ocupado"                                | busy   |
|            | container up and ERP logged in                             | "pronto"                                 | ok     |
|            | container up, no ERP session                               | "sem sessão"                             | warn   |
|            | starting / shutting down                                   | "subindo…" / "desligando…"               | busy   |
|            | raised by another program                                  | "de fora do app"                         | idle   |
|            | off                                                        | "parado"                                 | idle   |
| Spook      | no answer yet                                              | "…"                                      | idle   |
|            | not configured                                             | "sem endereço"                           | idle   |
|            | this app could raise it, not answering                     | "parado"                                 | idle   |
|            | configured elsewhere, not answering                        | "sem resposta"                           | warn   |
|            | answering, credential refused                              | "credencial recusada"                    | danger |
|            | answering, no credential stored                            | "sem credencial"                         | warn   |
|            | answering                                                  | "no ar"                                  | ok     |
| NFe Pack   | no answer yet                                              | "…"                                      | idle   |
|            | no folder                                                  | "sem pasta"                              | idle   |
|            | running (ours / raised outside the app)                    | "baixando" / "baixando · de fora do app" | ok     |
|            | not running                                                | "parado"                                 | idle   |

Tones today: ok = green icon; warn = amber icon + amber wash; danger = red
icon + red wash; busy = accent icon; idle = icon at 38 % ink. The block names
("MailBridgr", "MedLens", "Fantasma", "Spook", "NFe Pack") are proper names.

**Right side** (after a flexible gap), two small items:

- **"PIN ativo"** (`ph-lock`, green) — only when the system wallet says a PIN
  is stored. Opens the Perfil; tooltip "A janela se tranca sozinha — ajustes no
  Perfil".
- **"v0.1.0"** (`ph-info`, idle ink) — the app's version, once read. Opens
  Configurações (Sobre); tooltip "Sobre este app".

The blocks refresh themselves while the bar is on screen — MedLens every 5 s,
Fantasma every 20 s, Spook and NFe Pack every 30 s — so they stop in the wizard
and under the lock. The shell carries no on/off switch for any service: those
live in Configurações (the panels, and the Sobre list's autostart switches).

### 6. Toasts

- A stack fixed at the **bottom-right** corner, 11 px from the window's right
  and bottom edges — today it sits over the right end of the status bar.
  Newest at the bottom; at most **4** on screen (the oldest yields); each lives
  **5 s**; a click dismisses it.
- One toast: up to 360 px wide, surface ground, hairline border (tinted 45 %
  for warn and danger), large elevation, a tone icon (14 px) and one sentence
  (12 px). Tones: ok `ph-check-circle` (green), warn `ph-warning` (amber),
  danger `ph-warning-octagon` (red), accent `ph-info` (accent). It enters with
  a 140 ms fade and a 4 px rise. Announced politely to screen readers.
- Never drawn over the locked window; drawn in the wizard.
- The one toast the shell itself owns: a preference that could not be written
  — "A preferência vale nesta sessão, mas não foi gravada: {motivo}" (danger),
  from whichever screen changed it. Every other toast belongs to the screen
  that fires it and is listed in that screen's document.

### 7. The locked window

Locked, the body (sidebar, screen, status bar, toasts) is replaced by the lock
screen; the titlebar stays, because the window's chrome is still the window's.
Full description: [screens/16-profile-and-lock.md](screens/16-profile-and-lock.md).

### Theme and density, app-wide

- The resolved base goes on the root as `data-theme="dark" | "light"`
  ("Sistema" follows the desktop); the chosen palette paints its tokens over
  the base. Four palettes ship: Breeze Dark (the default), Breeze Light, GitHub
  Dark, GitHub Light; custom themes come from `config.json`. Details and the
  token roles: [03-platform.md](03-platform.md) § Theming.
- Density goes on the root as `data-density="standard" | "compact"` and scales
  the spacing tokens **and the chrome**: titlebar 38 → 32 px, sidebar 224 →
  200 px, rail 48 → 42 px, nav row padding, body text 13 → 12.5 px. The status
  bar's 25 px and the controls' sizes are fixed numbers today.
- Where they are set: theme, palettes and density in Configurações ›
  Aparência; sidebar mode and language on the Perfil. Every change applies at
  once — no Save button anywhere.

## Data shown

| Element                | Source                                       | Presentation rule                                             | Example                                  |
| ---------------------- | -------------------------------------------- | ------------------------------------------------------------- | ---------------------------------------- |
| "· context"            | the route                                    | the destination's label; blank for an unknown route           | "· Revisão"                              |
| First name             | profile name, first word                     | "Sem nome" (quiet) when blank                                 | "Camila"                                 |
| Face                   | profile avatar mode + seed + stored photo    | photo › drawn style › initials › person glyph                 | CR                                       |
| Menu header            | profile name and role                        | role line absent when blank                                   | "Camila Duarte Rocha" / "Faturista OPME" |
| Bell badge             | unread notices ([17](screens/17-notices.md)) | hidden at zero                                                | 3                                        |
| Menu's way out         | the wallet: is a PIN stored?                 | "Encerrar sessão" (PIN) / "Encerrar o app" (no PIN)           |                                          |
| Service sentences      | the five statuses (table above)              | "…" until the first answer                                    | "no ar"                                  |
| "PIN ativo"            | the wallet                                   | absent without a PIN, and until the wallet answers            | "PIN ativo"                              |
| Version                | the app itself                               | absent until read                                             | "v0.1.0"                                 |
| Disabled / dimmed item | stored configuration / live status           | disabled = never configured; dimmed = configured, not running |                                          |

**Counters:** the bell badge is the only count in the shell. The sidebar has
none; the status bar has none.

## States

| State                     | Trigger                           | What is drawn                                                                                                                             |
| ------------------------- | --------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| Windowed                  | default                           | rounded corners + hairline border                                                                                                         |
| Maximized                 | Maximizar, or the desktop         | square corners, no border; middle control shows `ph-corners-in` "Restaurar"                                                               |
| Narrow window             | ≤ 1180 px wide, mode "Automática" | the rail; the context and first name ellipsize; the search field shrinks                                                                  |
| Compact density           | Configurações › Aparência         | smaller chrome (sizes above)                                                                                                              |
| Dark / light / palette    | Aparência or the desktop          | tokens repaint; state colours keep their meaning                                                                                          |
| First moments after start | nothing answered yet              | sentences "…" (the Fantasma block says "falta configurar" instead when it is not configured); no "PIN ativo"; no version; no dimmed icons |
| No profile name           | profile never filled              | "Sem nome" in quiet ink, person glyph                                                                                                     |
| Service never configured  | configuration                     | sidebar item disabled with its reason; idle sentence ("sem endereço", "sem pasta"); the Fantasma's is the amber "falta configurar"        |
| Service stopped           | live status                       | sidebar icon dimmed; "parado"                                                                                                             |
| Service raised elsewhere  | live status                       | "· de fora do app" suffix, or "de fora do app" for the Fantasma                                                                           |
| Service starting / busy   | live status                       | accent icon, "subindo…", "ocupado", "desligando…"                                                                                         |
| Needs attention           | live status                       | amber or red wash on that block                                                                                                           |
| Menu or bell open         | click                             | popover under the control; closes on outside click or Esc                                                                                 |
| Search focused            | click or Ctrl K                   | accent-tinted border, `kbd` hidden                                                                                                        |
| Wizard                    | `/welcome`                        | reduced shell (table at the top)                                                                                                          |
| Locked                    | PIN lock                          | titlebar + lock screen; bell and user button disabled; no search                                                                          |
| Preference not written    | disk refused                      | the change stays for the session; danger toast                                                                                            |

## Actions

| Gesture                               | Where                           | Precondition / disabled reason                                              | Effect                                                                                  | Confirmation | Reversible                                 | Feedback           | Shortcut                                                     |
| ------------------------------------- | ------------------------------- | --------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- | ------------ | ------------------------------------------ | ------------------ | ------------------------------------------------------------ |
| Drag                                  | titlebar, non-interactive parts | —                                                                           | moves the window                                                                        | —            | yes                                        | —                  | —                                                            |
| Double-click                          | titlebar                        | unverified: the drag-region default of the window toolkit (toggle maximize) | —                                                                                       | —            | —                                          | —                  | —                                                            |
| Minimizar                             | window controls                 | —                                                                           | minimizes                                                                               | —            | yes                                        | —                  | —                                                            |
| Maximizar / Restaurar                 | window controls                 | —                                                                           | toggles maximized                                                                       | —            | yes                                        | corners change     | —                                                            |
| Fechar                                | window controls                 | —                                                                           | with a PIN stored: locks first (the lock is kept on disk), then closes; without: closes | none         | a PIN-locked close reopens locked          | —                  | —                                                            |
| Search                                | titlebar field                  | not in the wizard, not locked; a blank term does nothing                    | opens `/search` with the term                                                           | —            | Esc on the results returns                 | results screen     | Ctrl K focuses and selects; Enter runs; Esc leaves the field |
| Bell                                  | titlebar                        | disabled under the lock; absent in the wizard                               | opens the notices panel                                                                 | —            | —                                          | —                  | Esc closes                                                   |
| User button                           | titlebar                        | disabled under the lock; absent in the wizard                               | opens the menu                                                                          | —            | —                                          | —                  | Esc closes                                                   |
| Perfil / Preferências / Configurações | user menu                       | —                                                                           | navigates (Preferências lands on Aparência)                                             | —            | —                                          | —                  | —                                                            |
| Encerrar sessão                       | user menu (PIN stored)          | —                                                                           | locks the window now                                                                    | none         | unlock with the PIN or the MailBridgr code | the lock screen    | —                                                            |
| Encerrar o app                        | user menu (no PIN)              | —                                                                           | closes the app                                                                          | none         | relaunch                                   | —                  | —                                                            |
| Sidebar item                          | sidebar                         | disabled items do nothing; the reason is on hover                           | navigates                                                                               | —            | —                                          | the lit line moves | —                                                            |
| Fold caret                            | parent rows                     | absent in the rail                                                          | shows / hides the sub-items                                                             | —            | yes (session only)                         | caret turns        | —                                                            |
| Service block                         | status bar                      | —                                                                           | opens that service's Configurações section                                              | —            | —                                          | —                  | —                                                            |
| "PIN ativo"                           | status bar                      | only while a PIN is stored                                                  | opens the Perfil                                                                        | —            | —                                          | —                  | —                                                            |
| Version                               | status bar                      | once read                                                                   | opens Configurações (Sobre)                                                             | —            | —                                          | —                  | —                                                            |
| Toast                                 | bottom-right                    | —                                                                           | dismisses it                                                                            | —            | —                                          | —                  | —                                                            |

**Global keyboard** — these are all the app-wide bindings:

| Keys                  | Where it works               | Effect                                                                          |
| --------------------- | ---------------------------- | ------------------------------------------------------------------------------- |
| `Ctrl K`              | unlocked, outside the wizard | focuses and selects the search field — plain `Ctrl K` only                      |
| `Enter` / `Esc`       | in the search field          | runs the search / leaves the field                                              |
| `Esc`                 | anywhere                     | closes the user menu and the bell panel                                         |
| `Ctrl ⇧ K`            | a screen with a filter       | that screen's filter ([05-components.md](05-components.md)) — never the shell's |
| `0`–`9`, `⌫`, `Enter` | the locked window            | types, erases and submits the PIN                                               |

No binding exists for Configurações, for locking, or for a shortcuts overlay.

## Business rules

1. **One navigation table** (`nav.ts`) feeds the sidebar, the titlebar context
   and the window title; no screen sets its own title — a title every page must
   remember is the one somebody forgets, on the page where they are lost.
2. **The order is the order the work moves**: Triagem → Revisão → Faturamento.
   A conveyor listed out of order teaches the wrong sequence (`nav.ts`).
3. **One lit line.** Two lit lines make the person read the parent as where
   they are; the one exception is a folded parent sheltering its current child
   (Bridgr Pro decision 006 § 2, "The sidebar's sub-lists fold").
4. **Sub-items go one level deep, no deeper** — a 48 px rail has nowhere to
   draw a second level (`nav.ts`).
5. **A destination whose service was never configured is disabled with its
   reason, never hidden** — a menu that hides what it cannot do teaches nobody
   it exists. The test is a **stored** fact (Spook: folder **or** address;
   monitor: folder); a service that is merely down only dims the icon, or the
   menu would blink between existing and not at every restart (decision 024 § 7).
6. **No counters until something real is counted.** A "0" on every line, or a
   number nobody produced, is the screen asserting what nobody verified; and a
   counter would be amber only when the item waits on a person (`nav.ts`,
   [02-invariants.md](02-invariants.md) § 2–3).
7. **The status bar says only what somebody checked.** Live states come from
   the services' answers; MailBridgr speaks from a stored fact (the e-mail
   confirmed by code) and the Fantasma warns "falta configurar" from its stored
   fields before any live state. "…" means nobody answered yet. Busy overrides
   the label; a service raised elsewhere is said, never hidden; an unreachable
   service is never drawn as a broken credential — hence the Spook's separate
   sentences (decision 006 § 2; [02-invariants.md](02-invariants.md) § 15). Bar
   and Dashboard card share one source so they can never disagree.
8. **The wizard is a door, not a destination**: while it is open there is no
   sidebar, status bar, bell, user menu or search — nothing that offers a way
   around it (decision 006 § 2).
9. **The lock hides every door**: the body is replaced, the menu and the bell
   are disabled, the search disappears, the status bar goes (the bar tells what
   runs; the lock exists to not tell), and toasts go (one floating over the
   lock would tell a passer-by what just happened) (decisions 008, 010).
10. **The X always closes, and with a PIN it locks first**, so the next launch
    opens on the locked window — closing is never a way around the PIN. The
    menu's way out changes with the PIN: "Encerrar sessão" locks and stays,
    "Encerrar o app" closes (decision 008).
11. **A toast is feedback, a notice is a pending subject.** Toasts are never
    stored. A deliberate act may chirp on success; a setting that saves on
    every toggle stays silent on success and toasts only its failure — from any
    screen (decision 010).
12. **Preferences apply before they are written.** The screen changes first,
    the disk second; a failed write keeps the change for the session and says
    so. There is no Save: the effect of a theme is the screen in front of you.
13. **The window never takes a shadow of its own**, and its corner radius is
    written twice (page and GTK) and dropped when maximized
    ([03-platform.md](03-platform.md)).
14. **The auto-lock clock lives in the shell**: armed only while a PIN is
    stored and the chosen minutes are above zero; pointer, keyboard and wheel
    activity reset it; checked every 15 s. When it fires, the window locks and
    the bell receives "A janela se trancou sozinha" — it happened without a
    hand ([screens/16-profile-and-lock.md](screens/16-profile-and-lock.md)).
15. **Colours of state are not a palette choice**: themes repaint surfaces and
    the accent, never the state roles or the paper (`theme.svelte.ts`).
16. **Every icon must exist in the bundled set** — a missing icon class renders
    as nothing, and an empty box in a sidebar looks like a bug (`nav.ts`).

## Must survive the redesign

- A current-item mark that is **more than a tint** and survives the rail;
  exactly one lit line; the folded-parent exception.
- The conveyor order, and every destination reachable from the rail —
  sub-items included.
- Disabled-with-reason (never configured) and dimmed (not running) as two
  different, visible things; neither ever hides the item.
- Study items say "estudo" **before** the click.
- Five service blocks, each opening its service's settings, with the exact
  sentences and tone rules above: "…" for unknown (never green), busy
  overriding, the "de fora do app" suffix, amber and red only where the table
  puts them.
- "PIN ativo" drawn only from the wallet's answer.
- The reduced shell in the wizard and the replaced body under the lock; no
  toast over the lock.
- The X locks before closing when a PIN exists; the menu's way out changes
  wording with the PIN.
- No window shadow from the app; square corners when maximized.
- The titlebar remains a drag handle across its non-interactive parts.
- Toasts: short-lived, dismissible, never the only record of a failure that
  needs a decision (that is a notice).
- `Ctrl K` focuses search; `Esc` closes the popovers.
- Every colour through a token; density must reach the chrome.

## Free to redesign

- The look of all of it: heights, widths (within the 1024 px minimum), the
  kicker style, the current-item mark's form, the badge, the rail, the menu,
  the popovers, the toast card and its corner.
- Where the bell, user button and search sit inside the titlebar; how the
  status bar balances icon, name and sentence.
- Group names and grouping beyond the conveyor order — proposals welcome
  ([00-brief.md](00-brief.md) § What is fixed and what is free).
- The mark's shapes and its sparkle colour are brand, not chrome: a proposal,
  not a free choice.

## Sample content for mockups

- Titlebar: **Bridgr Pro · Revisão**; user button **CR · Camila**; menu header
  "Camila Duarte Rocha" / "Faturista OPME"; bell badge **3**.
- Status bar, a good morning: MailBridgr "canal confirmado · doca a caminho"
  (ok) · MedLens "no ar" (ok) · Fantasma "pronto" (ok) · Spook "sem
  credencial" (warn wash) · NFe Pack "baixando" (ok) … "PIN ativo" · "v0.1.0".
- Status bar, a bad hour: MedLens "sem credencial — 401 em tudo" (warn) ·
  Fantasma "ocupado" (busy) · Spook "sem resposta" (warn) · NFe Pack "parado"
  (idle).
- A new station: MailBridgr "doca ainda não vigiada" · MedLens "parado" ·
  Fantasma "falta configurar" (warn) · Spook "sem endereço" · NFe Pack "sem
  pasta"; in the sidebar Opmenexo and NFePackMonitor disabled with their
  reasons.
- Toasts: "PIN gravado." (ok) · "Foto salva." (ok) · "A preferência vale nesta
  sessão, mas não foi gravada: {motivo}" (danger; the reason is the system's
  own message).
- Machine and version, if shown: `estacao-faturamento-01`, `0.1.0`
  ([06-sample-data.md](06-sample-data.md)).

## Design opportunities

1. **Lift the toast stack above the status bar.** Today it floats 11 px from
   the bottom edge, over "PIN ativo" and the version.
2. **Let density reach the status bar.** Its 25 px height is a fixed number,
   the one piece of chrome compact mode does not tighten; a chrome token would
   fix it.
3. **Regroup "Ideias futuras".** Two built, working screens (NFePackMonitor,
   Opmenexo) sit under a kicker that says "future ideas", beside three studies
   that may never be built. Separating built portal/notes screens from studies
   would stop the heading from under-selling them.
4. **Make the disabled reason visible without a mouse.** It exists only as a
   platform tooltip, and a disabled item cannot receive focus, so a keyboard
   user never meets the reason. An inline hint, or a focusable disabled row,
   would keep the rule "disabled, saying why".
5. **Name the Settings section in the context** ("· Configurações ·
   Aparência"). The route already carries the section; today the titlebar says
   only "· Configurações".
6. **One word per act.** The menu's "Encerrar sessão" locks the window, and the
   Perfil's PIN card calls the same act "Trancar agora"; the menu's
   "Preferências" opens Aparência, while the Perfil has a card also titled
   "Preferências" holding sidebar and language. A proposal — labels change only
   with the operator.
7. **Doors by state in the status bar.** The Dashboard's Serviços card sends a
   running service to its logs/console and a stopped one to its settings; the
   bar always goes to settings. Matching them would make one click answer "what
   is it doing".
8. **Sidebar counters, now that the conveyor works.** The original design
   drew right-aligned counters, amber only when a person is needed. The
   Dashboard's funnel already reads real sources for Triagem, Revisão and
   Faturamento; a counter fed by them would be honest — never a "0" before the
   first read, "—" when a source could not be read. Needs implementation.
9. **A grouping cue in the rail.** Folded, Relatórios and Listas look like
   siblings of everything else; a cue that fits 48 px (not a second indent)
   would keep "this belongs to the line above".
10. **Keep "estudo" visible in the rail.** The badge is clipped away when
    folded, so the study nature disappears; a tiny, non-amber marker would
    keep it.
11. **Keyboard hints in the menu.** The original design shows "Ctrl ," beside
    Configurações, "Ctrl L" for locking and a "?" shortcuts overlay; none of
    these bindings exist, so drawing them requires implementing them first.

## Open questions

- Counters: when, and fed by which source? The code's own note promised them
  "with Triagem", which is now built.
- The design's "Serviços" health block in the sidebar footer was left out while
  the services were not wired; they are now, and the status bar carries them.
  Does the sidebar block ever come back?
- The MailBridgr block's sentences ("doca a caminho", "a Doca vigia quando
  existir") predate the Doca, which exists today. Should the block speak the
  dock's state — and from which stored fact?
- The "Atalhos do teclado" item of the design's user menu is not built. Is it
  coming?
