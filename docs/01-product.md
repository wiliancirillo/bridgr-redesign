# The product — who uses Bridgr Pro, and for what

## The operator

One person — the _faturista_ — at one machine, all day. A billing
professional at a distributor of **OPME** (_órteses, próteses e materiais
especiais_: screws, plates, implants and other materials used in surgery) in
Brazil. The screens are written in that person's vocabulary, in Brazilian
Portuguese.

The work is expert, repetitive and expensive when wrong: a wrong code in the
ERP is a wrong invoice, found far away, weeks later, by somebody who cannot
connect it back to the screen that produced it. The app exists to make every
decision visible to the person who has to take it — and to never pretend a
check happened when it did not.

⚠️ Nearly every screen shows **patient data** (name, procedure, surgery date,
medical record). That is personal health data under Brazilian law (LGPD).

## The work, end to end

A hospital uses materials in a surgery. It sends the distributor a consumption
report — the **informe** — by e-mail or through its purchasing portal. The
distributor has to invoice each surgery: one **conta** (bill) per patient, with
the patient, the date of use, the hospital's admission number (_atendimento_),
the insurer (_convênio_), the doctor, and every material used with its **lot**
and quantity. The invoice is made in the distributor's ERP, the **Everest**.

```
  e-mail / loose file          AI reading              human conferência         robot typing into the ERP
 ┌──────────────────┐     ┌──────────────────┐     ┌────────────────────┐     ┌──────────────────────────┐
 │ Triagem          │ ──▶ │ (MedLens)        │ ──▶ │ Revisão            │ ──▶ │ Faturamento              │
 │ whose attachment │     │ extracts bills   │     │ is every bill      │     │ where is each bill, and  │
 │ is this?         │     │ from the informe │     │ right and complete?│     │ what about the stuck one?│
 └──────────────────┘     └──────────────────┘     └────────────────────┘     └──────────────────────────┘
        │  ignore                                           │ "Aprovar" writes the bill
        ▼                                                   ▼ into the robot's queue
    Ignorados                                          (Fantasma → Everest ERP)
```

That is **the conveyor** (_esteira_), and the sidebar lists its three stations
in the order the work moves.

## The ecosystem around the app

Bridgr Pro is the **human station** of a set of local programs. It is also
their **console**: it raises them, configures them, shows their state and their
logs — a new machine is set up entirely from the app's own screens.

| Program            | Role                                                                                                      | Where the operator meets it                  |
| ------------------ | --------------------------------------------------------------------------------------------------------- | -------------------------------------------- |
| **MailBridgr**     | Thunderbird extension; delivers e-mail attachments into the app's inbox folder; generates one-time codes  | Triagem; e-mail confirmation; PIN reset      |
| **MedLens**        | Local AI extraction service; reads each informe with an adapter per hospital; spends money on an AI provider | Triagem (extract), Revisão (corrections), spend panels |
| **Fantasma**       | The robot: drives the Everest ERP through a remote-desktop session, one task at a time; also answers read-only SQL | Faturamento, Consulta, Catálogos             |
| **Spook**          | Automation of hospital purchasing portals — today the **OPMENEXO** portal                                 | OPMENEXO screen                              |
| **NFePackMonitor** | Downloads the distributor's issued electronic invoices (NF-e) from a portal, cycle after cycle            | NFe Pack monitor screen; attach-by-number    |
| **Everest**        | The ERP. Nothing in this app writes to it directly — only the robot does, one bill per confirmed click     | Everywhere a code appears                    |

## The screens

| Route                | Name on screen      | Sidebar group        | The question it answers                                                        | Document                                                       | Status        |
| -------------------- | ------------------- | -------------------- | ------------------------------------------------------------------------------ | -------------------------------------------------------------- | ------------- |
| `/dashboard`         | Dashboard           | (first, no heading)  | How is the day going, and what only moves with me?                             | [01](screens/01-dashboard.md)                                  | built         |
| `/triage`            | Triagem             | Esteira              | Whose attachment is this, and does it become an extraction or go out?          | [02](screens/02-triage.md)                                     | built         |
| `/review`, `/review/[id]` | Revisão        | Esteira              | Which file to open — and did the AI read every bill right?                     | [03](screens/03-review.md)                                     | built         |
| `/billing`           | Faturamento         | Esteira              | Where is each approved bill, and what do I do about the one that stopped?      | [04](screens/04-billing.md)                                    | built         |
| `/discarded`         | Ignorados           | Fora da esteira      | What did I take out of the dock — restore it or delete it for good?            | [05](screens/05-discarded.md)                                  | built         |
| `/query`             | Consulta            | Fora da esteira      | What does the ERP say, for the question the conveyor cannot answer?            | [06](screens/06-query.md)                                      | built         |
| `/sheets`, `/sheets/[id]` | Relatórios (under Consulta) | Fora da esteira | Which query results did I keep, and what did they say?                   | [07](screens/07-sheets.md)                                     | built         |
| `/catalogs`          | Catálogos           | Fora da esteira      | What do the ERP's registries hold, as of when?                                 | [08](screens/08-catalogs.md)                                   | built         |
| `/shortlists`, `/shortlists/[id]` | Listas (under Catálogos) | Fora da esteira | Which items of a registry did I set apart?                               | [09](screens/09-shortlists.md)                                 | built         |
| `/settings`          | Configurações       | Sistema              | How is this station set up — services, folders, credentials, appearance?       | [10](screens/10-settings.md)                                   | built         |
| `/logs`              | Logs                | Sistema              | What did the services say, and what did the app do?                            | [11](screens/11-logs.md)                                       | built         |
| `/nfe-pack-monitor`  | NFePackMonitor      | Ideias futuras       | Are the invoices coming down from the portal on their own?                     | [12](screens/12-nfe-pack-monitor.md)                           | built         |
| `/opmenexo`          | Opmenexo            | Ideias futuras       | Is there work waiting for me on the portal, and of what kind?                  | [13](screens/13-opmenexo.md)                                   | built         |
| `/inpart`, `/duplicates`, `/costantini` | Inpart Saúde, Duplicidades, Costantini | Ideias futuras | — (studies: empty rooms that say so)                          | [14](screens/14-future-studies.md)                             | empty study   |
| `/welcome`           | Primeira montagem   | — (a door, walked once) | Is this station assembled — who am I, and are the services connected?       | [15](screens/15-welcome.md)                                    | built         |
| `/profile`           | Perfil              | — (user menu)        | Who is at this bench, and how does the app show them?                          | [16](screens/16-profile-and-lock.md)                           | built         |
| `/notices`           | Avisos              | — (the bell)         | What is waiting for my decision, and what happened without me?                 | [17](screens/17-notices.md)                                    | built         |
| `/search`            | Busca               | — (titlebar field)   | Where does this term appear across everything this station holds?              | [18](screens/18-search.md)                                     | built         |

The frame around every screen — titlebar, sidebar, status bar, toasts — is in
[04-shell.md](04-shell.md).

## Glossary

The words that repeat. The operator reads the Portuguese; the code uses the
English in parentheses.

| Term (pt-BR)              | Meaning                                                                                                           |
| ------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| **informe** (report)      | The consumption report a hospital sends. One informe carries **several bills**.                                   |
| **peça** (piece)          | One attachment of an e-mail, the e-mail body, or a loose file from a watched folder — the unit of triage.         |
| **remessa** (shipment)    | One e-mail's delivery: the set of pieces that arrived together.                                                   |
| **doca** (dock)           | Where pieces wait to be triaged.                                                                                  |
| **adaptador / cliente**   | The hospital a piece belongs to — and the reading recipe the AI service uses for that hospital's informe.         |
| **extração**              | The AI service reading a piece and producing bills.                                                               |
| **conta** (bill)          | One patient's surgery to invoice: patient, date of use, atendimento, convênio, doctor, items. Never "account".     |
| **atendimento** (admission) | The hospital's admission number for that surgery.                                                               |
| **convênio** (insurer)    | The health plan that pays.                                                                                        |
| **item / material**       | A line of the bill: description, referência, lote, quantidade.                                                    |
| **lote** (lot)            | The manufacturer's batch. The ERP finds the item **from the lot** — it is the half the other side cannot guess.   |
| **referência**            | The material's reference code — audit trail for humans; the ERP does not invoice from it.                         |
| **colinha** (cheatsheet)  | Per-client settings the robot needs to invoice that hospital's bills (deposit, price table and others).           |
| **tabela**                | ERP price table, a code like `015` — the leading zero is part of it.                                              |
| **depósito** (deposit)    | ERP stock location.                                                                                               |
| **cadastro / espelho / catálogo** | An ERP registry (doctors, insurers, materials…) / its local copy / the screen that shows it.               |
| **fila** (queue)          | The robot's queue folder. A file in it is a bill waiting for the robot.                                           |
| **conferência** (ERP)     | The ERP's step that checks the items; once _encerrada_ (closed) it cannot be undone.                              |
| **pedido**                | The sales order the robot creates in the ERP. The robot's cycle ends there.                                       |
| **veredicto** (verdict)   | What the ERP says about a bill: _sem registro_, _faturada_, _provável_, or nothing asked yet (_não checado_).      |
| **NF / NF-e**             | Brazilian electronic invoice.                                                                                     |
| **relatório** (sheet)     | A saved result of a query — a photograph.                                                                          |
| **lista** (shortlist)     | A saved set of items of a catalog — live, re-read against the mirror each time.                                   |
| **aviso** (notice)        | A subject pending a human decision, behind the bell. Not a toast, which is feedback of an act.                    |
| **ordem** (OPMENEXO)      | A purchase order on the hospital's portal, waiting for an invoice or for a quotation answer.                      |
