repo: alatarithron/bridgr_reborn
branch: main
path: docs/
output: design/

## Last sync

date: 2026-09-12T05:55:00Z

### Updated in this project

- Rodada 37: em vez de abrir uma terceira exceção de tipo, o deck passou a declarar **duas camadas** — *interface* (tudo que compõe uma tela ou componente, e obedece aos pisos) e *anotação* (kicker da página a 11px, títulos de seção, notas e as legendas de amostra a 9,5px mono, que descrevem o desenho e não vão para o app). Com isso os 34 monos de 9,5px e o kicker de 11px deixam de furar regra: não são interface. A regra que sobra é uma — nenhum número da camada de anotação aparece dentro de um artboard.
- Rodada 36: o passe do sino não havia pegado o próprio arquivo de referência — corrigidos os três sinos de Tokens e Shell (o escuro perdeu o cráchá numérico, o claro ganhou o ponto com anel branco, o do artboard 960 ganhou caixa e alvo de clique). E a **regra dos pisos de tipo passou a dizer a verdade**: três tamanhos de caixa-alta (9,5 rótulo de cartão · 10 kicker de seção · 10,5 cabeçalho de grade) e duas exceções declaradas — monograma (9px) e papel (8px).
- Rodada 35 (varredura fina): **o sino virou caixa 27×25 com ponto de 7px no canto** em todas as 22 telas — o cráchá numérico engolia o ícone, e a contagem já vive no painel e na alça; **piso do mono de interface em 10px** (chips de atalho estavam a 9); **rótulo em caixa-alta reduzido a dois tamanhos** (9,5 em cartão, 10,5 em grade); **um neutro claro só** (#eceef0 → #eceef1); botões de janela e avatar dos artboards estreitos igualados aos das telas cheias.
- **Paleta de papel declarada** em Tokens e Shell, com a exceção de tipo a 8,5px escrita: no visualizador de informe o tipo é reprodução de documento, não texto de interface.
- Rodada 34 (família 4, mecânica): **crachás da barra lateral unificados** em 14 arquivos — Revisão 9 · Faturamento 10 · Duplicidades 2, e Ignorados sem crachá (peça posta de lado não espera ninguém); os números são os do Fechamento de setembro no dia 12, ou seja o mesmo instante da alça da bandeja.
- **`NFePackMonitor` → `NFe Pack Monitor`** em todo texto de tela (o nome do arquivo segue igual, para não quebrar o mapa do repo).
- Regra **"Um instante"** escrita na folha de porménores do Tokens e Shell: o deck mostra um momento só, e por isso todo número de casca é igual em todas as telas.
- Rodada 33 (varredura de consistência): criada a folha **`docs/06-sample-data-reborn.md`** — faixas de número (atendimento, pedido, nota, lote, material) com a regra raiz *nenhum número aparece como duas coisas*, as seis contas canônicas com cliente e convênio separados, a duplicidade da Marta e as quatro notas.
- Aplicado: nota 100246 passou a ser do Hospital Vale Verde no NFePackMonitor (era Santa Aurora); Marta virou Santa Aurora na Busca; o atendimento certo da leitura A da Marta em Duplicidades; `0000451288` saiu da coluna LOTE em Relatórios; o uso do João virou 02/09 na Revisão; e os cartões da trilha da Revisão passaram a mostrar **cliente**, não convênio.
- O `≥` do Estoque virou **`?`**: venda sem remessa dá piso zero, não dois — o estado passou a ser *em mãos desconhecido*. Bloco claro recuperou Fatur. e Devol., que ensinavam a subtração errada.
- Rodada 32: **Estoque em consignação** — a continuação do Espelho das notas, lendo os XMLs como movimento. Em mãos = remetido − faturado − devolvido (CFOPs); livre = em mãos − alocado, onde **alocado** são os lotes em pedidos deste app aguardando faturamento — reserva, não saída. Trilha por lote com saldo corrente, ponto vazado e crachá "registro do app" no que não é documento fiscal; saldo negativo, saldo com piso, movimento cancelado e lote zerado. Entrou na navegação e ganhou docs/screens/25-consignment-stock.md.
- Rodada 31: **Espelho das notas** — tela nova, o segundo espelho, montado dos XMLs que o NFe Pack já baixa e **sem passar pelo Fantasma**. Materiais com lotes, validade, quantidade e cliente que recebeu; a coluna **contra o espelho** (casa / difere / não está no espelho) que nunca escolhe um lado; a tabela dos dois espelhos dizendo onde as notas substituem o Everest e onde não; cobertura sempre escrita, nota cancelada riscada e não apagada, e as recusas de XML contadas para o total fechar.
- Entrou na barra lateral das 18 telas no mesmo movimento, nos dois temas, e ganhou `docs/screens/24-notes-mirror.md`.
- Rodada 30: **Doca** (pasta vigiada, o que fazer com o arquivo aceito, a verdade de que ninguém vigia com o app fechado, as três recusas), **Exportação** (formato, separador/codificação/decimal, planilha de registro travada como meio-desfecho, "gerar um exemplo") e a **página de intenção** dos dois estudos restantes — Inpart Saúde e Costantini.
- **Duplicidades era um dos três estudos do doc 14** e virou tela nesta sequência: saiu do grupo "Estudos" e perdeu o crachá, como manda a regra 4. A pergunta aberta do doc (quem mantém as marcas de dependência) foi respondida: a marca carrega **quem conferiu e quando**, e existe um terceiro estado — meio-pronto.
- Rodada 29: **janela estreita nas três telas novas** (960×540 cada, com a cessão própria de cada uma); **os três prazos decididos** e escritos num lugar só em Tokens e Shell (âmbar 7 d, vermelho 15 d, desfecho velho 3 d); **`box-sizing` virou regra escrita** (a página é content-box; bloco novo com width+padding declara border-box).
- **Cinco especificações novas em `docs/screens/`** — 19-work-tray, 20-duplicates, 21-bill-trail, 22-corrections, 23-period-close — no estilo e na língua dos docs existentes, marcadas `Status: proposed`, para o inventário do repo voltar a ser a fonte única.
- Rodada 28 (corre\u00e7\u00f5es do checkup): a aritm\u00e9tica do funil do Fechamento passou a fechar (68 \u2212 9 \u2212 5 \u2212 3 = **51** aprovadas; 51 \u2212 10 = 41 faturadas; 24 em aberto = 14 antes da aprova\u00e7\u00e3o + 10 depois, com a composi\u00e7\u00e3o escrita no subt\u00edtulo); **Fechamento e Corre\u00e7\u00f5es entraram na barra lateral de todas as telas** (Fechamento ao lado do Dashboard, Corre\u00e7\u00f5es no fim de \"Fora da esteira\"), nos dois temas.

- Rodada 27: **Fechamento do período** — tela nova: funil do período, as contas em aberto nomeadas uma por uma com "parada há" como ordem padrão, abas por cliente e por convênio, mês fechado, desfechos velhos com rechecagem (que segura o Fantasma), mês vazio e mês limpo. Conta **contas, não dinheiro**, e o período é pela **data de uso**.
- Três telas novas propostas nesta sequência (Trilha da conta, Correções, Fechamento), todas com tema claro e estados.
- Rodada 26: **Correções** — tela nova: adapters ranqueados por taxa (campo sobre campo), campos ranqueados com os padrões literais agrupados, e as duas portas de conserto (colinha ou exemplo no adapter). Estados: MedLens calado com a segunda fonte rotulada, adapter limpo (tique apagado, nunca verde), adapter sem leitura, a faixa âmbar de gravar na colinha com os **dois desfechos ditos como são**, e as correções aposentadas por reextração.
- Rodada 25: **Trilha da conta** — tela nova, fora do inventário: a vida de uma conta numa página (ator antes do gesto, corrida lote por lote, correções campo a campo), com a conta nova de duas linhas, o buraco da corrida sem desfecho desenhado como buraco, a reextração que aposenta correções e as recusas do próprio registro. Só lê; pede do backend o id da conta em cada linha do diário.
- Rodada 24: "ontem" é sempre o dia do calendário (regra aprovada); **Logs guarda só erro** nas quatro fontes de serviço e a linha de INFO vive no console do próprio serviço — a Busca agora diz qual das duas portas é a de cada acerto de log; **"Assinar como resolvida" passou a existir na conta provável** (investigação não tem falha registrada, e era a falha que fazia o botão aparecer).
- Rodada 23 (pormenores, a partir de `insights/project-insights.md`): sintaxe do SQL deixou de usar verde/âmbar (papéis de estado) — violeta/ciano/tinta comum, vermelho só no comentário; separador de milhar no paginador; par do Spook corrigido para **pasta ou endereço** + credencial (endereço vazio com pasta definida é de propósito); placeholder dos Catálogos casando com a coluna; "a tela NFe Pack Monitor". Regras de número, byte, tempo relativo, singular e nome único escritas em Tokens e Shell.
- Conferido no remote: os 21 `.dc.html` + `support.js` + `github.md` chegaram em `design/` (a pasta saiu de `docs/design/` para a raiz). Alça da Bandeja presente nas 21 telas, incluindo a barra clara do Tokens e Shell.
- Texto da alça no Dashboard igualado ao padrão (accent-200 `#9cc0ec`); ela vinha da rodada 8 com tinta neutra.
- Rodada 22 (coerência): **Duplicidades entrou na barra lateral de todas as telas** (grupo Esteira, depois de Faturamento, com contagem âmbar) e saiu do grupo "Estudos" em Tokens e Shell, onde constava como estudo; **a alça da Bandeja entrou na barra de status** de todas as telas que a têm. Mapa de telas corrigido (linhas contraditórias sobre a Revisão).
- Confirmado que as grades de Triagem, Perfil e Revisão (empilhada) já seguiam a grade da casa; o que resta a 9,5 px são rótulos de campo e títulos de vazio, outro componente.
- Rodada 21 (decisões pendentes): **Revisão alternativa promovida** a `Revisão.dc.html` (a antiga fica como `Revisão (empilhada)`), com tema claro novo; **Bandeja** fechou as três decisões (altura lembrada, atalho em Configurações, "terminou agora" = 2 h / 5 linhas); **Configurações → Atalhos** (seção nova, com conflito e recusa do ambiente); **janela estreita** — regras de 960×540 e 960×1080 em Tokens e Shell, com a Revisão desenhada em um quarto de Full HD.
- Rodada 20 (sistema): **tema claro nas 14 telas que faltavam** (bloco compacto por tela, com a paleta clara de cada padrão novo); **a regra das confirmações decidida e escrita** em Tokens e Shell (troca no lugar = rascunho; faixa âmbar = disco/ERP/serviço); **folhas de referência** de DatePicker, SqlEditor, EmptyState e EmptyScreen.
- Grade do Opmenexo convertida para a grade da casa (ferramentas, cabeçalho com ordenação, linhas de 32 px, tinta 82%, data como texto, paginador).
- Rodada 19: Primeira montagem — casca reduzida, trilha de cinco passos com visto nos cumpridos, passo 1 com o painel do código (relógio de expiração e os seis estados), MedLens com cartões de serviço, Spook como par visível (endereço + credencial), passo 5 dizendo "montada sem o Spook" em verde, e os motivos do Avançar travado.
- **Inventário de telas concluído**: 16 telas do repo + Bandeja de trabalho e Duplicidades (propostas novas).
- Rodada 18: Listas — lista de recortes (com carimbo do espelho e aviso de código sumido na linha) e uma lista aberta com crachá "viva", faixa accent de notícia e faixa âmbar de decisão, mais os estados que separam ausência de perda e os motivos do Salvar cinza. Substantivos de lista no lugar das palavras emprestadas de Relatórios.
- Rodada 17: Relatórios — lista (uma linha por recorte, fixado dito duas vezes, exclusão pergunta na linha) e um relatório aberto na grade da casa, com a data da foto separada de "salvo em", crachá de alterações não salvas, os três vazios, a chave perdida e o Rodar de novo como guia.
- Rodada 16: **Duplicidades** — tela nova, fora do inventário: a porta que o Dashboard promete. Bancada de três painéis (o que o sistema trouxe / o que tentei enviar / o ajuste), reenvio parcial como padrão, portões (desfecho desconhecido, segunda via com confirmação por número, Fantasma ocupado ou sem sessão), caso frio e a lista do que falta no backend.
- Rodada 15: Ignorados — lista de peças postas de lado com origem marcada por ícone, confirmação em linha com as duas frases honestas (e-mail vs pasta vigiada), sinal de vida por comando, os três silêncios e a proposta de agrupar por dia.
- Rodada 14: Busca — campo da barra de título (três estados), tela de resultados com trecho casado em destaque, varredura correndo/revarrendo, recusa por fonte com grupo cortado, e as quatro ausências.
- Aviso do monitor agora abre **Logs · NFe Pack com as linhas novas em destaque** (a porta passou a obedecer à frase do aviso); os Logs ganharam o estado de pouso correspondente.
- Rodada 13: Avisos — sino e Central de avisos (com os cinco estados do sino), a tela de 30 dias com seções por dia e narração "o que o app fez / não fez" no painel de 344 px, cinco ausências, tipos silenciados e propostas (desfazer arquivar, narração no estreito, um sentido de "ontem").
- Rodada 12: Logs — cinco fontes no seletor, diário do app em colunas (carimbo · evento · detalhe) com separador de dia, as quatro fontes cruas, seis ausências com frase própria e as propostas de pasta/leitura recusada/voltar ao topo.
- Rodada 11: Consulta ao Everest — rail de pergunta (editor, portão do Fantasma, mapa, rastro) + painel de resposta na grade da casa, as quatro ausências, sinal de vida, recusas literais, salvar como relatório e a pergunta antes de excluir.
- Rodada 10: Catálogos — nove espelhos com idade na trilha, prontidão do Fantasma no Coletar, coleta em voo, as quatro ausências, recusas literais e o diálogo do SQL.
- **Grade da casa unificada** (docs/05-components.md §1–§2) aplicada em 7 telas: cabeçalho mono 10,5 px, corpo 11,5 px, filete de linha mais fino que a moldura, marcada em accent 22% + barra, sinalizada em âmbar 9% + barra, identidade em accent, coluna numérica em accent-300, data não é número, zebra removida.

- Rodada 9: Configurações — trilha agrupada por tipo com ponto de estado, Sobre, Aparência, Doca, Exportação vazia, consoles MedLens/Fantasma/Spook/NFe Pack com todos os estados, Adapters e o diálogo da colinha.
- Rodada 8: Dashboard — tela cheia, resto da rolagem, faixa de falhas com o nome da fonte, primeira pintura, dia limpo, matriz de Serviços, tema claro e janela estreita.
- Bandeja de trabalho: gaveta no rodapé e alça na barra de status (proposta nova, fora do repo).
- Rodada 7: NFePackMonitor — console (serviço, métricas, fila, ciclos, notas + log), estados e o diálogo da nota (XML e DANFE).
- Revisão alternativa: trilha horizontal de contas + barra do portão no pé da coluna.
- Rodada 6: Perfil (`/profile`) e janela trancada — identidade, avatar/foto, enquadramento, cartão do PIN nos seus trêsidade corpos, teclado da tranca, painel do código e painel sem PIN.
- Rodada 5: Opmenexo — painel e pilhas, subtela FA (nota), folha de resposta SP (cotação + laudo), bloqueios/relatórios e o diálogo de conversa.
- Rodada 4: Faturamento — lista com veredito e estágio separados, painel de detalhe em todos os estados, recuperação, confirmações inline e toasts.
- Rodada 3: Triagem — Na doca, modo recorte, seleção em lote, faixas/banners/vazios e tema claro.
- Rodada 2: Revisão — antessala `/review` e tela da conta `/review/[id]`, com estados, portão, campo casado, variantes do painel do documento e toasts.
- Rodada 1: tokens (claro e escuro) e o shell completo.
- Direção: identidade própria sóbria, IBM Plex Sans/Mono, densidade um pouco mais respirada, escuro e claro.

## Screen map

| Tela / entregável | Arquivos do repo |
| --- | --- |
| Papéis de token (claro + escuro) | docs/03-platform.md, docs/02-invariants.md |
| Shell escuro / claro, rail, estados da lateral | docs/04-shell.md |
| Barra de status, toasts, avisos, menu do usuário | docs/04-shell.md, docs/06-sample-data.md |
| Revisão — trilha de contas + barra de ações (arranjo aprovado) | docs/screens/03-review.md, docs/06-sample-data.md |
| Triagem — doca, recorte, seleção | docs/screens/02-triage.md, docs/06-sample-data.md |
| Faturamento — lista, painel, estados | docs/screens/04-billing.md, docs/06-sample-data.md |
| Opmenexo — painel, pilhas, subtela, conversa | docs/screens/13-opmenexo.md, docs/06-sample-data.md |
| Perfil e janela trancada | docs/screens/16-profile-and-lock.md, docs/06-sample-data.md |
| Revisão (empilhada) — antessala e tela da conta, arranjo anterior | docs/screens/03-review.md |
| NFePackMonitor — console e diálogo da nota | docs/screens/12-nfe-pack-monitor.md, docs/06-sample-data.md |
| Dashboard — funil, só anda com você, duplicidade, gasto, serviços | docs/screens/01-dashboard.md, docs/06-sample-data.md |
| Configurações — nove seções, consoles de serviço, colinha | docs/screens/10-settings.md, docs/06-sample-data.md |
| Catálogos — nove espelhos, coleta, grade, diálogo do SQL | docs/screens/08-catalogs.md, docs/05-components.md |
| Consulta ao Everest — editor, rastro, resposta, rascunho | docs/screens/06-query.md, docs/05-components.md |
| Logs — cinco fontes, diário do app, ausências | docs/screens/11-logs.md |
| Avisos — sino, painel, tela de 30 dias, narração | docs/screens/17-notices.md, docs/04-shell.md |
| Busca — campo da barra de título e tela de resultados | docs/screens/18-search.md |
| Ignorados — peças postas de lado, restaurar e excluir | docs/screens/05-discarded.md |
| Duplicidades (tela nova, proposta) | docs/screens/01-dashboard.md, docs/screens/04-billing.md, docs/screens/03-review.md |
| Relatórios — lista de recortes e um relatório | docs/screens/07-sheets.md, docs/05-components.md |
| Listas — recortes de catálogo, vivos | docs/screens/09-shortlists.md, docs/05-components.md |
| Primeira montagem — a porta, andada uma vez | docs/screens/15-welcome.md, docs/screens/10-settings.md |
| Grade da casa (referência, §6 de Catálogos) | docs/05-components.md |
| Trilha da conta (tela nova, proposta) | docs/screens/03-review.md, docs/screens/04-billing.md, docs/screens/11-logs.md |
| Correções — o que o MedLens erra (tela nova, proposta) | docs/screens/10-settings.md, docs/screens/03-review.md |
| Espelho das notas — o segundo espelho, dos XMLs (tela nova) | docs/screens/24-notes-mirror.md, docs/screens/12-nfe-pack-monitor.md |
| Doca, Exportação e os dois estudos (página de intenção) | docs/screens/10-settings.md, docs/screens/14-future-studies.md |
| Fechamento do período (tela nova, proposta) | docs/screens/01-dashboard.md, docs/screens/04-billing.md |
| Confirmações: a regra + DatePicker, SqlEditor, EmptyState, EmptyScreen (referência em Tokens e Shell) | docs/05-components.md |
| Janela estreita 960×540 / 960×1080 (Tokens e Shell) | docs/04-shell.md |
| Configurações → Atalhos (seção nova) | docs/screens/10-settings.md |
| Bandeja de trabalho (proposta nova) | docs/04-shell.md |

## Sync history

- 2026-09-10T23:05:00Z — rodada 1 (tokens + shell), lida de 00-brief, 02-invariants, 03-platform, 04-shell, 06-sample-data.
