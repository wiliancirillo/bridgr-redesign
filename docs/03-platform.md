# The platform — what the look has to live inside

Technical facts that constrain any new look. They come from the code of Bridgr
Pro (`src/app.css`, `src/lib/theme.svelte.ts`, `src-tauri/tauri.conf.json`,
`src-tauri/src/frame.rs`) and from defects the app already paid for.

## The runtime

- A **desktop app** built with Tauri 2: the interface is HTML/CSS (Svelte 5 +
  SvelteKit, TailwindCSS 4) rendered by **WebKitGTK 4.1** — WebKit, not
  Chromium — on **Linux**, developed and used on KDE Plasma under Wayland.
- **One window**, one user, mouse and keyboard. No touch, no mobile, no
  second monitor layout.
- **Offline by construction.** The content security policy is
  `default-src 'self'; img-src 'self' asset: data:; style-src 'self' 'unsafe-inline'; font-src 'self' data:`
  — nothing is fetched from the network: no web fonts, no remote images, no
  CDN icons. Anything the design needs ships inside the app.

## The window

| Fact                  | Value                                                                                    |
| --------------------- | ---------------------------------------------------------------------------------------- |
| Default size          | 1440 × 900                                                                               |
| Minimum size          | 1024 × 768                                                                               |
| Decorations           | none from the system: the app draws its own titlebar in HTML, draggable, with its own window controls |
| Shadow                | drawn by **GTK** (client-side decorations), never by the app                             |
| Corners               | rounded by CSS on the root element (today 8 px), squared when maximized, tiled or fullscreen |
| Sidebar               | folds into a narrow icon rail below **1180 px** of window width (and by preference)      |

⚠️ **Never a `box-shadow` on the window itself.** Drawing the shadow inside an
oversized transparent window (a "halo") breaks snapping, tiling, maximize and
restore on the desktop — measured, and paid for. The window's shadow is the
desktop's. Menus, popovers and dialogs **inside** the window may have
elevation.

⚠️ The window corner radius is written in two places that cannot read each
other (the CSS and a GTK style provider in `frame.rs`); a new radius changes
both.

## Controls the platform would otherwise paint

WebKitGTK paints native form controls with the **GTK theme**, not the app's, so
the app draws its own:

- **Checkboxes** are drawn by the app (a hand-drawn box with a check, and a
  minus for "some of these").
- **No native date input** — the WebKitGTK calendar ignores the theme and does
  not close on an outside click. Dates are typed or picked in the app's own
  date picker ([05-components.md](05-components.md)).
- **No native `<select>` look** — choosers are the app's own popovers.
- **Tooltips** are the platform's (`title` attributes): the app uses them to
  say **why** a control is disabled. A design that needs richer tooltips has to
  draw them.

Surfaces the app does **not** control, and the design should expect as they
are:

- **PDF documents** are shown by WebKit's native `<embed>` viewer, with its
  own toolbar, zoom and pages. **Nothing can be drawn over it.** When the
  operator must mark regions on a PDF, the app swaps the embed for a page
  rendered by pdf.js.
- **HTML e-mail bodies** are shown in a fully sandboxed iframe (no scripts).
- **File choosers** are the desktop's own dialog (KDE's, through the XDG
  portal).
- Photos are shown in the app's own viewer (zoom, rotate, drag).

## Type and icons

- Today's font stack is `Inter, system-ui, sans-serif`, and ⚠️ **Inter is not
  bundled** — it renders only where it is installed; elsewhere the desktop's
  default face appears. The monospace stack (`ui-monospace, "SF Mono",
  "JetBrains Mono", Menlo, monospace`) is not bundled either. A typeface the
  new design depends on must be bundled with the app.
- Icons are **Phosphor**, bundled as a web font, in the regular and fill
  weights (bold is not shipped). A different set must also be bundled.
- Avatars in the "drawn" mode use **DiceBear** styles, bundled and drawn
  locally.
- Three code editors (CodeMirror 6) appear: SQL on the Consulta and the
  catalogs' SELECT dialog, TOML in the portal clients editor, XML for an NF-e.
  They follow the app's tokens.

## Theming — how a look is expressed

Every colour in the code is a **CSS custom property**; no hex is typed outside
the token file. The operator chooses **Sistema / Claro / Escuro** and a palette
per base, and can write custom themes in `config.json`. The new look has to be
delivered in this shape.

- `data-theme="dark" | "light"` on the root selects the base; "Sistema" is
  resolved against the desktop before it gets there.
- **Four palettes ship today**: Breeze Dark (the default), Breeze Light,
  GitHub Dark, GitHub Light.
- ⚠️ A **custom theme may paint only six roles** over its base — `bg`,
  `surface`, `text`, `divider`, `accent`, `desk`. Everything else comes from the
  base. So the rest of the system must stay legible under any background and
  accent a person picks. Today the accent's ramp (200…900) is **not** derived
  from the chosen accent — a custom accent keeps the default ramp. A new token
  system can fix this by deriving the ramp from the accent.
- ⚠️ **State roles and paper are not themeable, on purpose.** Amber means a
  human is needed and green means somebody checked; a theme that repaints
  danger green would not be ugly, it would be wrong. And the scanned document
  keeps its near-white sheet in both themes — paper is paper.

### The roles the code uses today

Keep the roles (or propose a mapping to new ones); the values are free.

| Group            | Tokens                                                                                     | Role                                                                   |
| ---------------- | ------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------- |
| Base             | `--color-bg`, `--color-surface`, `--color-text`, `--color-divider`, `--color-accent`       | ground, raised surface, ink, lines, interaction                        |
| Accent ramp      | `--color-accent-200/300/400/700/900`                                                       | sorted header, numeric cells, tints; flips between dark and light      |
| Neutral ramp     | `--color-neutral-100/200/300/400/600/700/800/900`                                           | text on tinted fills, quiet fills, hairlines; flips between themes     |
| State roles      | `--st-warn`, `--st-danger`, `--st-ok`, each with `-fg` (text on tint) and `-bg` (the tint)  | human needed · destructive/failed · somebody checked                   |
| Document viewer  | `--desk`, `--desk-inner`, `--paper`, `--paper-ink`, `--paper-rule`, `--paper-band`, `--paper-dim` | the surface a document lies on (darker than the ground); the sheet |
| Misc             | `--fill-quiet`, `--barra` (scrollbar thumb), `--code-ink`, `--brand-spark` (the logo's sparkle — brand, not state) | |
| Type             | `--font-body`, `--font-heading`, `--mono`, `--font-body-size`                              |                                                                        |
| Space            | `--space-1` … `--space-6`                                                                  | one compact scale                                                      |
| Shape            | `--radius-sm/md/lg`; `--shadow-sm/md/lg`                                                   | radius 4 / 8 / 14 today; elevation for menus and dialogs only          |
| Chrome sizes     | `--titlebar-h`, `--sidebar-w`, `--rail-w`, `--nav-item-y`                                  | tokens so density reaches the frame too                                |

Secondary ink is not a grey token: it is the text colour faded with
`color-mix()` at a small ladder of strengths (≈40 %, 45–48 %, 55 %, 70–72 %).

### Density

- **App density** (Configurações → Aparência): the default is already a compact
  scale; the "compact" mode tightens it further across everything — spacing,
  titlebar (38 → 32 px), sidebar (224 → 200 px), rail (48 → 42 px), body text
  (13 → 12.5 px).
- **Grid density** (the conforto/compacto pair over every grid): reaches the
  rows only, and is remembered. "More rows" and "a tighter app" are two
  different wishes, with two controls.

## Interaction basics already fixed in code

- Focus: a 2 px accent outline with a 2 px offset (`:focus-visible`); inputs
  turn their border to the accent instead.
- Text selection: accent at 30 %.
- Scrollbars: 9 px, rounded thumb, transparent track.
- The only animation is the "alive" spinner (1.1 s per turn), stopped under
  `prefers-reduced-motion`.
- Drag-and-drop of files onto the page is enabled for the app's own handlers
  (the photo crop, the avatar photo).

## Scale the design must hold

These are real magnitudes, so the grid and lists must scroll and page rather
than squeeze:

- A `SELECT *` on the Consulta can return **dozens of columns** (37 on one
  common ERP table); the grid grows sideways and scrolls — it never squeezes
  columns into the viewport.
- The materials catalog holds **thousands of items**; the ERP schema has
  **over three thousand tables** for the SQL editor's completion.
- The NF-e history streams in batches of 500 rows.
- One informe can carry **eighteen bills** in one table.
