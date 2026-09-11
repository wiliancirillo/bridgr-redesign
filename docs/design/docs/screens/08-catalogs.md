# Catálogos — `/catalogs`

> Where: sidebar › Fora da esteira › Catálogos (Listas is its indented sub-item) · Status: built · Code: `src/routes/catalogs/+page.svelte`, `src/lib/catalogs.svelte.ts`, `src/lib/catalog-columns.ts`, `src/lib/components/CatalogSqlDialog.svelte`, `src/lib/sql-lint.ts`, `src/lib/components/GridSurface.svelte`, `src/lib/grid-view.svelte.ts`, `src/lib/csv.ts`, `src/lib/ipc.ts`, `src-tauri/src/registry.rs`, `src-tauri/src/lib.rs` (`list_catalogs`, `read_catalog`, `collect_catalog`, `collection_progress`, `catalog_sql`, `set_catalog_sql`, `catalog_layout`, `set_catalog_layout`), `src-tauri/src/error.rs`, `src-tauri/src/seal.rs`, `src/lib/i18n/locales/pt-BR.ts` (`catalogs`, `shortlists`, `sheets`, `query`, `grid`)

## The question

"Contra o que a Revisão confere?" (the subtitle under the title). The screen
answers three things about the ERP registries this machine holds: **which**
copies exist, **how old** each one is, and **what is in it**. It is also the
only place where a copy is refreshed (Coletar) and where the SELECT that fills
it can be read and edited. Each catalog adds its own review question under its
name (see [Data shown](#what-each-catalog-holds)).

## Where it sits in the work

- The ERP is the Everest. Its registries are copied here as **mirrors**,
  through the Fantasma's read-only backdoor (the robot types a SELECT into the
  Everest and exports the grid). A mirror rests sealed on this machine, because
  a doctor's name and CRM are personal data.
- **Who reads the mirrors.** Review checks the report header's insurer and
  doctor against Convênios and Médicos ([Revisão](03-review.md)); when that
  catalog was never collected, Review's field says "Este catálogo ainda não foi
  coletado — colete em Catálogos para ter contra o que casar. O campo segue
  livre." The adapter *colinha* dialog picks codes from Depósitos, Locais de
  uso, Comissionados, Tipos de pedido, Tabelas de preço and Condições de venda.
  The global search's "Códigos do ERP" group searches every collected mirror
  ([Busca](18-search.md)).
- **Doors in.** The sidebar; the Dashboard's Serviços card row "Catálogos"
  (state "espelho de {data} · {n} de 9 catálogos", or "sem espelho — colete
  quando o Fantasma estiver pronto") whose button "Abrir" opens this screen
  ([Dashboard](01-dashboard.md)); Listas' "Nova lista" button
  ([Listas](09-shortlists.md)); a code hit in the global search. Every door
  lands on **Materiais**: the screen always opens with it chosen, and a search
  hit from another catalog does not preselect that catalog.
- **Door out.** Ticked rows become a list (Salvar como lista), which then lives
  in [Listas](09-shortlists.md).
- **What a collection needs and takes.** The Fantasma must be running with its
  Everest session open (Configurações › Fantasma, [Configurações](10-settings.md)).
  While a collection runs it **owns** the Fantasma for minutes: a Consulta or
  the robot asking meanwhile is refused as busy, and the status bar's Fantasma
  block reads "ocupado" ([shell](../04-shell.md)).
- **The trail.** Each collection writes one line per page to Logs ›
  "Fantasma · Sessão" and a journal event to Logs › "Aplicativo"
  ([Logs](11-logs.md)). It writes nothing to Avisos.
- The mirror **suggests and never blocks**: with no mirror, no bill is stopped;
  only the checking and the suggesting stop.

## Anatomy

A header, then a two-column body **[functional]**: the rail of the nine
catalogs on the left, the chosen catalog on the right. No tabs, no footer.

### 1. Header

- Title "Catálogos"; subtitle "Contra o que a Revisão confere?".
- Right end — the **progress tag**, drawn only while a collection is in flight
  (started here or not): an accent pill with a turning ↻ icon and
  "{lidos} de {total} lidos" (e.g. "1.500 de 2.418 lidos"), or "{lidos} lidos"
  when the ERP's count failed. It does not name the catalog.

### 2. Rail (212 px wide, scrolls vertically)

- Kicker "Cadastros do ERP" (rendered uppercase).
- Nine entries in this fixed order, each `icon · name · count`:

  | Entry              | Icon (Phosphor)          |
  | ------------------ | ------------------------ |
  | Materiais          | `ph-package`             |
  | Médicos            | `ph-user-focus`          |
  | Convênios          | `ph-identification-card` |
  | Locais de uso      | `ph-buildings`           |
  | Depósitos          | `ph-warehouse`           |
  | Comissionados      | `ph-handshake`           |
  | Tipos de pedido    | `ph-list-dashes`         |
  | Tabelas de preço   | `ph-tag`                 |
  | Condições de venda | `ph-scroll`              |

- The chosen entry wears the accent tint. The name truncates with an ellipsis.
- The count is the number of items in the stored mirror, pt-BR thousands dot
  ("2.418"). ⚠️ **Blank — not "0" — for a catalog never collected**
  **[functional]**. While that catalog is being collected, its count turns
  accent (the number is still the stored mirror's, not the progress).
- There is no per-entry stamp or age on the rail.

### 3. Panel — drawn only once the list of catalogs has answered

**3a. Panel head** (one row):

- Left: catalog name (h2) and, beside it, a quiet **SQL icon** (`ph-file-sql`,
  22 px; tooltip and label "Ver o SQL da coleta") — only when the catalog has a
  collector. Under the name: the catalog's question, muted.
- Right, in order:
  - **[Copiar tudo]** (copy icon; the label becomes "Copiado" for about 1.6 s
    after a click) and **[Exportar CSV]** (download icon) — only when the
    catalog was collected **and** its mirror is in memory.
  - **[Coletar do ERP]** (↻ icon, the accent button) — only when the catalog has
    a collector **and** the screen has already asked the other side (see
    States › not asked yet). Its three faces:

    | Situation                          | Label                  | Enabled | Icon    | Tooltip                                                                                   |
    | ---------------------------------- | ---------------------- | ------- | ------- | ----------------------------------------------------------------------------------------- |
    | No collection in flight            | "Coletar do ERP"       | yes     | still   | "A coleta segura o Fantasma por alguns minutos — ninguém consulta enquanto ela roda."      |
    | This catalog is being collected    | "Coletando…"           | no      | turning | same as above                                                                             |
    | Another catalog is being collected | "Coletando Médicos…"   | no      | turning | "Coletando Médicos…"                                                                      |

**3b. Stamp line** — drawn whenever the catalog was ever collected
**[functional]**, never behind a tooltip:
"Lido em **08/09/2026 07:52:14** · 2.418 itens · everest/fantasma" — the date
and time in bold, the source (`everest/fantasma`, the collector that produced
the file) in monospace.

**3c. Save-as-list box** — an inline accent-bordered strip, drawn ABOVE the
grid **[functional]** while the box is open and at least one row is ticked:
text field (placeholder "Nome da lista", up to 360 px) · scope "{n} itens
marcados" · [Cancelar] · [Salvar] (accent; "Salvando…" while in flight;
disabled while the name is blank or a save runs). A failure line in danger ink
sits right under the strip. Never a dialog.

**3d. Content area** — exactly one of these, first match wins:

| # | When                                         | Drawn                                                                                                                                                                   |
| - | -------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1 | the store holds a refusal                    | the refusal text alone, danger ink                                                                                                                                      |
| 2 | the catalog has no collector                 | empty state `ph-wrench`: "O coletor deste catálogo ainda não foi escrito" / "O catálogo aparece aqui para dizer que existe — não para fingir que está vazio."            |
| 3 | never collected                              | empty state `ph-cloud-arrow-down`: "Este catálogo ainda não foi lido" / "Colete do ERP para ter contra o que conferir. A coleta segura o Fantasma inteiro por alguns minutos." |
| 4 | collected, not yet in memory                 | turning `ph-circle-notch` + "Abrindo o espelho…"                                                                                                                       |
| 5 | in memory                                    | the grid surface (below)                                                                                                                                                |

### 4. The grid surface

The same surface as the Consulta and the saved reports — the shared component
is described in [components](../05-components.md) (GridSurface, Grid,
ColumnsMenu, FilterField). This screen configures it as follows:

- **Tools row**: filter field (funnel, placeholder "Buscar por código, nome ou
  referência…", clear ✕ while text is typed, hint "Ctrl ⇧ K") · count
  "{n} itens" · Colunas popover · density pair (tooltips "Linhas confortáveis" /
  "Linhas compactas") · page size (25, 50, 100, 250; 50 on entry) + "por
  página".
- **Selection bar** (inside the framed box, over the grid, accent tint) while
  any row is ticked: "1 linha selecionada" / "{n} linhas selecionadas" ·
  "Marcar as {n} que o filtro achou" (only once the whole page is ticked and the
  filter holds more) · **[Salvar como lista]** (`ph-list-checks`) ·
  **[Exportar seleção]** (download) · spacer · [Limpar].
  ⚠️ **No row removal** — no delete button in the bar, no row actions
  **[functional]**.
- **Grid**: a tick column; sortable headers (tooltip "Ordenar por {coluna}");
  a resize grip on each header edge (tooltip "Arraste para redimensionar; duplo
  clique volta ao automático."); the first column (Código) in accent ink.
- **Pager**: "← Anterior · {primeira}–{última} de {total} · Próxima →". Nothing
  at its right end (no "ordenado por" on this screen).
- **Filter found nothing**: inside the box, the header row stays and the empty
  state (magnifier) says "Nada com o que você digitou".

### 5. The SQL dialog

A modal over a dark scrim (click on the scrim, ✕ or Esc closes it), up to
880 px wide and 84 % of the window high.

1. **Header**: "O SQL deste catálogo"; sub "{Catálogo} · É esta consulta que a
   coleta manda ao Everest, página por página."; at the right, the tag
   "EDITADO" (text "editado", uppercase) only when a hand-written SELECT is
   stored; the ✕ ("Fechar").
2. **Amber warning** (warning icon) — always, even before the SQL loads:
   "Editar aqui muda o que o catálogo traz. Um SELECT errado custa uma coleta
   inteira — e o filtro de classe é o que separa um espelho de 153 mil nomes de
   pacientes: ele não pode sair, e o app recusa se sair."
3. **What must stay** — "O que precisa continuar aí: `{after}` · as colunas
   `id` `descricao` …" — `{after}` and each required column as a monospace chip.
4. **Raw values line**: "⚠️ Os valores são os do ERP, não os da tela: SITUAÇÃO
   aparece como “ativo” aqui e é `A` no banco." (the backticks are literal
   characters of the string).
5. **Page query pane** **[functional]**: label "A CONSULTA DAS PÁGINAS" (text "A
   consulta das páginas") over a SQL editor that takes the remaining height —
   the Consulta's editor ([Consulta](06-query.md)): SQL colouring, keyword
   completion, soft wrap. The shipped SELECT arrives as one long line. When the
   editor is empty its placeholder reads "Ctrl+Enter".
6. **Count pane** **[functional]**: label "A CONTAGEM" (text "A contagem"), a
   short editor (up to 92 px), and the note "É ela que prova que a coleta chegou
   ao fim. Se você filtrar a consulta acima, filtre aqui também — senão a coleta
   é recusada por ter parado cedo." Both panes are visible together.
7. **Hints** — a list under the pair, drawn only when the text trips one; each
   line is a lightbulb (amber) + sentence, deduplicated across both editors:

   | Trigger                                            | Sentence                                                                                                                                     |
   | -------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
   | a double-quoted value after `=`, `<`, `>`, LIKE, IN | 'Aspas duplas são identificador no SQL Server, não texto — "ativo" vira nome de coluna e a consulta é recusada. Use aspas simples.'          |
   | `= NULL`, `<> NULL`, `!= NULL`                     | "Comparar com NULL (= NULL, <> NULL) nunca é verdadeiro: as linhas somem sem erro nenhum. Use IS NULL ou IS NOT NULL."                        |
   | a trailing `;`                                     | "Tire o ponto-e-vírgula do fim: a consulta de página é embrulhada para paginar, e ele cai no meio do embrulho."                               |
   | `BETWEEN 'aaaa-mm-dd' AND 'aaaa-mm-dd'`            | "BETWEEN com duas datas sem hora perde o último dia inteiro se a coluna for datetime (o limite vira meia-noite). Prefira >= data e < dia seguinte." |

8. **Failure line** (danger) — a refusal, verbatim (see States).
9. **Footer**: note "O espelho em disco continua o da coleta anterior; o novo
   SQL vale a partir da próxima." · spacer · **[Restaurar o original]**
   (counter-clockwise arrow; only when "editado") · **[Salvar o SQL]** (floppy
   icon, accent; disabled until either editor differs from what is stored, and
   while a save runs).

## Data shown

### What each catalog holds

| On screen          | id                  | Holds (ERP source)                                                                                         | Grid columns                                                | Question under the name                                             |
| ------------------ | ------------------- | ---------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------- | ------------------------------------------------------------------- |
| Materiais          | `materials`         | ERP items — with the shipped SELECT, only **active** items that carry a manufacturer reference              | Código · Descrição · ANVISA · Unidade · Situação · Código alt. | "Qual é o código do material que o informe chama por outro nome?"   |
| Médicos            | `doctors`           | registry entries of class MEDICO; alias = trade name; CRM from the registry's extension                     | Código · Descrição · Apelido · CRM · Situação                | "O médico do cabeçalho é o do cadastro?"                            |
| Convênios          | `insurers`          | registry entries of class CONVENIO (homonyms appear as separate rows)                                       | same five (CRM empty)                                       | "O convênio lido está escrito como o ERP o conhece?"                |
| Locais de uso      | `clients`           | registry entries of class LOCAL DE USO                                                                     | same five (CRM empty)                                       | "O hospital do informe é qual local de uso?"                        |
| Depósitos          | `deposits`          | the deposit table                                                                                          | Código · Descrição · Situação                                | "De qual depósito a conta movimenta?"                               |
| Comissionados      | `commission_agents` | commission holders, named from the registry                                                                | Código · Descrição                                           | "Qual comissionado assina o pedido deste cliente?"                  |
| Tipos de pedido    | `order_kinds`       | order kinds                                                                                                | Código · Descrição                                           | "Que tipo de pedido a conta gera?"                                  |
| Tabelas de preço   | `price_tables`      | price tables, one row per code (grouped across companies)                                                   | Código · Descrição                                           | "Por qual tabela o material é cobrado?"                             |
| Condições de venda | `sale_terms`        | sale terms                                                                                                 | Código · Descrição                                           | "Em que condição a conta é faturada?"                               |

The column set is the **catalog's**, fixed in code, not derived from which
fields happen to be filled — a column empty for a whole catalog (CRM on
Convênios) is still drawn, and hiding it is the operator's gesture.

### Field rules

| Field                          | Meaning                                         | Presentation rule                                                                                                                                                              | Example                                   |
| ------------------------------ | ----------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------- |
| Código                         | the ERP's key; it travels into queue documents  | exactly as the ERP spelled it — only trimmed, never re-cased, leading zeros kept; first column in accent ink                                                                    | `004512`, `015`                           |
| Descrição, Apelido, Unidade    | registry text                                   | normalized on entry: UPPERCASE, inner spaces collapsed, trimmed                                                                                                                 | DRA. HELENA CASTRO                        |
| ANVISA, CRM, Código alt.       | codes                                           | only trimmed, never re-cased                                                                                                                                                    | CRM-PR 00002                              |
| Situação                       | the ERP's status                                | three values: "ativo" (ERP `A`), "inativo" (ERP `I`), "—" (anything else). ⚠️ The third is **never** drawn as "inativo". The four support catalogs have no Situação column.       | ativo                                     |
| empty cell                     | the ERP sent nothing                            | a muted "—" whose tooltip is "célula vazia" (the unknown status is a literal "—" in normal ink)                                                                                  | CRM on Convênios                          |
| any column of digits only      | inferred over the **whole** mirror, not the page | right-aligned, monospace, tabular figures; sorts as numbers; the text itself is shown unchanged ("015" stays "015")                                                              | Código of Convênios                       |
| long cell                      | —                                               | capped at 280 px with an ellipsis; the cell's tooltip carries the full value; a dragged width overrides the cap                                                                  | a long material description               |

### Counters — what each number counts

| Number                           | Counts                                                                                                                                                                                                                         |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Rail count                       | items in that catalog's stored mirror; blank when never collected                                                                                                                                                              |
| Header tag "{lidos} de {total} lidos" | `lidos` = items gathered before the page now being read — it moves in steps of up to 500 (one page); `total` = the ERP's COUNT for the same filter, absent when that count failed. The tag vanishes when the collection ends, so the final number is never shown there |
| Stamp "{n} itens"                | items in the stored mirror (same as the rail)                                                                                                                                                                                  |
| Stamp "Lido em …"                | when the collection that produced the stored mirror **finished**, `dd/mm/aaaa hh:mm:ss`, local time                                                                                                                             |
| Grid count "{n} itens"           | rows matching the filter across the whole mirror (all rows when the filter is empty)                                                                                                                                           |
| Selection label                  | ticked rows, across every page and including rows the filter now hides                                                                                                                                                          |
| "{n} itens marcados"             | the same ticked rows                                                                                                                                                                                                           |
| "Marcar as {n} que o filtro achou" | rows the filter found                                                                                                                                                                                                         |
| Pager "{a}–{b} de {total}"       | position within the filtered rows (the pager's total has no thousands dot)                                                                                                                                                      |

## States

### The screen

| State | Trigger | What is drawn |
| ----- | ------- | ------------- |
| Not asked yet | first visit of the app session, before the list of catalogs answers | header; rail kicker with no entries; empty panel; no Coletar. ⚠️ Nothing conclusive — no "nunca lido", no empty grid, no inviting button. Later visits in the same session are born knowing (the list is kept in memory). |
| List refused | the list of catalogs failed (for example the wallet key is gone while mirror files remain) | on a first visit the panel has no catalog to draw in, so it stays as above and the refusal text is not shown; if a list had answered earlier in the session, the rail keeps it and the content area shows the refusal in danger ink |
| Chosen catalog never collected | no mirror file | rail count blank; head with name, SQL icon, question, Coletar; no stamp, no Copiar/Exportar; empty state #3 |
| No collector | a declared catalog without a collector | no SQL icon, no Coletar; empty state #2. Unreachable today — all nine declare one — but the state exists and is worded |
| Opening | mirror on disk, not yet in memory this session | stamp line visible; no Copiar/Exportar; "Abrindo o espelho…" with a turning icon. Once opened, a mirror stays in memory for the whole app session |
| Mirror refuses to open | the sealed file cannot be read | the refusal in danger ink, e.g. "selado: a chave do catálogo sumiu do chaveiro — os arquivos continuam, ilegíveis", "selado: a chave do catálogo no chaveiro está corrompida", "selado: o arquivo do catálogo está ilegível — o selo não confere", "selado: a leitura do catálogo morreu antes de terminar" |
| Loaded | — | stamp, Copiar tudo, Exportar CSV, Coletar, grid |
| Filter found nothing | filter typed | count "0 itens"; header row + "Nada com o que você digitou" inside the box; pager "0–0 de 0" |
| Switching catalog | rail click | the grid resets (filter, page, sort, columns, widths, selection), then that catalog's saved layout lands on top; page size is kept while the screen stays open |
| Narrow window | — | the page has no narrow-width rule: the rail keeps 212 px and the panel head stays one row; only the grid's tools row wraps ([components](../05-components.md)) |

### A collection

| State | Trigger | What is drawn |
| ----- | ------- | ------------- |
| Starting | Coletar clicked | ⚠️ the button turns "Coletando…" (disabled, turning icon) **before** the call leaves — the sign of life; the tag appears on the next poll (every 2 s while this screen is open) |
| In flight, this catalog chosen | — | button "Coletando…"; tag counting; rail count in accent; the grid and stamp still show the **previous** mirror; the SQL icon still opens |
| In flight, another catalog chosen | — | button "Coletando {Nome}…" disabled on every catalog |
| Left and came back | the operator navigated away mid-collection | the state survives: the button is still "Coletando…", the tag resumes on the next poll |
| Adopted | a collection this screen did not start is running (another window, a reloaded view) | learned at mount through the progress poll; drawn like "in flight"; when it ends the button returns and the stamp and count refresh **without a toast** |
| Succeeded | — | toast (ok) "Catálogo coletado."; the new mirror opens ("Abrindo o espelho…" briefly), stamp and rail count update; filter, sort and columns stay |
| Failed | — | toast (danger) "A coleta falhou: {mensagem}", or — when this catalog's SELECT is edited — "A coleta falhou e este catálogo está com SQL editado — comece pelo SQL. O Everest recusa a consulta com uma caixa de erro, e o Fantasma relata isso como problema de foco/tempo. Texto original: {mensagem}". ⚠️ **Nothing was written**: the previous mirror, count and stamp stay |
| Outcome while away | the collection ended with the operator on another screen | no toast there; the toast appears on the next visit to Catálogos |

`{mensagem}` is the other side's text, verbatim. The ones this app itself can produce:

| Cause | Message |
| ----- | ------- |
| Fantasma busy with another gesture | "Fantasma: o Fantasma está ocupado — aguarde o gesto atual terminar" |
| Everest session not open | "Fantasma: o Fantasma precisa estar com a sessão aberta para ler o cadastro — ligue em Configurações → Fantasma" |
| Fantasma not configured | "Fantasma: the Fantasma is not fully configured" (English in the current build) |
| ERP count is zero | "coleta: o ERP respondeu que não há NENHUM registro para {id}. Isso costuma ser um filtro que deixou de casar, e não um cadastro vazio — o espelho anterior foi preservado." (`{id}` is the English catalog id, e.g. `doctors`) |
| Empty page before the end | "coleta: a coleta parou em {lidos} de {total} itens: o ERP devolveu uma página vazia antes do fim do cadastro. Nada foi gravado — um espelho pela metade com carimbo de agora é pior que espelho nenhum. Tente de novo." |
| Result of another question | "coleta: o ERP devolveu outra tabela: esperava as colunas [{esperadas}] e vieram [{recebidas}]. Costuma ser o F5 que não pegou e a grade que ficou na consulta anterior. Nada foi gravado." |
| A tab inside a description | "coleta: a linha {n} do cadastro veio com {x} coluna(s) em vez de {y} — provavelmente uma descrição com tabulação dentro. A carga foi recusada inteira; nada foi gravado." |
| Paging stuck | "coleta: a coleta parou de avançar no id `{id}` — o ERP devolveu a mesma página duas vezes. Nada foi gravado." |
| A code unfit for a query | "coleta: o id `{id}` tem caractere que não cabe numa consulta ao ERP. A coleta parou aqui, e nada foi gravado." |
| Page ceiling | "coleta: a coleta passou de 60 páginas sem terminar. Nada foi gravado — é mais provável um defeito na paginação que um cadastro deste tamanho." |
| Died | "coleta: a coleta morreu antes de terminar — nada foi gravado" |
| The backdoor refused a page | the Fantasma's own words (unverified: that text lives in the Fantasma's repository) |

⚠️ **Fantasma off, unconfigured, or busy with something that is not a
collection**: nothing on this screen shows it beforehand. Coletar stays
enabled, and the click ends in one of the refusals above. Only a collection in
flight greys the button.

### The SQL dialog

| State | What is drawn |
| ----- | ------------- |
| Loading | header + amber warning only (no editors, no spinner) |
| Could not load | header + warning + the refusal line |
| Pristine | editors filled with what the **next** collection will run (the stored hand-written pair, else the shipped one); "EDITADO" only when a hand-written pair is stored; [Salvar o SQL] disabled |
| Edited draft | [Salvar o SQL] enabled; hints appear or vanish as the text changes |
| Saving | both footer buttons disabled |
| Saved | toast (ok) "SQL salvo — vale na próxima coleta."; the dialog stays open, re-read from disk, now "EDITADO"; the mirror on disk is untouched |
| Restored | toast (ok) "SQL original restaurado."; editors show the shipped pair; the tag goes away |
| Refused | danger line "O SQL foi recusado: {motivo}"; nothing stored |
| Closed with a draft | the draft is discarded without a question |

The refusals (`{motivo}`, verbatim as the current build renders them — the
English fragment comes from the other side's wrapper):

1. "the catalog was refused: cadastro de paciente não é coletável, por nenhum caminho" — the word PACIENTE anywhere, in either query.
2. "the catalog was refused: uma consulta a dbo.Cadastro precisa filtrar a classe (cl.nome = '…') ou vir por dbo.Comissionado: sem isso ela devolve 153 mil nomes de pacientes" — either query.
3. "the catalog was refused: o SQL precisa manter `{after}` — é por ele que a coleta continua da última página, e sem ele ela repete a primeira até parar" — page query.
4. "the catalog was refused: o SQL precisa continuar devolvendo a coluna `{coluna}` — é pelo cabeçalho que a coleta prova que o resultado é desta pergunta" — page query, first missing column.
5. The count must be a count: "the catalog was refused: a contagem precisa ser um COUNT — é ela que prova que a coleta chegou ao fim, e sem isso um espelho pela metade passaria por inteiro".

## Actions

| Gesture | Where | Preconditions / disabled reason | Effect | Confirmation | Reversible | Feedback | Shortcut |
| ------- | ----- | ------------------------------- | ------ | ------------ | ---------- | -------- | -------- |
| Choose a catalog | rail entry | — | panel shows it; grid resets then its saved layout applies; the mirror opens if not in memory | no | yes | entry highlighted | — |
| Coletar do ERP | panel head | the catalog has a collector; the screen has asked; no collection in flight (else disabled, label says which is running) | reads the whole registry page by page through the Fantasma, holding it for minutes; on proven success replaces the sealed mirror and its stamp | no | no (the previous mirror is replaced; nothing is written on failure) | label + tag + rail tint; toast at the end | — |
| Ver o SQL da coleta | SQL icon | the catalog has a collector | opens the dialog | — | — | — | Esc / ✕ / scrim closes |
| Salvar o SQL | dialog footer | a draft differs from what is stored | stores this catalog's page + count pair; used from the next collection on | no | yes (Restaurar o original) | toast; "EDITADO" appears; or the refusal line | — |
| Restaurar o original | dialog footer | the catalog is "editado" | removes the stored pair at once | no | by re-typing | toast | — |
| Filter | tools row | — | case-insensitive substring over the **visible** columns; back to page 1 | — | clear ✕ | count updates | Ctrl ⇧ K |
| Sort / columns / resize / density / page size | grid, Colunas popover, tools row | — | as in [components](../05-components.md); order, hidden columns, sort and widths are **saved per catalog automatically** and survive restarts and re-collections; "Desfazer edições" returns to the shipped shape and clears the saved entry; filter, page, page size and selection are not saved | no | yes | — | — |
| Tick rows | tick column, header tick, row | — | selection by key; Shift+click extends a range, Ctrl/⌘+click toggles one; header tick = this page ("Marcar os desta página" / "Desmarcar os desta página", tooltip "Marca só o que está nesta página.") | — | [Limpar] | selection bar | — |
| Salvar como lista | selection bar | ≥ 1 row ticked | toggles the inline box above the grid; Salvar (or Enter in the field) stores a new list: the ticked codes in on-screen order, plus the grid's current column order, hidden columns, sort and widths, unpinned | no | the list can be deleted in Listas | toast "Lista salva."; box closes, name cleared, selection cleared; failure line under the box. The screen stays here | Enter |
| Exportar seleção | selection bar | ≥ 1 ticked; not while an export dialog is open | save dialog "Exportar o recorte como CSV", filter "Planilha CSV", name suggested "{Catálogo}-selecao.csv"; writes the ticked rows (hidden-by-filter ones included), visible columns in screen order and sort | the save dialog | — | toast "CSV exportado — o arquivo sai do app sem cifra."; nothing on cancel; "Não consegui gravar o CSV: {mensagem}" on failure | — |
| Exportar CSV | panel head | mirror in memory | same dialog, name "{Catálogo}.csv"; writes the **filtered** rows | the save dialog | — | same toasts | — |
| Copiar tudo | panel head | mirror in memory | copies the header and the **filtered** rows, tab-separated, to the clipboard | no | — | label "Copiado" ~1.6 s | — |

The CSV uses `;` as separator and CRLF line ends, and a cell starting with `=`,
`+`, `-` or `@` is prefixed so a spreadsheet does not run it as a formula.

## Business rules

1. **Collecting is a manual gesture, one catalog at a time, never a timer.** It
   holds the Fantasma's single-flight lock from the first page to the last;
   releasing it midway would let another request kill the collection after all
   the work. (decision 015, `registry.rs` header, `lib.rs` `collect_catalog`)
2. **Nothing is stored unless the collection proved it finished.** The ERP's
   count comes first; an empty page before that count is a failure; a count of
   zero is a filter that stopped matching and preserves the previous mirror; 60
   pages is the brake. A half mirror stamped "now" is worse than the older whole
   one. (`registry.rs` `really_finished`, `collect_paged`)
3. **The stamp is in sight, always.** A mirror ages, and an old mirror
   presented as current makes the error invisible. (`catalogs/+page.svelte`, `ipc.ts` `CatalogSummary`)
4. **Absences are told apart**: never collected (blank count + its sentence),
   no collector (its sentence), opening (sign of life), filter found nothing
   (inside the grid). A zero is never drawn for a catalog never read — "0 itens"
   would say the ERP answered an empty registry.
5. **The screen asserts nothing before it has asked**, and the collection in
   flight and its outcome live outside the screen, so leaving and coming back
   neither loses a running collection nor invites a click that could only be
   refused. (decision 015 § 6b, `catalogs.svelte.ts`)
6. **Status has three values**; what the ERP said and this side could not read
   is "—", never "inativo". ("Absence must look like absence", Bridgr Pro `AGENTS.md`)
7. **Normalize on entry, never the key.** Text goes uppercase and collapsed;
   codes are only trimmed — re-casing a code invents one the ERP does not have,
   and the error surfaces far away, in the order. (`registry.rs` `normalized`)
8. **The registry table holds patients.** Doctors, insurers and sites of use
   are read by class **name** from an allow-list; PACIENTE is forbidden by name;
   a hand-written SELECT is refused if it reads that table without the class
   filter (or the commission-holder join), loses `{after}`, or drops a required
   column; a count must be a COUNT. The dialog's warning is not the defence —
   the refusal is. (`registry.rs` `check_custom_sql`, `check_patient_rule`, `check_custom_total`)
9. **The dialog shows what the next collection will run** — the stored pair
   when there is one, the shipped one otherwise — and a collection uses the pair
   stored when it **started**; an edit mid-run changes nothing until the next.
   (`lib.rs` `catalog_sql`, `run_collection`)
10. **A page and its count are one question.** Both are edited together;
    narrowing one without the other gets the collection refused for stopping
    early. (decision 015 § 6d, § 8b)
11. **Hints warn, never block.** Who validates a SELECT is the Fantasma; a
    second gate here would be a second, staler source of truth. (`sql-lint.ts`)
12. **The screen's words are not the ERP's** ("ativo" is `A` in the database),
    and the dialog says so before anything is typed.
13. **A failure on an edited catalog names the SQL first** — from this side's
    own fact (the catalog is edited), never from reading the robot's sentence.
    (`catalogs.svelte.ts` `CollectOutcome.edited`)
14. **No row removal on the mirror.** The mirror is what the ERP said; editing
    it in place would be erased by the next collection. Keeping a cut is a
    list, stored outside the mirror. (decision 016 § 2)
15. **A list is always a selection** — the gesture exists only while rows are
    ticked, so a whole catalog cannot become a "list" by accident.
    (decision 016 § 7)
16. **Columns are the catalog's shape and only grow by appending**: lists store
    their cut as positions in this column list. (`catalog-columns.ts`)
17. **The layout is remembered per catalog, by column identity**, and a
    collection refreshes data, never columns. (decision 015 § 9, decision 026)
18. **The mirror rests sealed with its own key**; CSV export is the deliberate
    hole, taken at a save dialog each time, at a path the operator picks.
    (`registry.rs` `store`, `csv.ts`)
19. **Ticked rows come from the whole mirror**, never only the filtered ones,
    so the number in the bar is the number acted on. (`grid-view.svelte.ts` `selectedRows`)
20. **Price-table codes are text** — `015` is not `15` — and codes are shown as
    stored. (Bridgr Pro `AGENTS.md`)

## Must survive the redesign

- The **stamp with date and time**, visible without hovering, for the chosen
  catalog — and a way to tell a never-collected catalog from a collected one at
  a glance on the list of nine, without drawing "0".
- The four absences as four different pictures and sentences (never collected,
  no collector, opening, filter found nothing), plus the not-asked-yet state
  that draws nothing conclusive.
- **Coletar is one catalog, one click**: no "collect all", no timer, no
  automatic refresh; greyed with the running catalog's name while any
  collection runs; the sign of life appears before the call; the running state
  and the outcome survive leaving the screen.
- Visible progress while collecting.
- A failed collection says **nothing was written**, keeps the previous mirror
  and stamp on screen, and shows the other side's words verbatim; the
  edited-SQL variant names the SQL first.
- Status in three values; "—" never collapses into "inativo".
- Codes drawn exactly as stored (no re-casing, leading zeros kept).
- The SQL dialog: the consequence warning before any edit, the page and the
  count visible and editable together, the list of what must stay (`{after}` +
  columns), the raw-values warning, hints that never disable Save, refusals
  shown verbatim, the "editado" marker, "vale na próxima coleta", and a way back
  to the shipped SQL.
- No delete, edit or prune gesture on the catalog grid; "Salvar como lista"
  exists only with a selection, and its name box sits next to the selection,
  never past the end of a long page.
- Export only through a save dialog, with the "sai do app sem cifra" warning.
- Switching catalogs clears the filter and the selection (a filter typed for
  one catalog silently emptying the next would look like a broken mirror); the
  per-catalog layout is kept.

## Free to redesign

- The rail as such (it could be tabs, a list, a segmented header) — as long as
  all nine are visible at once with their state.
- Icons, the look of the progress tag, where Copiar/Exportar/Coletar sit, the
  size and position of the SQL icon.
- The stamp's typography, and whether the source label is shown.
- The dialog's layout and proportions, the hint list's look, the tag's look.
- The empty-state illustrations and the grid's colours within the tokens.

## Sample content for mockups

**Rail** (fictitious counts): Materiais 2.418 · Médicos 312 · Convênios 58 ·
Locais de uso 41 · Depósitos 9 · Comissionados 21 · Tipos de pedido *(blank —
never collected)* · Tabelas de preço 38 · Condições de venda 15.

**Stamp**: "Lido em 08/09/2026 07:52:14 · 2.418 itens · everest/fantasma".
**In flight**: tag "1.500 de 2.418 lidos"; on another catalog, "Coletando
Materiais…".

**Materiais** (Código · Descrição · ANVISA · Unidade · Situação · Código alt.;
with the shipped SELECT every row is "ativo"):

| Código | Descrição                                | ANVISA      | Unidade | Situação | Código alt. |
| ------ | ---------------------------------------- | ----------- | ------- | -------- | ----------- |
| 10245  | PARAFUSO CORTICAL 3,5 MM X 24 MM TITÂNIO | 80000000011 | UN      | ativo    | FAB-3524    |
| 10311  | PLACA BLOQUEADA 3,5 MM 6 FUROS           | 80000000012 | UN      | ativo    | FAB-3506    |
| 10877  | CIMENTO ÓSSEO 40 G                       | 80000000013 | UN      | ativo    | —           |
| 10955  | FIO DE KIRSCHNER 1,6 MM                  | 80000000014 | UN      | ativo    | FAB-1616    |

**Médicos**: 004512 · DR. RAFAEL NOGUEIRA · RAFAEL NOGUEIRA · CRM-PR 00001 ·
ativo; 004533 · DRA. HELENA CASTRO · HELENA CASTRO · CRM-PR 00002 · ativo.
(DR. OTÁVIO PIRES is not in the mirror.)

**Convênios** (CRM column empty — muted "—"): 000001 · SUS · ativo; 000002 ·
PARTICULAR · ativo; 000388 · AMPARO SAÚDE · ativo; 000391 · AMPARO SAÚDE ·
inativo; 000412 · VIDA PLENA SAÚDE · "—" (status the ERP sent and this side does
not recognize).

**Locais de uso**: 002101 · HOSPITAL VALE VERDE · VALE VERDE; 002102 · HOSPITAL
SANTA AURORA; 002103 · INSTITUTO ORTOPÉDICO HORIZONTE; 002104 · HOSPITAL
INFANTIL SOL NASCENTE.

**Depósitos**: 01 · MATRIZ · ativo; 04 · CONSIGNADO VALE VERDE · ativo.
**Tabelas de preço**: 015 · TABELA OPME PADRÃO. **Tipos de pedido**: 3 · VENDA
DIRETA; 5 · REPOSIÇÃO DE CONSIGNADO. **Condições de venda**: 01 · À VISTA;
30 · 30 DIAS. **Comissionados**: 003310 · REPRESENTAÇÕES EXEMPLO LTDA.

**SQL dialog, Médicos** — page query (one soft-wrapped line):

```sql
SELECT TOP 500 c.cd_cadastro AS id, RTRIM(LTRIM(c.razao)) AS descricao, RTRIM(LTRIM(c.fantasia)) AS apelido, RTRIM(LTRIM(ISNULL(e.cd_crm, ''))) AS registro, RTRIM(LTRIM(c.situacao)) AS situacao, '.' AS fim FROM dbo.Cadastro c INNER JOIN dbo.CadastroClasse cc ON cc.cd_cadastro = c.cd_cadastro INNER JOIN dbo.Classe cl ON cl.cd_classe = cc.cd_classe LEFT JOIN dbo.cadastroEXT e ON e.cd_cadastro = c.cd_cadastro WHERE c.cd_cadastro > '{after}' AND cl.nome = 'MEDICO' ORDER BY c.cd_cadastro
```

Count: `SELECT COUNT(*) AS total FROM dbo.Cadastro c INNER JOIN dbo.CadastroClasse cc ON cc.cd_cadastro = c.cd_cadastro INNER JOIN dbo.Classe cl ON cl.cd_classe = cc.cd_classe WHERE cl.nome = 'MEDICO'`.
Keep line: "O que precisa continuar aí: `{after}` · as colunas `id` `descricao`
`apelido` `registro` `situacao` `fim`". For Materiais the columns are `id`
`descricao` `unidade` `situacao` `anvisa` `codigo_alt` `fim`.

**Toasts**: "Catálogo coletado." · "A coleta falhou: Fantasma: o Fantasma está
ocupado — aguarde o gesto atual terminar" · "Lista salva." · "CSV exportado — o
arquivo sai do app sem cifra."

## Design opportunities

1. **Name the catalog and the pace in the progress tag.** The tag says
   "1.500 de 2.418 lidos" without saying which catalog, and the progress also
   carries the page number and the start time, which nothing draws. Reason: a
   collection lasts minutes; "Materiais · página 4 · 3 min" answers "is it
   stuck?".
2. **Show a running collection outside this screen.** Today its only trace
   elsewhere is the Fantasma reading "ocupado"; the outcome toast waits for a
   return to Catálogos, and no notice is written. Reason: the collection
   survives navigation, but the operator's attention does not.
3. **Show the Fantasma's readiness beside Coletar.** The button cannot tell
   that the Fantasma is off, unconfigured or busy with a Consulta; the first
   sign is a refusal toast. Reason: an inviting button over a refusal is the
   exact trap this screen was built to avoid.
4. **Give the stamp an age and a tone** ("há 9 dias"), and bring it to the
   rail. Reason: the rule is that an old mirror must not pass for a current
   one; an age reads faster than a date.
5. **Make "never collected" explicit on the rail** (e.g. a quiet "não lido"
   chip) instead of a blank. Reason: the blank is correct but easy to miss; a
   word that is not a number keeps the rule.
6. **Give a refused list of catalogs a place to speak**, independent of a
   chosen catalog. Reason: today the panel needs a catalog to draw text in.
7. **Open the new list from the "Lista salva." moment** (a link in the toast,
   or a way to Listas from this screen). Reason: after saving, the list lives on
   a screen the operator must now find in the sidebar.
8. **Land a search hit on its catalog, filtered to the code.** Reason: a
   doctor's code hit lands on Materiais today.
9. **Guard the SQL draft**: ask before closing with unsaved edits, show a diff
   against the shipped SQL, and drop the "Ctrl+Enter" placeholder in the page
   editor (nothing is bound to it in this dialog). Reason: the dialog edits a
   rule that costs minutes of the Fantasma when wrong.
10. **Say which layout is saved** (a quiet "layout salvo" with a reset) and add
    the pager's "ordenado por" that Listas already shows. Reason: the layout
    persists silently, and the two catalog grids should read alike.

## Open questions

- The filter's placeholder says "Buscar por código, nome ou referência…", but
  no catalog draws a Referência column today (Materiais shows ANVISA in that
  place) and the filter only searches visible columns. Keep, or reword? (A
  wording change is the operator's call.)
- The selection bar says "linhas" ("3 linhas selecionadas") where the rest of
  the screen says "itens" ("3 itens marcados", "2.418 itens"). One noun?
- Some refusals arrive with English fragments ("the catalog was refused:",
  "the Fantasma is not fully configured") and an English catalog id inside a
  Portuguese sentence ("para doctors"). Draw them as the app renders them
  today, or propose pt-BR text?
- Should the screen remember the last catalog chosen instead of always opening
  on Materiais?
