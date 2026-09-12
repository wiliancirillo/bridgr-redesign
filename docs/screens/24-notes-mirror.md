# Espelho das notas — \`/notes-mirror\`

> Where: sidebar › **Fora da esteira** › Espelho das notas, between Catálogos and
> Correções.
>
> **Status: proposed.** Designed in the reborn deck
> (\`Bridgr Reborn - Espelho das notas.dc.html\`), not built.

## The question

**O que as notas já disseram sobre os materiais?**

The NFe Pack already downloads the XMLs ([12-nfe-pack-monitor](12-nfe-pack-monitor.md)).
Each one is a document accepted by SEFAZ and carries, per item: \`cProd\`, \`xProd\`,
\`uCom\`, the ANVISA registry, and — what the ERP mirror **does not have** — \`rastro\`
(lot, fabrication, validity, quantity) plus the price actually charged and the client who
received it.

This is the second mirror: built from the notes, **without asking the Fantasma for
anything**. It does not replace the first one; it has different coverage, and the screen
exists precisely to name where the two disagree.

## The two mirrors

| What is wanted | ERP mirror | Notes mirror | Which decides |
| --- | --- | --- | --- |
| Description and unit | today's registry | what was invoiced | **ERP mirror** |
| The material exists in the registry | all of them | only the invoiced ones | **ERP mirror** |
| Lot, fabrication, validity | — | per note item | **notes** (no competition) |
| Quantity and price that left | price table only | what was charged | **notes** |
| Which client got which lot | — | yes | **notes** |
| How current it is | last collection | last note read | both stamps stay visible |

**Where the notes replace a Everest consult:** lot, validity, quantity sent, price charged,
who received. The ERP mirror never had those fields — a query to find a lot is a query that
never needed to exist.

**Where they do not:** existence. A material registered and never invoiced **is not here**,
and it is exactly the one the gate must find when approving a new bill.

## Anatomy

- **Header**: notes read, materials, lots, and the period covered. The primary button reads
  **"Ler as notas novas"** with a line under it: *não passa pelo Fantasma*.
- **Tabs**: Materiais · Lotes · Notas.
- **The grid** (the house grid): código · descrição na nota mais recente · unidade · ANVISA ·
  notas · lotes · **contra o espelho**. Opening a row expands it inline with the lots
  (lote · qtde · validade · foi para · nota) and, when they disagree, the two texts with
  their dates and the two repairs (recoletar or colinha).
- **"Contra o espelho"** takes three values and never picks a side: \`casa\` (no colour),
  \`difere\` and \`não está no espelho\` (amber).

## Invariants

1. **Reading is reading.** No XML is moved, renamed or deleted; the folder stays the
   monitor's. Same rule as the Doca: this app does not touch someone else's file on its own.
2. **The coverage is always written.** "As notas cobrem 70 dias, de … a …" — it is what stops
   somebody concluding "this material does not exist" from a screen that only knows what was
   sold.
3. **The app never silently prefers one source.** It shows both with dates and names the
   gesture that resolves it.
4. **A cancelled note is struck through, not deleted.** The lot physically left and came back;
   somebody will ask where it is. Deleting the line would make the mirror lie by omission, and
   the file total would stop closing. A \`CC-e\` enters the same way — the new version is added
   without the old one disappearing.
5. **Validity is only marked when it is a decision**: under 90 days **and** already with the
   client is amber. A distant expiry is a fact, and facts get no colour.
6. **Reading does not hold the Fantasma**, and that is stated under the button — it is the most
   important practical difference from Coletar.
7. **Refused files say why, verbatim, and are counted** so the total closes: not an NFe
   (a cancellation event), truncated XML, an inbound note from another CNPJ.

## Sample data

1.482 notes, 387 materials, 2.914 lots, covering 02/07/2026 → 10/09/2026; 23 materials differ
from the mirror. \`10245\` PARAFUSO CORTICAL 3,5 MM X 24 MM TITÂNIO · 213 notes · 36 lots ·
descrição difere (the mirror still has \`PARAFUSO CORTICAL 3,5MM X 24MM TITANIO\`; the notes
changed spelling on 14/08/2026). Lots: \`A7K2219\` qtde 1, val 09/2029, Vale Verde, nota 100246;
\`B19F4402\` qtde 2, val 12/2026 (amber — under 90 days, already with the client), Santa Aurora,
nota 100238; \`A7K1180\` val 11/2026, Instituto Horizonte, nota 100201 — **cancelled** 08/09/2026,
struck through and out of the 36.

## What this needs that does not exist yet

An NFe reader that understands \`det\`, \`rastro\` and \`med\` and stores per note item, plus the
cancellation events already sitting in the monitor's folder. No network, no robot: what is
already on disk is enough.
