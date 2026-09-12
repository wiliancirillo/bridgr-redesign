# Exports do Claude Design

Esta pasta guarda a cópia local revisada das telas baixadas do projeto Bridgr no Claude Design.

## Estrutura

- `*.dc.html`: um canvas exportado por tela ou fluxo. Abra no navegador para consultar o desenho.
- `uploads/`: imagens referenciadas pelos exports.
- `support.js`: suporte gerado junto com o export.
- `github.md`: registro auxiliar do sync e do inventário de telas.

O `.thumbnail` do ZIP não entra no projeto: é metadado do arquivo baixado, não um asset usado pelas telas.

## Fonte de verdade

- `docs/` é a fonte canônica do briefing funcional e o conjunto enviado ao Claude Design.
- `design/` é material visual de referência. Um export não altera, por si só, o comportamento descrito nos documentos.
- Textos, exemplos e mockups continuam sujeitos às regras de `AGENTS.md`, especialmente à proibição de dados reais de pacientes.

## Importação

Ao baixar um novo ZIP:

1. valide a integridade do arquivo;
2. compare cada entrada com o caminho correspondente em `docs/` ou `design/`;
3. copie somente arquivos novos ou cujo conteúdo mudou;
4. não mantenha duas cópias do mesmo conteúdo com nomes diferentes;
5. revise `git diff --check` e `~/Projects/agents/check.sh .` antes de criar o commit.

Os arquivos `.dc.html` e os PNGs de `design/uploads/` são intencionalmente versionados; capturas externas ou dados reais continuam proibidos.
