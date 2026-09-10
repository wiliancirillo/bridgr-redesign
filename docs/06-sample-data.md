# Sample data — the fictitious cast

Every mockup uses these values and nothing else. **All of them are fictitious**:
people, hospitals, insurers, codes, orders and documents were made up for this
brief, and any resemblance to a real one is coincidence.

⚠️ Never use a capture of the current app as reference material: every screen
of it shows a real patient's name, procedure and surgery date, and that is
personal health data under Brazilian law (LGPD).

## The operator

| Field                      | Value                           |
| -------------------------- | ------------------------------- |
| Nome                       | Camila Duarte Rocha             |
| Função                     | Faturista OPME                  |
| E-mail                     | camila.rocha@empresa.example    |
| Assinatura dos relatórios  | C. D. ROCHA · FATURAMENTO OPME  |
| Iniciais (avatar)          | CR                              |
| Estação (hostname)         | estacao-faturamento-01          |
| Versão do app              | 0.1.0                           |

## Clients — the hospitals that send informes

Each client is also an **adapter**: the recipe the extraction service uses to
read that hospital's informe.

| Client (as shown)                | Adapter id     |
| -------------------------------- | -------------- |
| Hospital Vale Verde              | `vale_verde`   |
| Hospital Santa Aurora            | `santa_aurora` |
| Instituto Ortopédico Horizonte   | `horizonte`    |
| Hospital Infantil Sol Nascente   | `sol_nascente` |

## ERP registries

| Kind      | Name as read            | ERP code | Note                                   |
| --------- | ----------------------- | -------- | -------------------------------------- |
| Convênio  | VIDA PLENA SAÚDE        | 000412   | matches automatically                  |
| Convênio  | AMPARO SAÚDE            | 000388   | ⚠️ homonym — the "found two" case      |
| Convênio  | AMPARO SAÚDE            | 000391   | ⚠️ homonym — the "found two" case      |
| Convênio  | SUS                     | 000001   |                                        |
| Convênio  | PARTICULAR              | 000002   |                                        |
| Médico    | DR. RAFAEL NOGUEIRA     | 004512   | CRM-PR 00001                           |
| Médico    | DRA. HELENA CASTRO      | 004533   | CRM-PR 00002                           |
| Médico    | DR. OTÁVIO PIRES        | —        | not in the mirror — the "not found" case |

Price table (tabela): `015` — always written with its leading zero, because it
is a text code. Deposits are plain numbers in the ERP: `1 — MATRIZ`,
`4 — CONSIGNADO VALE VERDE`, and the order's consigned-stock deposit `2`.

## Patients and bills

Dates are indicative: each screen document may place a bill on another day of
early September 2026 — always shown `dd/mm/aaaa`.

| Patient               | Atendimento | Data de uso | Client              | Convênio         |
| --------------------- | ----------- | ----------- | ------------------- | ---------------- |
| MARTA LEMOS FERREIRA  | 2026004517  | 02/09/2026  | Hospital Vale Verde | VIDA PLENA SAÚDE |
| JOÃO CARLOS PRADO     | 2026004602  | 02/09/2026  | Hospital Vale Verde | AMPARO SAÚDE (?) |
| LUÍSA ANDRADE REIS    | 2026004633  | 03/09/2026  | Hospital Santa Aurora | SUS            |
| PEDRO HENRIQUE SALLES | 2026004651  | 03/09/2026  | Instituto Ortopédico Horizonte | PARTICULAR |
| BEATRIZ MONTEIRO      | 2026004688  | 04/09/2026  | Hospital Santa Aurora | VIDA PLENA SAÚDE |
| CARLOS EDUARDO LIMA   | 2026004702  | 04/09/2026  | Hospital Infantil Sol Nascente | SUS  |

## Materials

| Descrição                                 | Referência | Lote (as read) | Lote (as the ERP stores it) |
| ----------------------------------------- | ---------- | -------------- | --------------------------- |
| PARAFUSO CORTICAL 3,5 MM X 24 MM TITÂNIO   | PC35-24T   | 451288         | 0000451288                  |
| PLACA BLOQUEADA 3,5 MM 6 FUROS             | PB35-06    | A7K2219        | A7K2219                     |
| CIMENTO ÓSSEO 40 G                         | CO-40      | C24B0917       | C24B0917                    |
| FIO DE KIRSCHNER 1,6 MM                    | FK16       | 902211         | 0000902211                  |

## Deliveries

- E-mail from `faturamento@valeverde.example`, subject "Informe de consumo OPME — semana 36",
  received 02/09/2026 08:41, three pieces: `informe_vale_verde_2026-09-02.pdf`,
  `etiquetas_sala3.jpg`, and the e-mail body.
- Watched folder `~/Documentos/Informes soltos` with a loose `etiquetas_sala5.jpg`.

## ERP and portals

- Everest ERP: conferência `170001`, pedido `420001`.
- OPMENEXO orders `7100231` (awaiting an invoice) and `7100244` (a quotation to answer);
  NF numbers `100245`, `100246`; supplier "DISTRIBUIDORA EXEMPLO LTDA".

## Services and folders

| Service        | Folder (placeholder)        | Address           |
| -------------- | --------------------------- | ----------------- |
| MedLens        | `~/Projetos/medlens`        | `127.0.0.1:8000`  |
| Fantasma       | `~/Projetos/fantasma`       | — (container)     |
| Spook          | `~/Projetos/spook`          | `127.0.0.1:8765`  |
| NFePackMonitor | `~/Projetos/nfepackmonitor` | — (no port)       |
| Fantasma queue | `/srv/fila-fantasma`        |                   |

Secrets are never drawn: a stored secret appears as the masked placeholder
`••••••••••`.
