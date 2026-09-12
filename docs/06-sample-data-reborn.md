# Dados canônicos do reborn — uma conta, um número, um significado

> Complementa `docs/06-sample-data.md`. Existe porque as telas do reborn foram desenhadas em 32
> rodadas e a mesma conta aparece em oito delas: sem uma folha só, cada tela inventa a sua versão
> da Marta. **Regra raiz: nenhum número aparece como duas coisas diferentes.**

## As faixas de número

| Coisa              | Faixa                            | Exemplos                          |
| ------------------ | -------------------------------- | --------------------------------- |
| Atendimento        | `20260046xx` – `20260047xx`      | 2026004602 · 2026004517           |
| Pedido no Everest  | `00004514xx`                     | 0000451288 · 0000451402           |
| Nota fiscal        | `1002xx`                         | 100246 · 100238 · 100201 · 100104 |
| Lote               | alfanumérico **ou** `00009xxxxx` | A7K2219 · C24B0917 · 0000902211   |
| Código de material | 5 dígitos                        | 10245 · 10311 · 10877             |

Um número usado numa faixa **nunca** reaparece em outra. O caso que quebrou isso foi
`0000451288`: pedido do João em três telas e coluna LOTE em duas outras.

## As seis contas

| Paciente              | Atend.     | Cliente               | Convênio         | Uso        | Itens | Onde está                                  |
| --------------------- | ---------- | --------------------- | ---------------- | ---------- | ----- | ------------------------------------------ |
| JOÃO CARLOS PRADO     | 2026004602 | Hospital Vale Verde   | AMPARO SAÚDE     | 02/09/2026 | 7     | faturada · pedido 0000451288 · nota 100246 |
| MARTA LEMOS FERREIRA  | 2026004517 | Hospital Santa Aurora | VIDA PLENA SAÚDE | 01/09/2026 | 9     | duplicidade aberta (dois arquivos)         |
| LUÍSA ANDRADE REIS    | 2026004633 | Instituto Horizonte   | SUS              | 03/09/2026 | 6     | duplicidade fria                           |
| ANA BEATRIZ MOURA     | 2026004688 | Hospital Sol Nascente | AMPARO SAÚDE     | 04/09/2026 | 4     | na fila do Fantasma · pedido 0000451402    |
| CARLOS EDUARDO LIMA   | 2026004733 | Hospital Santa Aurora | AMPARO SAÚDE     | 05/09/2026 | 5     | falhou no Everest                          |
| PEDRO HENRIQUE SALLES | 2026004651 | Hospital Vale Verde   | PARTICULAR       | 03/09/2026 | 3     | na Revisão                                 |

**Cliente é o hospital; convênio é quem paga.** Os dois nunca ocupam o mesmo lugar na tela: onde
cabe um só, cabe o cliente.

## A duplicidade da Marta

Duas leituras do mesmo uso, e é nisso que elas discordam:

|                  | `lote_setembro_01.pdf` | `reenvio_marta.pdf`     |
| ---------------- | ---------------------- | ----------------------- |
| Atendimento lido | **2026004517** (certo) | **2026004512** (errado) |
| Itens            | 7                      | 9                       |
| Lida em          | 01/09/2026 14:22       | 04/09/2026 08:51        |

## As notas e os lotes

| Nota   | Cliente                 | Operação               | Emissão    | Observação                     |
| ------ | ----------------------- | ---------------------- | ---------- | ------------------------------ |
| 100104 | Hospital Vale Verde     | remessa em consignação | 12/07/2026 | +5 do lote A7K2219             |
| 100201 | Instituto Horizonte     | venda                  | 31/08/2026 | **cancelada** em 08/09/2026    |
| 100238 | Hospital Santa Aurora   | venda                  | 02/09/2026 | lote B19F4402                  |
| 100246 | **Hospital Vale Verde** | venda                  | 10/09/2026 | a conta do João · lote A7K2219 |

Lotes em uso nas telas: `A7K2219` `A7K1180` `B19F4402` `C24B0917` `PL35-8871` `HF10-2204`
`CO40-1192` `D33K0071` `TP15-4410` `0000902211`.

## Cobertura das notas lidas

02/07/2026 → 10/09/2026 (70 dias), nas duas telas que a citam — Espelho das notas e Estoque.
