# Exports do Claude Design

Esta pasta guarda os exports visuais versionados do projeto **Bridgr Redesign**, baixados do Claude Design. Eles acompanham o clone de `alatarithron/bridgr-redesign` para consulta em outras máquinas. Os nomes `Bridgr Reborn - …` foram preservados como no export; renomear o repositório não renomeia os canvases.

## Estrutura

- `*.dc.html`: canvases, estados e alternativas. Abra no navegador mantendo `support.js` e `uploads/` em seus caminhos relativos. Fontes e ícones externos podem exigir internet.
- `uploads/`: imagens referenciadas pelos exports.
- `support.js`: suporte gerado junto com o export.
- `github.md`: registro auxiliar do sync e do inventário de telas.

O `.thumbnail` do ZIP não entra no projeto: é metadado do arquivo baixado, não um asset usado pelas telas.

## Fonte de verdade

- [docs/](../docs/) reúne os briefings funcionais e propostas enviados ao Claude Design. As telas 19–25 estão marcadas como propostas, não como funcionalidades implementadas.
- `design/` é material visual de referência. Um export não altera, por si só, o comportamento descrito nos documentos.
- Textos, exemplos e mockups continuam sujeitos às regras de `AGENTS.md`, especialmente à proibição de dados reais de pacientes.

## Importação

Ao baixar um novo ZIP:

1. valide a integridade do arquivo;
2. compare cada entrada com o caminho correspondente em `docs/` ou `design/`;
3. copie somente arquivos novos ou cujo conteúdo mudou;
4. não mantenha duas cópias do mesmo conteúdo com nomes diferentes;
5. preserve os arquivos locais ausentes do ZIP e investigue divergências nos briefings antes de sobrescrevê-los; uma exportação não valida comportamento do aplicativo;
6. atualize o README e a memória quando houver mudanças de organização ou convenções;
7. revise `git diff --check` e `../agents/check.sh .` antes de um commit autorizado (ajuste o caminho do clone de `agents` se necessário).

Os arquivos `.dc.html` e os PNGs de `design/uploads/` são intencionalmente versionados; capturas externas ou dados reais continuam proibidos.
