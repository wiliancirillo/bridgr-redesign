# Bridgr Reborn

Brief funcional para dar uma **cara nova ao Bridgr Pro** no Claude Design.

Cada tela do app está descrita aqui a partir do **código** (não do desenho
atual): o que ela responde, o que mostra, todos os estados, todos os gestos, as
regras de negócio — e o que uma nova aparência pode mudar à vontade e o que ela
não pode derrubar.

> ⚠️ **Nenhum dado real de paciente entra aqui**, nem captura do app atual:
> toda tela dele mostra nome, procedimento e data de cirurgia (LGPD). Os
> exemplos usam o elenco fictício de `docs/06-sample-data.md`.

## Estrutura

```
bridgr_reborn/
├── docs/                         ← fonte canônica e conjunto que sobe para o Claude Design
│   ├── 00-brief.md               o pedido: entregáveis, regras dos mockups, o que é fixo e o que é livre
│   ├── 01-product.md             quem usa, o fluxo, o ecossistema, o mapa das telas, glossário
│   ├── 02-invariants.md          as regras que valem em toda tela e passam na frente da estética
│   ├── 03-platform.md            restrições técnicas: WebKitGTK, janela, controles nativos, tokens, densidade
│   ├── 04-shell.md               barra de título, lateral, barra de status, toasts
│   ├── 05-components.md          componentes compartilhados (grade, filtro, listas salvas, confirmação…)
│   ├── 06-sample-data.md         o elenco fictício base
│   ├── 06-sample-data-reborn.md  dados canônicos adicionais das telas novas
│   ├── inventario-de-instancias.md inventário dos componentes compartilhados nos exports
│   └── screens/                  uma tela por arquivo, na ordem da barra lateral
├── design/                       ← exports baixados e revisados do Claude Design
│   ├── *.dc.html                 uma tela/canvas por arquivo
│   ├── uploads/                  assets referenciados pelos exports
│   ├── support.js                suporte gerado pelo export
│   └── github.md                 registro auxiliar do sync do design
├── insights/
│   └── project-insights.md       ← achados de engenharia para o desenvolvedor (NÃO sobe)
├── AGENTS.md                     regras para agentes neste repositório
└── .agents/                      memória do projeto e decisões
```

## Como usar com o Claude Design

1. **Feche a direção** que o brief deixa em aberto (`docs/00-brief.md`, última
   seção): manter o parentesco com o KDE/Breeze ou uma identidade própria;
   escuro como padrão; fonte (precisa ir embutida no app); ícones; densidade.
2. **Crie um projeto novo** no Claude Design e suba os documentos-base
   (`docs/00` a `docs/06`, ~125 KB). O conjunto inteiro passa de 800 KB
   (~200 mil tokens): em vez de subir tudo de uma vez, acrescente em cada
   rodada o arquivo da tela em foco (`docs/screens/NN-*.md`, de 10 a 90 KB
   cada). Não suba `insights/`, nem capturas do app.
3. **Prompt sugerido** para a primeira rodada:

   > Anexei o brief funcional do Bridgr Pro. Leia primeiro `00-brief.md`,
   > `01-product.md`, `02-invariants.md` e `03-platform.md`. Quero uma
   > identidade visual nova para o app: _[a direção escolhida no passo 1]_.
   > Comece pelos tokens (claro e escuro) e pelo shell (`04-shell.md`); depois
   > desenhe a Revisão (`screens/03-review.md`) com todos os estados que o
   > documento lista. Use apenas os dados fictícios de `06-sample-data.md` e os
   > textos em português exatamente como estão nos documentos. O que estiver em
   > "Must survive the redesign" e em `02-invariants.md` é regra; "Design
   > opportunities" são sugestões que você pode adotar ou não.

4. **Itere tela a tela**, apontando o documento da tela em cada pedido
   (`screens/NN-nome.md`) — ele traz os estados, os gestos e as linhas de
   exemplo prontas. As telas 19 a 25 usam também `06-sample-data-reborn.md`,
   que mantém números e contas consistentes entre os novos fluxos.
5. Os exports visuais baixados ficam em `design/` para consulta e revisão. Eles
   não substituem os briefings de `docs/`, que continuam sendo a fonte canônica
   e o conjunto enviado ao Claude Design. `inventario-de-instancias.md` registra
   a cobertura dos componentes compartilhados nos exports.
6. Quando a direção estiver aprovada, o caminho de volta para o código é pelos
   **papéis de token** de `docs/03-platform.md`: cada papel novo precisa
   mapear para um papel que o código usa (ou vir com a proposta de mapeamento).

## Manutenção

Os documentos descrevem o Bridgr Pro no commit registrado em
`.agents/PROJECT_MEMORY.md`. Quando uma tela mudar no app, o documento dela
muda junto e o registro do commit avança. Verificação de forma:

```sh
~/Projects/agents/check.sh .
```
