# Triagem (Doca de recebimento) — `/triage`

> Where: sidebar › Esteira › Triagem (icon `ph-tray-arrow-down`) · Status: built (dock 09/08/2026, crop 11/08/2026, watched folder 12/08/2026, manifest-borne adapter 14/08/2026, grouping 17/08/2026) · Code: `src/routes/triage/+page.svelte`, `src/lib/dock.svelte.ts`, `src/lib/crop.svelte.ts`, `src/lib/crop.ts`, `src/lib/viewer.svelte.ts`, `src/lib/pdf.ts`, `src/lib/components/ViewerZoom.svelte`, `src/lib/ipc.ts` (dock types and commands), `src-tauri/src/dock.rs`, `src-tauri/src/delivery.rs`, `src-tauri/src/crop.rs`, `src-tauri/src/lib.rs` (`read_dock` … `medlens_adapters`)

## The question

«De quem é este anexo, e ele vira extração ou vai fora?» (i18n `question.triage`; this screen does not draw it — other built screens show their question as the header subtitle, while Triagem's subtitle carries its counts).

In practice: for each attachment that arrived, the operator picks the MedLens **adapter** (the client whose report this is — it decides how MedLens reads it), then either sends it to extraction (whole, or only the cropped parts that matter) or sets it aside.

## Where it sits in the work

First station of the conveyor: MailBridgr / watched folder → **Triagem** → MedLens reads → [Revisão](03-review.md) → [Faturamento](04-billing.md).

- **In.** MailBridgr's Thunderbird extension writes each _delivery_ (the attachments the human chose in one e-mail, sometimes the e-mail body too) into this app's own spool. A second entrance, the **watched folder** (Configurações → Doca › «Pasta vigiada»), takes loose PDFs and photos that never rode an e-mail. With the screen open, both show up within about 4 s, with no gesture needed.
- **Out.** An extracted piece moves to the «Já extraídos» view, and its bills are conferred on [Revisão](03-review.md). An ignored piece goes to [Ignorados](05-discarded.md).
- **Reached from.** The sidebar. The [Dashboard](01-dashboard.md): the «Começar pela doca» button, the funnel's «Na doca» stage, the «Só anda com você» rows about pieces without a client and refused extractions, and the MailBridgr service line when it is healthy. The bell notice «Uma extração falhou» ([Avisos](17-notices.md)). A hit in the global search's «Arquivos da doca» group ([Busca](18-search.md)) — which opens this screen but does **not** focus the piece that was found.
- **Depends on.**
  - MedLens: the adapter list, and extraction itself. Only extraction is gated — the dock stays readable and decidable with MedLens down.
  - Configurações → Adapters: can hide adapters from this screen's two choosers.
  - Configurações → Doca: holds «Pasta de entregas (inbox do spool)» (blank = the default path) and «Pasta vigiada» (blank = the entrance does not exist). See [Configurações](10-settings.md).

## Anatomy

Chrome (titlebar, sidebar, status bar, toasts, window frame) is in [04-shell.md](../04-shell.md). Reading order:

```text
Doca de recebimento [reload]                                  [ Na doca (6) | Já extraídos (1) ]
4 aguardando decisão · 1 em extração
(warn strip, when MedLens is known down)      (danger strip, after a refusal)
+----------- rail, 398 px ------------------+------------------- focused piece -------------------+
| [funnel Filtrar peças na doca… Ctrl ⇧ K] [group] | [tile] etiquetas_sala3.jpg          (sem cliente) |
| (Precisam de decisão 4) (Sem cliente 3) … |  2,4 MB · imagem · de faturamento@valeverde.example |
| [!] sender · 02/09/2026 · 08:14 · assunto  3 anexos | (error / no-table banners)                |
| [ ] [icon] file name / meta   (state pill)| CLIENTE / ADAPTER [— escolher — v]  [Ignorar] [Extrair] |
| [ ] [icon] …                              | ↑ ↓ navegar · Enter extrair · Esc ignorar            |
| [folder] Pasta vigiada …                  +-----------------------------------------------------+
|                                           | [icon] etiquetas_sala3.jpg [- 38% + turn] [Recortar] [Abrir] |
|                                           |                  (the document)                     |
+-- selection bar (only while ticked) ------+-- crop bar (crop mode only) ------------------------+
```

### 1. Header (full width, hairline below)

- Title «Doca de recebimento» (the sidebar item says «Triagem»). Right beside it, a small square icon button (`ph-arrows-clockwise`) whose tooltip and accessible name are «Reler a doca»; it re-reads the dock.
- Subtitle, under the title, drawn only once the first read has answered: «{N} aguardando decisão», then « · {M} em extração» only while M > 0. When read, it is drawn even at zero («0 aguardando decisão»).
- At the right, a two-option segmented control: «Na doca ({n})» and «Já extraídos ({m})». The active option wears an accent ring and accent text. **[functional]** It chooses which pieces the rail lists, and switching clears the ticked selection.

### 2. Service strip (conditional, full width, warn wash)

«A extração precisa do MedLens no ar — ligue em Configurações → MedLens.» Drawn when MedLens's state is known and is neither running nor «de fora do app» (i.e. starting up or stopped). Not drawn while the state is unknown.

### 3. Failure strip (conditional, full width, danger wash)

The last dock operation's refusal, verbatim (list under States › Refusals). It stays until the next successful read, choice, ignore or extraction.

### 4. Body — two columns **[functional]**

Left, the dock rail at a fixed 398 px; right, the focused piece and its document, taking the rest. Seeing both at once is the point: the keyboard walks the rail while the document stays in view.

#### 4a. Rail — tools (hairline below)

- **Row 1.** The shared filter field ([05-components](../05-components.md) › FilterField), configured as this screen's primary filter: funnel icon, placeholder «Filtrar peças na doca…», a clear «×» («Limpar filtro») only while there is text, and the key hint «Ctrl ⇧ K» (the binding is live). Here the field **stretches** to fill the row; elsewhere it is 320 px (a recorded departure). Beside it, a 30 px square button (`ph-list-dashes`, tooltip «Agrupar») opens the grouping menu. The button wears the accent while the grouping is anything but the default.
- **Grouping menu** (drops below, right-aligned, at least 168 px wide): four radio items, with a check before the current one — «Por remessa» (default), «Por remetente», «Por adaptador», «Por data». It closes on a pick or on a click outside.
- **Row 2**, only while at least one pill is drawn: the filter pills, wrapping. Each pill is a label plus a small monospace count badge. In order: «Precisam de decisão», «Sem cliente», «Falhou», «Em voo», «Aguardando extração», «Prontas».
  - One pill is active at a time; clicking toggles it.
  - The active pill wears an accent border and an accent wash.
  - The count badge is quiet, except on «Precisam de decisão», «Sem cliente» and «Falhou», where it is **amber** while the pill is not active.

#### 4b. Rail — the list (scrolls)

Groups, each a header line followed by piece rows. **Order:** e-mail deliveries in the order they arrived (oldest first), then the watched folder last. Inside a delivery, pieces keep the manifest's order; inside the watched folder, files are ordered by modification time, then name.

**Header when grouping «Por remessa».** A _shipment_ is one delivery e-mail, or the watched folder.

- Leading icon: an amber filled warning (`ph-warning`) if any piece drawn in this group needs a human (sem cliente or falhou). Otherwise, a folder (`ph-folder-open`) for the watched folder, or an envelope (`ph-envelope-simple`) for an e-mail.
- Sender, semi-bold, ellipsised at about 215 px. The watched folder reads «Pasta vigiada».
- Meta, quiet and ellipsised: «dd/mm/aaaa · hh:mm · {assunto}». For the watched folder the "subject" slot shows the folder's full path, and the date is its newest file's modification time.
- Count at the right: «{n} anexo» / «{n} anexos» (what it counts: Data shown › Counters).

**Header for the other groupings.** These cut across shipments.

- Icon: `ph-user` by sender, `ph-folder-open` for the watched folder, `ph-plugs` by adapter, `ph-calendar-blank` by date.
- Title: the sender name or «Pasta vigiada»; the adapter's label or «Sem adaptador»; or «dd/mm/aaaa».
- No meta.
- Count: the pieces drawn in the group.
- Groups follow the order in which their first piece appears in the dock.

**Piece row.** A checkbox (only in «Na doca»), then a card button holding:

- a 28 px tile with the kind icon — `ph-file-pdf` for a PDF (and anything unknown), `ph-image` for a photo, `ph-table` for an e-mail body;
- the file name (monospace, ellipsised) — for an e-mail body piece, the "file name" is the e-mail's subject;
- a meta line (Data shown);
- a crop badge — an accent chip with a number, only while this piece has regions in the crop basket (tooltip «Regiões marcadas para o recorte»);
- the state pill (States › Piece states).

The card's border turns amber-tinted for sem cliente and falhou. The focused card wears an accent border and an accent wash.

#### 4c. Rail — selection bar (bottom of the rail) **[functional]**

Only while at least one piece is ticked, and only in «Na doca».

- A full-width note: «{n} peça selecionada · a barra age sobre a seleção» / «{n} peças selecionadas · a barra age sobre a seleção».
- The «Cliente do lote…» ▾ menu button. It opens **upward** and lists the visible adapters, with no check mark and no "none" option.
- A small «×» button, «Limpar seleção».
- Spacer, then «Ignorar», then the primary «→ Extrair ({n})» (`ph-arrow-right`), disabled while MedLens is not up.

#### 4d. Right panel — nothing focused

A centred empty state (`ph-files`): «Escolha uma peça na doca para ver o documento.» On arrival, the first listed piece is focused automatically.

#### 4e. Right panel — focus head (hairline below)

- **Identity row.** A 30 px accent tile with the kind icon; the file name (monospace, a little larger); the meta «{tamanho} · {kind} · de {remetente}», where `kind` is the raw word `pdf`, `imagem` or `corpo` (a watched-folder piece has no sender); and the state pill at the right.
- **Error banner** (danger wash, `ph-warning-circle`), when the piece carries an error. It shows the words verbatim (States › Piece errors). When the piece's adapter is known not to read images, a **separate second line** follows: «Este cliente lê o texto do documento, não a imagem: recorte, foto ou PDF escaneado voltam vazios. Envie a peça inteira, ou peça o informe em PDF com texto.»
- **No-table banner** (warn wash, `ph-warning`), for an e-mail body that brought no report table: «Este corpo de e-mail veio sem tabela de informe — o MedLens não teria o que ler. Se o informe está no anexo, é ele que se extrai; esta peça pode ser ignorada.»
- **Action row [functional].**
  - The kicker «Cliente / adapter» (drawn uppercase), then the adapter menu button: the chosen adapter's label, or a muted «— escolher —». Its menu drops below and lists the visible adapters, with a filled check beside the current one.
  - Spacer, then «Ignorar».
  - The primary button (icon `ph-arrows-clockwise`, spinning while this piece extracts or a crop is sending): «Extrair», or «Reextrair» on a pronta piece. In crop mode it becomes the crop's send (4h).
- **Keys line** (quiet, key caps): «↑ ↓ navegar» · «Enter extrair» · «Esc ignorar». In crop mode the last two read «Enter extrair recorte» · «Esc cancelar recorte».

#### 4f. Right panel — viewer strip (hairline below)

The kind icon and the file name (monospace, ellipsised), then, when they apply:

- **Zoom row** (the shared ViewerZoom): «−» (tooltip «Afastar»), the level «NN%» — itself a button, tooltip «Enquadrar (voltar ao início)», which refits and zeroes rotation and pan — then «+» («Aproximar») and a turn button («Girar 90°»). It is drawn for photos, and for a PDF only in crop mode; outside crop mode a PDF uses its own viewer's controls.
- **Pager** (a PDF in crop mode with more than one page): ‹ (tooltip «Página anterior») «{p} de {N}», plus « · {k}» when this piece holds regions — k counts all of its pages — then › («Próxima página»). The arrows disable at the ends.
- **«Recortar»** (`ph-crop`), for photos and PDFs only. In crop mode it becomes «Cancelar recorte», drawn pressed (accent border and wash). Tooltips and disabled reasons are under Actions.
- **«Abrir»** (`ph-arrow-square-out`): opens the piece's file in the desktop's own application.

#### 4g. Right panel — the document (fills the rest, on a darker "desk" ground)

- **Photo.** Centred and fitted, with a soft shadow. Zoom really resizes it (it stays sharp at any level), it rotates in 90° steps, and it can be dragged (grab cursor) once zoomed past the fit. No scrollbars.
- **PDF.** The webview's own PDF viewer, with its own toolbar, scrolling and text selection. It cannot be restyled, and nothing can be drawn over it. In crop mode the pane swaps to the current page rendered as an image (see Crop mode).
- **E-mail body (HTML).** Shown in a sandbox that runs nothing, on a paper-white ground.
- **Messages in the pane:** «Carregando o documento…» while it loads; «Não consegui mostrar o documento: {motivo}» on failure.

#### 4h. Crop bar (bottom of the right panel, crop mode only) **[functional]**

- **Hint/count.**
  - With an empty basket: «Arraste sobre a imagem para marcar o que enviar — cabeçalho, etiqueta, o que for. A marcação sobrevive à troca de foto.»
  - Otherwise: «{n} região marcada» / «{n} regiões marcadas», plus « em {m} fotos» when the basket spans more than one piece, plus « · vão numa imagem só, de cima para baixo».
- **Pan hint**, quieter, only while the image is zoomed past the fit (something is out of sight): a key cap «Espaço» followed by « ou o botão do meio arrasta a imagem».
- Spacer, then the primary «Extrair recorte», which reads «Enviando…» with a spinning icon while it runs.

**Over the image in crop mode.** The cursor is a crosshair, and regions are rectangles: a 2 px accent outline over a light accent veil.

- A **landed** region carries:
  - its **number in the composition**, in an accent badge top-left;
  - a small red «×» top-right (tooltip «Remover a região»);
  - four corner handles — small accent squares ringed in paper white, so they read over dark photos and white paper alike. They rotate with the image.
- A rectangle still being drawn has only the outline and veil.
- While Space is held and the image is zoomed past the fit, every cursor inside it becomes a hand; while panning, a closed hand.

## Data shown

| Field               | Meaning                                                                                                                                                                  | Presentation                                                                                        | Example                                 |
| ------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------- | --------------------------------------- |
| Sender              | The e-mail sender's name as MailBridgr's manifest gives it, or «desconhecido» when blank. The watched folder has none; the screen names it «Pasta vigiada».              | Group header (and «Por remetente» titles); focus-head meta after «de ».                             | «faturamento@valeverde.example»         |
| Subject             | The e-mail subject, or «(sem assunto)» when blank. For the watched folder, the folder's path.                                                                            | Group header meta, after the date.                                                                  | «Informe de consumo OPME — semana 36»   |
| Received            | The e-mail's date (the delivery's time when the e-mail has none). For the watched folder, its newest file's modification time.                                           | «dd/mm/aaaa · hh:mm» in the header; «dd/mm/aaaa» as a «Por data» title.                             | «02/09/2026 · 08:14»                    |
| File name           | The attachment's name in the e-mail (slashes become «\_»). For a body piece, the e-mail subject. For a watched file, its name.                                           | Monospace, ellipsised.                                                                              | «informe_vale_verde_2026-09-02.pdf»     |
| Kind                | `pdf`, `imagem` or `corpo` — MailBridgr's words. Watched files are only `pdf` or `imagem`.                                                                               | Tile icon; printed raw in the focus-head meta.                                                      | `imagem`                                |
| Size                | Bytes of the sent file.                                                                                                                                                  | «812 B», «240 KB», «1,2 MB» (one decimal, comma).                                                   | «2,4 MB»                                |
| Adapter             | The MedLens adapter id, shown by the label MedLens gives it. When the adapter list has not been loaded in this run, the raw id shows instead. With none: «— escolher —». | Row meta after the size; focus menu button; «Por adaptador» group title («Sem adaptador» for none). | «Hospital Vale Verde» (id `vale_verde`) |
| Bills and items     | After extraction: how many bills (documents) MedLens read, and their items summed.                                                                                       | Row meta for a pronta piece: «{tamanho} · {n} conta(s) · {m} item/itens».                           | «1,1 MB · 6 contas · 41 itens»          |
| Row meta, otherwise | —                                                                                                                                                                        | With an adapter: «{tamanho} · {adapter}». Without one: «{tamanho}».                                 | «1,2 MB · Hospital Vale Verde»          |
| Error               | MedLens's own failure words; or the app's («não consegui ler o documento: …»); or the re-extraction note (States).                                                       | Focus-head danger banner, verbatim.                                                                 | «No data extracted from document»       |
| No table            | An e-mail body whose HTML has fewer than three table rows.                                                                                                               | Focus-head warn banner.                                                                             | —                                       |
| Crop count          | Regions of this piece in the basket, across all its pages.                                                                                                               | Accent badge on the row; pager suffix.                                                              | «2»                                     |
| Region number       | The region's place in the composition that will be sent.                                                                                                                 | Accent badge on the region.                                                                         | «3»                                     |

**Counters — exactly what each counts.**

- **«Na doca ({n})»**: every piece in the dock whose on-screen state is neither pronta nor ignorada — all shipments, whatever the filter, pill or grouping. A pronta piece being re-extracted counts here while in flight.
- **«Já extraídos ({m})»**: every piece whose state is pronta, whatever the filter.
- **Subtitle.** «{N} aguardando decisão» is sem cliente plus falhou across the whole dock. «{M} em extração» is extraindo… across the whole dock.
- **Pill counts**: the pieces in the current view (segment) that match the text filter and are in the pill's state. The pills never narrow one another.
  - A pill is drawn while its count is above zero **or** while it is the active one (so an active pill carried into the other view can read 0).
  - «Precisam de decisão» is sem cliente ∪ falhou, and «Em voo» is extraindo….
  - Because the two views split by state, only «Prontas» can ever count in «Já extraídos», and «Prontas» never draws in «Na doca».
- **Group header count.** «Por remessa» counts every piece of that shipment the dock holds, including those listed in the other view or hidden by the filter. The other groupings count only the pieces drawn in the group.
- **Selection note**: the number of ticked pieces. Ticks survive filtering and are cleared when the view switches.
- **«Extrair ({n})»**: the number of ticked pieces, not the number that will actually go (Actions).
- **Crop.** «{n} regiões marcadas» counts regions across the whole basket (all photos, all pages). «em {m} fotos» counts distinct pieces in the basket. The primary's «({n} · {m} fotos)» uses the same two numbers.

## States

### Piece states

The id is the wire's; the label comes from i18n.

| id           | Label              | Tone (colour role) | Needs a human | Meaning                                                                        |
| ------------ | ------------------ | ------------------ | ------------- | ------------------------------------------------------------------------------ |
| `no_client`  | «sem cliente»      | warn (amber)       | **yes**       | No adapter chosen: nobody knows whose report this is.                          |
| `failed`     | «falhou»           | danger (red)       | **yes**       | The extraction errored; the reason is on the piece.                            |
| `queued`     | «aguarda extração» | idle (neutral)     | no            | Has an adapter and waits for the send.                                         |
| `extracting` | «extraindo…»       | busy (accent)      | no            | Sent to MedLens, not answered yet.                                             |
| `ready`      | «pronta»           | ok (green)         | no            | Extracted; listed under «Já extraídos».                                        |
| `ignored`    | «ignorada»         | idle               | —             | Set aside; never drawn in either view — it is on [Ignorados](05-discarded.md). |

The screen draws «extraindo…» from the instant of the click (the sign of life), before the service has answered anything.

### Screen states

- **Not asked yet** (the first read of this app run has not answered): no subtitle, a blank rail list (no empty sentence), no pills, and the right panel's empty state.
- **Arriving on the screen.** The dock is re-read, and the first listed piece is focused, loading its document.
  - View, filter, pill, grouping, ticks and focus are the screen's own and **reset when the operator leaves** (back to «Na doca», «Por remessa», no filter).
  - The crop basket, crop mode and extractions in flight live outside the screen and **survive** leaving and returning.
- **Empty dock.**
  - In «Na doca», an empty state (`ph-tray`): «Nada na doca», with the note «Os informes chegam pelo MailBridgr: escolha os anexos no Thunderbird e envie — eles aparecem aqui, mesmo com o app fechado na hora do envio.»
  - In «Já extraídos», (`ph-arrow-right`) «Nada extraído ainda», with «O que for extraído sai da fila de decisão e fica aqui, a caminho da Revisão.»
- **Filter or pill matches nothing**: a small centred line, «Nenhuma peça responde a este filtro.»
- **Arrival while open.** A cheap pulse runs every 4 s only while this screen is mounted, and a new delivery or loose file appears on its own. With the screen closed nothing polls; the next visit re-reads.
- **MedLens up** (running, or «de fora do app»). The adapter list is loaded once per app run, and manifest suggestions are then adopted (below).
- **MedLens known down** (starting up or stopped):
  - the warn strip shows, and «Extrair», «Reextrair» and «Extrair ({n})» are disabled;
  - «Ignorar» and the adapter choice still work;
  - if the adapter list was never loaded in this run, both menus open **empty**, and pieces that already have an adapter show its raw id.
- **MedLens state unknown** (not polled yet, or its status call failed): extraction is disabled, and **no** strip says why.
- **Suggested adapter.** When the operator marked the client in MailBridgr's popup, the pieces arrive already «aguarda extração», wearing that adapter. On screen this is identical to a human choice. It happens only once MedLens's adapter list (answered at least once in this run) knows the id; otherwise the piece stays «sem cliente».
- **In flight.**
  - The pieces show «extraindo…»; the focused piece's primary spins and disables, and «Ignorar» on the focused piece disables.
  - A batch is sent one piece at a time, but every piece of it keeps «extraindo…» until the whole batch answers, and the outcomes land together.
  - The work continues if the operator leaves the screen.
- **Re-extraction in flight.**
  - A pronta piece being re-read at once leaves «Já extraídos» (its on-screen state becomes extraindo…) and appears under «Na doca» (counted by «Em voo»).
  - The focus jumps to the first piece still in view.
  - When the answer lands, the piece goes back to «Já extraídos», or stays under «Na doca» as falhou. (See Open questions.)
- **Outcome**, said in both channels:
  - Toasts. Success: «1 peça extraída — está em Já extraídos, a caminho da Revisão.» / «{n} peças extraídas — estão em Já extraídos, a caminho da Revisão.» Failure: «A extração falhou em 1 peça — ela continua na doca e diz o motivo.» / «A extração falhou em {n} peças — elas continuam na doca e dizem o motivo.» A mixed batch shows both.
  - Bell notices, posted by the app: «Extração concluída», which leads to Revisão, and «Uma extração falhou», which leads back here — both carrying the count. Counts are of **pieces**: a crop across two photos reports «2 peças extraídas».
- **Pronta with a warning.** After a successful re-extraction whose previous corrections could not be retired, the piece is «pronta» and its focus head shows the danger banner «Reextraído, mas as correções anteriores continuam sobrepostas ({erro}). Confira esta conta antes de aprovar.»
- **Piece errors** (the banner): MedLens's words verbatim, e.g. «No data extracted from document». When MedLens answered «erro» without a reason, the banner reads «o MedLens respondeu erro, sem dizer o motivo». When the file could not be read: «não consegui ler o documento: {erro}».
- **Refusals** (the failure strip, plus a danger toast when an extraction or crop send was refused). The texts are the app's, prefixed «doca: »:
  - «doca: sem cliente definido: {arquivos}. Escolha o adapter antes de enviar.»
  - «doca: escolha o adapter antes de enviar o recorte»
  - «doca: o recorte junta fotos de adapters diferentes ({a} e {b}) — duas fotos de clientes diferentes não são uma conta»
  - «doca: o recorte aponta para peças que não estão mais na doca: {ids}»
  - «doca: nenhuma peça no recorte»
  - «doca: o recorte não é um PNG — nada foi enviado»
  - «doca: o recorte tem {n} bytes, acima do limite de 10485760 — nada foi enviado»
  - «doca: o estado da doca está envenenado — reinicie o app»
  - «doca: a leitura da doca morreu antes de terminar»

  Disk failures read «{caminho}: {erro do sistema}».

- **Viewer states.** Loading and failure as in 4g — e.g. «Não consegui mostrar o documento: doca: a peça não está na doca». Switching piece resets zoom, rotation and pan.
- **Crop states.**
  - A PDF page being prepared: «Preparando a página para o recorte…».
  - A PDF this app cannot open for cropping shows in the pane: «Não consegui abrir este PDF para recortar — ele continua legível no visor, e dá para enviar a peça inteira.» The embed shows it again once the mode is left.
  - Composition failure (toast): «Não consegui compor o recorte: {motivo}».
- **Narrow window.** The window's minimum is 1024 × 768. The rail keeps its 398 px and the right panel absorbs the rest. The focus action row, the viewer strip and the crop bar do not wrap (unverified: how they render at the minimum width).

## Actions

| Gesture (where)                                                             | Precondition / disabled reason                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | Effect                                                                                                                                                                                                                                                                                                                                                                                                | Confirm · reversible                                                                                                         | Feedback                                                                     | Key                                           |
| --------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- | --------------------------------------------- |
| Reler a doca (header reload icon)                                           | —                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | Re-reads the spool and the watched folder. Then, with MedLens up, recognizes pieces MedLens already extracted, which turn «pronta».                                                                                                                                                                                                                                                                   | none · n/a                                                                                                                   | The list updates; the failure strip on refusal.                              | —                                             |
| Switch view (segment)                                                       | —                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | «Na doca» ↔ «Já extraídos»; clears the ticks.                                                                                                                                                                                                                                                                                                                                                         | none · yes                                                                                                                   | —                                                                            | —                                             |
| Type in the filter                                                          | —                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | Keeps pieces whose file name, shipment sender or subject contains the text (case-insensitive).                                                                                                                                                                                                                                                                                                        | — · clear with «×»                                                                                                           | List and pill counts recount.                                                | `Ctrl ⇧ K` focuses the field                  |
| Pill                                                                        | Drawn only while it counts, or while active.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   | Narrows the list to that state.                                                                                                                                                                                                                                                                                                                                                                       | — · click again                                                                                                              | —                                                                            | —                                             |
| Group (Agrupar menu)                                                        | —                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | Re-cuts the list's groups.                                                                                                                                                                                                                                                                                                                                                                            | — · pick again                                                                                                               | —                                                                            | —                                             |
| Focus a piece (click its card)                                              | —                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | Loads its document and resets the viewer. A half-drawn crop rectangle is dropped; the crop basket survives.                                                                                                                                                                                                                                                                                           | —                                                                                                                            | —                                                                            | `↑` / `↓`, stopping at the ends               |
| Tick (checkbox)                                                             | «Na doca» only.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | Adds the piece to the selection; the bar appears.                                                                                                                                                                                                                                                                                                                                                     | — · untick                                                                                                                   | Bar note                                                                     | —                                             |
| Choose adapter (focus menu)                                                 | The menu lists adapters only once MedLens answered its list in this run.                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | Durable choice for this piece. A sem cliente piece becomes aguarda extração; a falhou or pronta piece keeps its state (the adapter changes).                                                                                                                                                                                                                                                          | none · choose another (there is **no** "clear" item)                                                                         | The label updates; no toast.                                                 | —                                             |
| Choose adapter (bar «Cliente do lote…»)                                     | Same.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | The same, in one durable write for every ticked piece — ticked pieces hidden by the filter included. The ticks stay.                                                                                                                                                                                                                                                                                  | none · yes                                                                                                                   | No toast.                                                                    | —                                             |
| Ignorar (focus head)                                                        | Disabled while this piece is extracting.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | Durable: the piece leaves the dock for [Ignorados](05-discarded.md), and the focus moves to the first piece still in view. A pronta piece can be ignored too; its extraction is not undone.                                                                                                                                                                                                           | **none** · yes, «Restaurar» on Ignorados                                                                                     | The piece disappears; no toast.                                              | `Esc`                                         |
| Ignorar (bar)                                                               | —                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | The same, for every ticked piece (hidden ones included); the ticks are cleared.                                                                                                                                                                                                                                                                                                                       | none · yes                                                                                                                   | No toast.                                                                    | —                                             |
| Extrair (focus-head primary)                                                | Needs MedLens up, an adapter, and the piece not extracting. Otherwise disabled, with no tooltip.                                                                                                                                                                                                                                                                                                                                                                                                                                               | Sends the whole file; spends API. Retrying a falhou piece always asks MedLens afresh.                                                                                                                                                                                                                                                                                                                 | none · no (the spend is done)                                                                                                | «extraindo…» at once, then toasts and a bell notice.                         | `Enter`                                       |
| Reextrair (same button, on a pronta piece)                                  | Same.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | A forced fresh read. On success, the previous reading's corrections (made on Revisão) are retired.                                                                                                                                                                                                                                                                                                    | none · no                                                                                                                    | Same.                                                                        | `Enter`                                       |
| Extrair ({n}) (bar primary)                                                 | Disabled only while MedLens is not up.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         | Sends only the ticked pieces that are visible, have an adapter and are not in flight. The rest are skipped silently; if none qualifies, nothing happens (the ticks stay). Otherwise the ticks are cleared.                                                                                                                                                                                            | none · no                                                                                                                    | Same as above.                                                               | —                                             |
| Limpar seleção (bar «×»)                                                    | —                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | Clears the ticks.                                                                                                                                                                                                                                                                                                                                                                                     | —                                                                                                                            | —                                                                            | —                                             |
| Abrir (viewer strip)                                                        | —                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | Opens the **sent** file in the desktop's application — for an e-mail body that is the raw e-mail file, not the HTML shown here.                                                                                                                                                                                                                                                                       | none · —                                                                                                                     | None (a failure says nothing).                                               | —                                             |
| Zoom / fit / rotate / pan (zoom row, wheel, drag)                           | Photo; a PDF only in crop mode.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | View only — it never changes what is sent. The wheel zooms ±12 % per notch and the buttons ±25 %, from 5 % to 800 %. 100 % is one image pixel per screen pixel. The fit follows the pane until the first manual zoom. Drag-to-pan works only when zoomed past the fit. The middle button pans in both modes.                                                                                          | — · «NN%» refits                                                                                                             | —                                                                            | wheel; `Space`+drag in crop mode              |
| Recortar (viewer strip)                                                     | Photo or PDF. Disabled when the adapter reads only text — tooltip «{adapter} lê o texto do PDF ou do e-mail, não a imagem — recorte não serve para este cliente; envie a peça inteira.» Also while a PDF's page count is being read («Lendo o PDF…») or when the PDF cannot be opened (tooltip = the «Não consegui abrir este PDF para recortar…» sentence). Otherwise the tooltip is «Marcar regiões e enviar só elas».                                                                                                                       | Enters crop mode.                                                                                                                                                                                                                                                                                                                                                                                     | — · Cancelar recorte                                                                                                         | The button turns pressed.                                                    | —                                             |
| Extrair recorte (crop bar primary, and the focus-head primary in crop mode) | Enabled only with at least one region, exactly one adapter across the basket's pieces, none of them text-only, and nothing already sending. The tooltip says why before the click: «Marque ao menos uma região na imagem» / «Escolha o cliente / adapter antes» / «Fotos de adapters diferentes ({ids separados por « · »}) não compõem uma conta» / «{adapters} lê o texto do PDF ou do e-mail, não a imagem — recorte não serve para este cliente; envie a peça inteira.» When enabled: «Gasta API: envia ao MedLens só o que está marcado». | Composes one image and sends it once, for every piece in the basket. The basket's adapter becomes the durable choice of every basket piece. Once the send is accepted, crop mode ends and the basket empties — **even if MedLens then fails the reading** (retrying a crop means marking it again). On a refusal (the app's «doca: …» messages, or a composition failure), the basket stays as it is. | none · no                                                                                                                    | «Enviando…», then the pieces go «extraindo…», then toasts and a bell notice. | `Enter` (in crop mode)                        |
| Cancelar recorte (pressed strip button)                                     | —                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | Leaves crop mode and **throws the whole basket away**, including the other photos' and pages' regions.                                                                                                                                                                                                                                                                                                | **none**; the tooltip warns: «Sai do modo recorte», or «Descarta {n} marcadas», plus « em {m} fotos» when more than one · no | The mode ends.                                                               | `Esc` (in crop mode, same effect, no warning) |

### Keyboard

- `↑` / `↓` move the focus through the pieces in drawn order, across groups, stopping at the ends.
- `Enter` extracts, or re-extracts, the focused piece. In crop mode it sends the crop.
- `Esc` first closes an open adapter menu (focus or batch). Otherwise it ignores the focused piece — or, in crop mode, cancels the crop.
- `Space`, only in crop mode over a photo or PDF: held down, it is the hand tool. Elsewhere Space keeps its normal meaning, pressing the focused button.
- `Ctrl ⇧ K` focuses the filter.
- None of them act while the operator is typing in a field.

### Crop mode, in full

- **What it is for.** A report photo carries noise — a signature, a letterhead, or two labels of different clients in one picture. Cropping tells MedLens where to look.
- **Draw.** Press and drag with the left button over the image. The rectangle follows the pointer, clamped to the image. On release it lands only if both sides exceed 16 image pixels, so a plain click never leaves a region behind.
- **Move** by dragging inside a landed region; it stays inside the image. **Resize** by dragging a corner handle: the opposite corner stays put, and neither side can shrink below 16 px. **Remove** with the region's «×».
- **Pan.** Hold Space and drag, or drag with the middle button; both also work outside the mode. The wheel zooms in both modes. Outside the mode, a left drag pans a zoomed photo.
- **The basket crosses photos and pages.**
  - Focusing another piece, or walking to another page, keeps every landed region; only a half-drawn rectangle is dropped.
  - The basket and the mode itself also survive leaving the screen.
  - Only the regions of the piece and page on screen are drawn over the image. The rest show up as counts: the row badge, the pager suffix, the bar hint, the primary's count and the cancel tooltip.
- **Composition.**
  - All regions are stacked vertically into one image on a white background, with 16 px white gaps; the image is as wide as the widest region.
  - Order: the pieces' order in the dock (the whole dock, not the filtered view), then page, then top to bottom, then left to right — never the order the mouse drew them in. A piece that left the dock goes last.
  - A region's badge number is its place in this order.
  - The image travels to MedLens named «{first file} (crop).png», or «{first file} +{n} (crop).png» when it spans several pieces — the name Revisão lists.
- **PDF.**
  - Entering the mode swaps the native viewer for the current page, rendered as an image at 200 dpi — what MedLens itself reads at — capped at 6000 px on the longest side.
  - For a multi-page PDF the pager appears, and each page keeps its own regions. Walking pages keeps the zoom; switching piece resets it.
- **Rotation is view-only.** The composition uses the original, unrotated pixels.
- **Sending.**
  - Every basket piece shows «extraindo…».
  - A basket that holds a falhou piece forces MedLens to read afresh.
  - All the basket's pieces get the same answer: the same bill and item counts, and one extraction behind all of them.
  - A labels-only photo with no adapter inherits the basket's adapter.
  - Once the send is accepted, the mode ends and the basket empties — even when MedLens then fails the reading. A refusal keeps the basket.
  - What was sent is stored, so [Revisão](03-review.md) can show what the machine actually read, beside the original (see that file for its toggle).
- **E-mail bodies cannot be cropped** (no «Recortar»). If crop mode is on while a body piece is focused, the crop bar stays and the strip draws no crop button; only `Esc` or a successful send ends the mode.

## Business rules

1. **The unit of work is the piece (attachment), never the message.** A shipment of three attachments is three items, and two can move on while one is stuck. The shipment is provenance, drawn as a header. [docs/telas.md § /triage; decision 017]
2. **Two modes that do not compete.** The focused piece has its own controls, keyboard included; with pieces ticked, the bottom bar acts on the selection. [telas.md; 017 § 5]
3. **Exactly two states need a human: sem cliente and falhou.** Only they feed «Precisam de decisão», the subtitle and the shipment's warning icon. Amber means "a human decides here" and is never decoration. [telas.md; style manual]
4. **Absence must look like absence.** Nothing conclusive is drawn before the first read answers. An adapter nobody chose, or one the service did not describe, is **not** a refusal to crop. An unknown MedLens state is not a stopped one. [AGENTS.md; decision 019 § 10; `dock.svelte.ts`]
5. **A long command shows a sign of life before it leaves**: «extraindo…», the spinner, «Enviando…». [AGENTS.md › The machine]
6. **The disk rules existence; decisions are durable; only memory knows what is in flight.**
   - A piece exists because its file exists.
   - Ignore, adapter choice and extraction link are stored per piece, keyed by the file, never by position.
   - A re-read merges over memory, so an extraction under way never falls back to «sem cliente». [017 § 2]
7. **The local read never depends on MedLens.** A new delivery must show up even with MedLens down; recognizing already-extracted work is a courtesy that runs after it. [017 § 3; AGENTS.md "two parts, two tries"]
8. **A delivery or loose file arriving with the screen open shows up within about 4 s.** [017 § 3; 020 § 6]
9. **A batch is checked whole before any piece is marked**, so a refusal never leaves pieces stuck in «extraindo…». [017 § 4]
10. **Retrying a falhou piece forces a fresh read.** MedLens caches failures too. [017 § 7]
11. **Re-extraction forces a fresh read, and after a success retires the previous reading's corrections** — they described another reading. [017 § 4]
12. **An errored extraction is a failure, never a pronta piece with zero bills.** The service's words stay verbatim; the app's explanation is a separate second line, keyed on a fact (`reads_image`) and never on the wording. [017 § 7; 019 § 10]
13. **The outcome is said both ways.** A toast is feedback on the act; the bell holds a notice for whoever was not watching (an extraction lasts minutes). Both carry counts, never file names. [017 § 7; decision 010]
14. **A manifest's suggested adapter is a hint.**
    - It is adopted only onto pieces still undecided, and only when MedLens's last answered list knows the id.
    - It lives in memory only; a human choice always outranks it. [`dock.rs` › adopt; project memory]
15. **Hidden adapters** (Configurações → Adapters) filter only the choice. A piece already wearing one keeps its label, and what was extracted stays on Revisão. [decision 021]
16. **An e-mail body with no report table is shown, saying why — never hidden.** [017 § 1]
17. **What is shown is not always what is sent.** An e-mail body is shown as HTML, but what goes to MedLens (and what «Abrir» opens) is the raw e-mail file. [017 § 1]
18. **E-mail HTML is displayed in a sandbox that executes nothing.** [017 § 5]
19. **Cropping is a mode, not a gesture on the image; panning never belongs only to the left button; the wheel means zoom in both modes.** [019 § 1]
20. **Several regions become one image, in document order.** The basket crosses photos and pages, and the order never depends on the mouse. [019 §§ 2, 3, 9]
21. **One adapter per composition — the basket's, never the focused piece's.** Two adapters are two bills, and the send is refused, both on screen and in the backend. The elected adapter is stored for every basket piece. [019 § 4]
22. **In crop mode the primary button, `Enter` and `Esc` change target.** Nothing may send the whole file while regions are marked. [019 § 5]
23. **The basket's count is said everywhere the basket is not visible**: the primary, the bar, the row badge, the pager, the cancel tooltip. [019 § 3]
24. **Not every adapter reads pixels.** The crop is refused **before** the gesture, naming the client, and the fact comes from MedLens, never from a list kept here. [019 § 10]
25. **What was sent is kept**, so Revisão can show what the machine actually read. [019 § 6]
26. **Viewing never rewrites what is sent**: zoom and rotation are view-only. [`viewer.svelte.ts`; 019]
27. **Zoom resets per piece, not per PDF page.** [019 § 9]
28. **The watched folder is a second entrance with the same rules.**
    - Top-level PDF/PNG/JPG/JPEG/WEBP only, and no e-mail files.
    - A replaced file is a new document: the old decisions do not dress it.
    - It is named by i18n («Pasta vigiada»), never by the backend. [decision 020]
29. **Ignoring is durable and reversible from Ignorados, and does not undo an extraction already made.** [017; `lib.rs` › ignore_pieces]
30. **Selection exists only in «Na doca».** The bar acts on what still takes orders. [`+page.svelte`]

## Must survive the redesign

- The **piece row as the unit**, grouped under a quieter provenance header (sender · date · subject · count). The watched folder is named «Pasta vigiada», with its own icon, and rides last.
- The **six states with their labels and roles**. Sem cliente (amber) and falhou (danger) read as "a human is needed" at a glance — on the pill, on the row and on the group header. Nothing else is amber.
- **Pills drawn only while they count** (or while active), the «Precisam de decisão» union pill first, amber counts only on the human-needed pills.
- **The Na doca / Já extraídos split with both counts**, and the subtitle's decision and in-flight counts.
- **The two non-competing places**: focused-piece controls next to the document, and the ticked-selection bar with its own note «… · a barra age sobre a seleção».
- The **keyboard** (↑↓ / Enter / Esc) with a visible hint line that **changes wording in crop mode**.
- **Sign of life** («extraindo…», spinners, «Enviando…») the instant a long command starts.
- The **MedLens-down strip**: the dock stays readable, only extraction is gated, and the screen says why.
- **Failure words verbatim, with the app's second line visually separate** (never merged into one sentence), the **no-table** banner, and the "pronta with a warning" banner.
- **Not-asked-yet draws nothing conclusive**; three distinct empty sentences (dock, extracted, filter).
- **Document beside the list**: native PDF viewer for reading, sandboxed e-mail HTML, photo zoom/rotate/pan with the level as the way back to the fit.
- **Crop:**
  - a mode that is visibly on;
  - numbered regions (composition order), with delete and four corner handles;
  - counts wherever the basket is not visible;
  - the pan gesture announced while something is out of sight;
  - the disabled reason said before the click;
  - the one-adapter rule, and the text-only refusal naming the client;
  - no control that sends the whole file while regions are marked.
- dd/mm/aaaa dates; theme tokens only ([02-invariants](../02-invariants.md)).

## Free to redesign

- Rail width and card density, the tile icons, pill shape, the badge style, the look of the segment (tabs are fine, as long as it clearly switches views), and where the reload lives.
- The grouping control's form (menu, segmented control…). Keep the four cuts and «Por remessa» as the default.
- Banner, strip and key-hint styling; the zoom row's look; the pager's form.
- The crop bar's layout and position, provided the count and the send sit near the image. Region styling (keep the number, the delete and the handles). In crop mode there are currently **two** send buttons (focus head and crop bar); keeping one is fine.
- Whether the header shows the screen question.

## Sample content for mockups

Fictitious; «[!]» below marks the amber warning icon. Adapters (MedLens labels): «Hospital Vale Verde» (`vale_verde`), «Hospital Santa Aurora» (`santa_aurora`, a text-only adapter), «Instituto Ortopédico Horizonte» (`horizonte`), «Hospital Infantil Sol Nascente» (`sol_nascente`).

Header: «Doca de recebimento» · «4 aguardando decisão · 1 em extração» · «Na doca (6)» | «Já extraídos (1)». Pills in «Na doca»: «Precisam de decisão 4» · «Sem cliente 3» · «Falhou 1» · «Em voo 1» · «Aguardando extração 1».

| Group header                                                                                                         | Piece                                                                                                   | Meta                                    | State            |
| -------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------- | --------------------------------------- | ---------------- |
| [!] faturamento@valeverde.example · 02/09/2026 · 08:14 · Informe de consumo OPME — semana 36 · 3 anexos              | `informe_vale_verde_2026-09-02.pdf` (adapter suggested in MailBridgr's popup)                           | 1,2 MB · Hospital Vale Verde            | aguarda extração |
|                                                                                                                      | `etiquetas_sala3.jpg`                                                                                   | 2,4 MB                                  | sem cliente      |
|                                                                                                                      | `Informe de consumo OPME — semana 36` (e-mail body, no table → warn banner)                             | 18 KB                                   | sem cliente      |
| [!] contas.opme@santaaurora.example · 01/09/2026 · 17:40 · Consumo OPME — 01/09/2026 · 2 anexos                      | `foto_informe_santa_aurora.jpg` (banner: «No data extracted from document» + the text-only second line) | 3,1 MB · Hospital Santa Aurora          | falhou           |
|                                                                                                                      | `informe_santa_aurora_2026-09-01.pdf` (shown under «Já extraídos»)                                      | 1,1 MB · 6 contas · 41 itens            | pronta           |
| [!] Pasta vigiada (the warning outranks the folder icon) · 03/09/2026 · 16:40 · /home/camila/OPME/vigiada · 2 anexos | `nota_horizonte_2026-09-03.pdf`                                                                         | 640 KB · Instituto Ortopédico Horizonte | extraindo…       |
|                                                                                                                      | `etiquetas_sol_nascente.webp`                                                                           | 1,8 MB                                  | sem cliente      |

- **Crop scenario.** One region is marked over the header of `informe_vale_verde_2026-09-02.pdf` (page 1) and two over labels in `etiquetas_sala3.jpg`. The row badges read «1» and «2», the crop bar reads «3 regiões marcadas em 2 fotos · vão numa imagem só, de cima para baixo», and the primary reads «Extrair recorte (3 · 2 fotos)». The cancel tooltip reads «Descarta 3 marcadas em 2 fotos». The labels-only photo inherits «Hospital Vale Verde».
- **Refused crop tooltip:** «Fotos de adapters diferentes (vale_verde · santa_aurora) não compõem uma conta».
- **Text-only tooltip:** «Hospital Santa Aurora lê o texto do PDF ou do e-mail, não a imagem — recorte não serve para este cliente; envie a peça inteira.»
- **Toasts:** «2 peças extraídas — estão em Já extraídos, a caminho da Revisão.» · «A extração falhou em 1 peça — ela continua na doca e diz o motivo.»
- **Failure strip:** «doca: sem cliente definido: etiquetas_sala3.jpg. Escolha o adapter antes de enviar.»
- **Selection bar:** «2 peças selecionadas · a barra age sobre a seleção» · «Cliente do lote…» · «Ignorar» · «Extrair (2)».

## Design opportunities

Proposals, not decisions.

1. **Keep the focused row in sight.** `↑`/`↓` move the focus but the rail does not scroll with it, so walking a long dock by keyboard loses the focused card below the fold.
2. **Show a re-extraction in place.** Today the piece jumps from «Já extraídos» to «Na doca» and the focus moves to another document (see Open questions); an in-place «extraindo…» would keep the operator's context.
3. **Tell a suggestion from a decision.** A MailBridgr-suggested adapter looks exactly like a human choice. A quiet «sugerido» mark (cleared once the operator confirms or changes it) would say who decided.
4. **Explain a disabled «Extrair».** The reasons — MedLens down or unknown, no adapter, already in flight — are said nowhere near the button (only the MedLens strip, and not when the state is unknown).
5. **Honest batch counts.** «Extrair ({n})» counts every tick, but pieces without an adapter are skipped silently, and if none qualifies nothing happens. Say «{k} de {n} vão; {n−k} sem cliente», or disable with the reason.
6. **Make «Reextrair» weightier than «Extrair».** It spends API and retires the reviewer's corrections, and `Enter` triggers it on a pronta piece with no confirmation. At least a tooltip saying what it discards; possibly a confirmation.
7. **A safety net for the crop basket.** `Esc` or «Cancelar recorte» throws away regions across photos with no confirmation (the warning lives only in a tooltip). An undo toast, or a confirmation when the basket spans more than the photo on screen.
8. **A basket strip.** Region thumbnails in composition order, each removable, plus a preview of the composed image before sending. Today the only view of what will be sent is counts, until it reaches Revisão.
9. **A visible way out of crop mode on non-croppable pieces.** With an e-mail body focused, no crop button is drawn and only `Esc` or a send ends the mode.
10. **Say why the adapter menu is empty** when MedLens has not answered its list in this run, instead of an empty box. Mark raw ids shown in place of labels.
11. **One counting rule for group headers.** «Por remessa» counts pieces that are not drawn (the other view, the filter); the other groupings count what is drawn. Consider «1 de 3 anexos».
12. **Name the watched folder by its last path segment**, with the full path on hover; give loose files their own dates (the header shows only the newest).
13. **Word the kind in the focus head** («PDF», «foto», «corpo do e-mail») instead of MailBridgr's raw word.
14. **Feedback for «Abrir»** when the desktop cannot open the file.
15. **Keyboard for PDF pages in crop mode** (e.g. PgUp/PgDn) and hints for them. Today only the pager's arrows walk pages.
16. **Per-piece progress inside a batch** (all stay «extraindo…» until the whole batch answers). Needs progress events from the backend.
17. **Land on the found piece from Busca.** A search hit opens the dock without focusing the file that was found.
18. **Keep the marking when the reading fails.** A crop whose extraction MedLens then fails has already emptied the basket, so retrying means marking every region again on every photo. Keeping the last sent basket for a retry would save that work.

## Open questions

1. `docs/telas.md` says a piece being re-extracted "must not disappear from view while the work happens"; the build moves it out of «Já extraídos» for the duration. Which should the redesign draw?
2. Should `Enter` re-extract a pronta piece — spending API and retiring corrections — without asking?
3. Ignoring a pronta piece is allowed and its bills stay on Revisão. Should the screen say so, or refuse?
4. Should view, filter, pill and grouping persist when the operator leaves and returns? A code comment says the grouping "lives in the session", but it resets on every visit.
5. For a crop across several photos, toasts and the bell count **pieces** («2 peças extraídas») for **one** extraction. Which number should the operator read?
6. Is the manifest's suggested adapter meant to be distinguishable on screen (proposal 3), or deliberately silent?
