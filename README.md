# Bridgr Redesign

Briefings funcionais, propostas e arquivos de design para o redesenho do **Bridgr Pro** no Claude Design. Este repositório reúne o que o app faz, as regras que a nova interface deve preservar e os canvases exportados para consulta em outras máquinas. Não contém a implementação do aplicativo.

- **Comportamento existente:** descrito a partir do código, no commit registrado em [PROJECT_MEMORY.md](.agents/PROJECT_MEMORY.md).
- **Propostas:** identificadas como tal nos documentos; estar desenhado não significa estar implementado. As telas 19–25 são propostas de novos fluxos.
- **Design visual:** exports versionados em [design/](design/), com alternativas e estados de referência. Eles não substituem as regras funcionais nem comprovam aprovação de todas as propostas.

> **Nenhum dado real de paciente entra aqui**, nem captura do app atual. Os exemplos usam o elenco fictício de [06-sample-data.md](docs/06-sample-data.md); [06-sample-data-reborn.md](docs/06-sample-data-reborn.md) reúne os dados de referência usados nos novos desenhos.

## Estrutura

```text
bridgr-redesign/
├── docs/                         briefings e propostas enviados ao Claude Design
│   ├── 00-brief.md               pedido, entregáveis e limites do redesenho
│   ├── 01-product.md             produto, fluxo e ecossistema
│   ├── 02-invariants.md          regras que passam na frente da estética
│   ├── 03-platform.md            plataforma, tokens e restrições técnicas
│   ├── 04-shell.md               estrutura da janela e navegação
│   ├── 05-components.md          componentes compartilhados
│   ├── 06-sample-data.md         elenco fictício base
│   ├── 06-sample-data-reborn.md  dados de referência dos novos desenhos
│   ├── inventario-de-instancias.md  cobertura de componentes e lacunas conhecidas
│   └── screens/                  documentos por tela e propostas de novos fluxos
├── design/                       exports visuais versionados
│   ├── README.md                 consulta e procedimento de importação
│   ├── *.dc.html                 canvases, estados e alternativas
│   ├── uploads/                  imagens que acompanham os exports
│   ├── support.js                suporte de renderização do export
│   └── github.md                 registro de sincronização e mapa dos desenhos
├── insights/                     achados de engenharia; não enviar ao Claude Design
├── AGENTS.md                     regras para agentes
└── .agents/                      memória e decisões do projeto
```

## Consultar os designs

Clone o repositório completo para ter os canvases e seus arquivos de suporte:

```sh
git clone git@github.com:alatarithron/bridgr-redesign.git
cd bridgr-redesign
```

Abra um arquivo `.dc.html` de `design/` no navegador. Mantenha `support.js` e `uploads/` ao lado dos HTMLs, na estrutura do repositório. O GitHub permite consultar o código do arquivo, mas não funciona como visualizador desses canvases.

Comece por [Tokens e Shell](design/Bridgr%20Reborn%20-%20Tokens%20e%20Shell.dc.html), que reúne a linguagem visual e os componentes de referência. O conjunto inclui:

- **Esteira:** Dashboard, Triagem, Revisão, Faturamento e Duplicidades.
- **Consulta e cadastros:** Consulta, Catálogos, Relatórios, Listas e Ignorados.
- **Serviços e portais:** NFePackMonitor e Opmenexo.
- **Sistema:** Configurações, Perfil e janela trancada, Primeira montagem, Busca, Avisos e Logs.
- **Novos fluxos:** Bandeja de trabalho, Trilha da conta, Correções, Fechamento do período, Espelho das notas e Estoque em consignação.
- **Referências e alternativas:** Tokens e Shell, Doca, Exportação e estudos e Revisão (empilhada).

Os nomes `Bridgr Reborn - …` são os nomes originais dos exports do Claude Design; foram preservados apesar da renomeação do repositório. A variante `Revisão (empilhada)` é distinta de `Revisão`, não uma cópia para excluir por nome parecido.

Há referências externas a fontes do Google Fonts e ícones do Phosphor: os arquivos locais não garantem uma visualização idêntica sem acesso à internet. Consulte [design/README.md](design/README.md) para os cuidados de importação e [design/github.md](design/github.md) para o mapa dos desenhos.

## Trabalhar com os briefings

1. Leia [00-brief.md](docs/00-brief.md), [01-product.md](docs/01-product.md) e [02-invariants.md](docs/02-invariants.md). Eles separam liberdade visual de regras de negócio.
2. Use [03-platform.md](docs/03-platform.md), [04-shell.md](docs/04-shell.md) e [05-components.md](docs/05-components.md) como referências compartilhadas.
3. Envie ao Claude Design o documento da tela em foco de [docs/screens/](docs/screens/), junto dos dados fictícios pertinentes. Não envie `insights/` nem capturas reais do app.
4. Nas telas 19–25, trate o conteúdo como **proposta**, não como descrição de código existente. O [inventário de instâncias](docs/inventario-de-instancias.md) registra a cobertura dos componentes e suas lacunas; não é um selo de validação completa.
5. Compare o desenho com as seções “Must survive” e com as invariantes. Quando uma direção for aprovada, o retorno à implementação passa pelo mapeamento dos papéis de token e das regras funcionais, no repositório do aplicativo.

## Atualizar os exports

Siga o procedimento em [design/README.md](design/README.md): valide o ZIP, compare os conteúdos por hash e importe apenas novidades ou alterações. Documentos idênticos permanecem uma única vez em `docs/`; canvases e suporte ficam em `design/`. Não recrie `docs/design/docs/` nem armazene o ZIP ou `.thumbnail` no projeto.

Os `.dc.html` e os assets permitidos em `design/uploads/` são versionados para acesso em outras máquinas. Preserve nomes e caminhos relativos exigidos pelos exports. Atualize este README e a memória quando a organização ou as convenções mudarem.

## Validação e manutenção

Quando o comportamento do app mudar, revise o documento correspondente a partir do código e atualize o commit de referência na memória. Uma nova exportação visual, por si só, não muda esse commit.

```sh
git diff --check
../agents/check.sh .
```

O segundo comando pressupõe o clone de `agents` ao lado deste repositório; ajuste o caminho se necessário. A checagem valida a política dos arquivos de instruções e memória, não a aparência dos canvases nem sua correspondência com o aplicativo. Não há suíte de testes ou build de aplicativo neste repositório.
