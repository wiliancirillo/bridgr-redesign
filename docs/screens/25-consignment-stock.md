# Estoque em consignação — \`/stock\`

> Where: sidebar › **Fora da esteira** › Estoque, after Espelho das notas.
>
> **Status: proposed.** Designed in the reborn deck
> (\`Bridgr Reborn - Estoque em consignação.dc.html\`), not built. It is the continuation of
> [24-notes-mirror](24-notes-mirror.md): the same XMLs, read as **movement**.

## The question

**Onde está cada lote?**

OPME runs on consignment: material is sent to the hospital, sits there, and is invoiced only
when it is used. The notes already say all of it — remessa em consignação adds to the client,
venda originada de consignação and devolução subtract — so the balance per lot is derivable
from documents alone, with the trail of every note that moved it.

\`saldo = remetido − faturado − devolvido\`

## What it is not

**It is the paper, not the shelf.** Material used in surgery and not yet invoiced still counts
as in the client's hands here. That gap is not a defect: it is precisely the number to take to
the physical count. The screen says so in the toolbar — _o papel, não a prateleira_.

## Anatomy

- **Rail**: clients, with lots-with-balance, items, and the oldest one; amber when something is
  expiring, red when a balance is negative.
- **Grid** (house grid): lote · material · remet. · fatur. · devol. · **alocado** · **em mãos** ·
  **livre** · **há** (default sort, descending) · validade. The three movement columns are quiet
  ink; alocado is accent (an app record), em mãos and livre carry the weight.
- **Expanding a lot** shows the movement trail: date · dot · movement name + CFOP + note (+ the
  bill, on a venda) · the signed quantity · **the running balance**. A venda links to the
  [Trilha da conta](21-bill-trail.md).
- A side card sums it: **em mãos** and **livre** side by side, alocado smaller to their right, then
  the decomposition, with alocado below a rule labelled _reserva, não saída_.
- In the trail, an allocation carries a **hollow dot** and the badge _registro do app_: every other
  movement is a document accepted by SEFAZ, this one is a promise by this app. Drawing them alike
  would give a pedido the authority of a nota.

## The three signals

| Signal                     | When                                          | Means                                                                                           |
| -------------------------- | --------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| no colour                  | balance ≥ 0 with full coverage                | normal                                                                                          |
| **?**, amber               | only vendas were read, no remessa             | the remessa predates the coverage; **what is left is unknown**, because what came in is unknown |
| **−n**, red                | invoiced more than was sent                   | **a document is missing**, not a client debt                                                    |
| **livre 0** with alocado   | everything in hand is committed               | nothing can be promised to a new bill                                                           |
| **alocado > em mãos**, red | the app promised stock the papers do not show | a remessa is missing, or a pedido was built on a lot that already left                          |

The negative case is worded carefully: red because the balance is impossible and somebody must
find the note — an alert that sounds like a charge produces the wrong phone call to the hospital.

## Invariants

1. **A cancelled movement stays in the trail**, struck through with a hollow dot, reading
   _não conta_ instead of a balance. Removing it would make the column add up and the story
   wrong.
2. **A zeroed lot leaves the list and stays findable by search** — it is the answer to "where
   did that lot end up?", the most common question after it closes.
3. **The coverage is written next to every total**: "saldo desde 02/07/2026". A client with
   nothing in hand says _dentro da cobertura_.
4. **"Em mãos há" is the default sort.** In consignment what hurts is not quantity, it is
   time: a lot parked 68 days is money parked and expiry risk.
5. **The + and − in the trail are direction, not state** — the only place in this app where the
   two tones are used arithmetically, and it works because the running balance sits beside them.

## Sample data

Hospital Vale Verde: 34 lots, 41 items, oldest 68 days, saldo desde 02/07/2026.
\`A7K2219\` PARAFUSO CORTICAL · remetido 5 · faturado 1 · saldo 4 · 68 dias · val 09/2029;
movements: 12/07 remessa CFOP 5917 nota 100104 +5 (saldo 5) → 02/09 venda CFOP 5114 nota 100246
−1 (saldo 4), the bill of JOÃO CARLOS PRADO. \`TP15-4410\` saldo ≥ 2 (remessa before coverage).
\`D33K0071\` saldo −1 (no remessa read). \`CO40-1188\` saldo 0, closed 28/08/2026.

## What this needs that does not exist yet

The NFe reader storing \`CFOP\` and \`dest\` per item, plus **a configurable table of which CFOP
adds and which subtracts** — each company uses its own set, and hard-coding 5917/5114/5918
would be this app asserting a rule it did not verify.
