# Fechamento do período — what is still open

> Where: sidebar, next to Dashboard, above the **Esteira** group.
>
> **Status: proposed.** Designed in the reborn deck
> (`Bridgr Reborn - Fechamento do período.dc.html`), not built.

## The question

**O que de setembro ainda está em aberto, por cliente?**

The Dashboard answers "hoje" and [Faturamento](04-billing.md) answers "esta conta". Nobody
answers the month-end question, so today it is an ad-hoc Consulta the operator rebuilds every
time.

## What the screen refuses to say

**It does not state the month's revenue.** The Everest owns the value of a bill after it leaves
here — price table, portal negotiation, glosa — so a total in reais would be read as
faturamento and would be false. The screen counts **contas**, and says so in the footer, with a
door to the Consulta for the real number.

It also says, in the same band, that a bill invoiced **directly in the Everest** never passed
through here and is not counted.

## The period

Chosen by month, with prev/next and a custom range. **The period is by use date**
(`dt_uso`), not by the date the bill was invoiced: it is how the hospital charges and how the
reconciliation closes. A bill used in August and invoiced in October counts in August — and
that is exactly the one that disappears from reports.

The header always carries two stamps: whether the month is still running ("12 de 30 dias") and
**the age of the oldest verdict in the numbers** ("o desfecho mais velho aqui foi checado há
3 dias").

## The funnel of the period

Five numbers that must close, with what fell out written under each:

| Step | Sample | Under it |
| --- | --- | --- |
| Chegaram | 74 | peças na doca |
| Viraram conta | 68 | 3 não foram lidas · 3 postas de lado |
| Aprovadas | 51 | 9 não conferidas · 5 no portão · 3 não enviar |
| Faturadas | 41 | conferidas no Everest |
| Em aberto | 24 | + 3 fechadas por decisão |

`41 + 24 + 3 = 68`. **The arithmetic must close**: on a closing screen the first thing the
operator does is add the columns.

## The open ones, named

A house grid, one row per bill, **"parada há" as the first column and the default sort**: at a
closing the question is not which bill is biggest, it is which is rotting longest. Columns:
parada há · atendimento · paciente · cliente · onde parou · por quê. Every row is a door.

Tabs: **Em aberto · Por cliente · Por convênio · Faturadas**. The per-client table has a
footer that sums its columns against the funnel; the per-row remainder (contas − faturadas −
em aberto) is the "não enviar", so **the row does not close on purpose** and the note says it.

## Invariants

1. **Em aberto and fechada por decisão are different things.** The "não enviar" bills do not
   count as work waiting, and they do not disappear either — at month end somebody asks
   about them.
2. **Rechecar os desfechos holds the Fantasma**, and the screen says so before the click.
3. **An empty month and a clean month say opposite things** and get different sentences. The
   clean month's tick is muted: nobody checked that everything is right; what happened is that
   no bill was left hanging.
4. **"Mês fechado" means the period ended, not that the bills ended.** Bills still open stay
   listed.

## Thresholds

Written once, in the shell's token sheet: a stalled bill turns **amber at 7 days** and **red at
15**; a verdict is **stale at 3 days** during a closing. The 3 is the least obvious — rechecking
holds the Fantasma, so it must be short enough for the closing to mean something and long
enough not to recheck all day.

## Sample data

September 2026 on the 12th: 74 pieces → 68 bills → 51 approved → 41 invoiced, 24 open + 3
closed by decision. Open by kind: 9 Revisão · 5 portão · 4 falhou · 2 provável · 2 fila ·
1 duplicidade · 1 sem desfecho. Per client: Vale Verde 24/14/9 · Santa Aurora 19/12/6 ·
Horizonte 13/6/6 · Sol Nascente 12/9/3. August closed: 1.184 bills, 1.176 invoiced, 8 open.
June clean: 1.052 bills, none open. July empty.

## What this needs that does not exist yet

- Aggregates by use-date period, per client and per insurer, over the bills' own records.
- The oldest `checked_at` inside each aggregate, so the header stamp can exist.
