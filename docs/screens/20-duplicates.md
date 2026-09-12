# Duplicidades — the reconciliation bench

> Where: sidebar › **Esteira** › Duplicidades, after Faturamento. Also the door the
> Dashboard's duplicate comparison already promises.
>
> **Status: proposed.** Designed in the reborn deck (`Bridgr Reborn - Duplicidades.dc.html`),
> not built. The comparison exists ([01-dashboard](01-dashboard.md)); the screen it points
> at does not.

## The question

**A mesma conta entrou duas vezes — qual delas vale, e o que já foi para o Everest?**

The dangerous question is never "which file is better". It is **what already exists in the
ERP**: if one of the two readings produced a pedido, redoing the whole bill creates the
second invoice. That is the failure this screen exists to prevent.

## Two kinds of duplicate, two designs

| | Cold case | Dangerous case |
| --- | --- | --- |
| What happened | neither reading was ever sent | one reading has a pedido in the Everest |
| The decision | which reading follows the belt | what to re-send, and into which pedido |
| Tone | accent — nothing is written to the ERP | amber — the gesture writes to the ERP |
| Reversible | yes, the other piece goes to Ignorados | no |

Treating them alike is what produced the eleven bills invoiced twice.

## The bench — three panels, always in this order

1. **O que o sistema trouxe** — the two MedLens readings side by side, with a middle column
   naming *what differs* (atendimento, item count, file). The operator ticks which one holds.
2. **O que eu tentei enviar** — every run, **lot by lot**, with the Everest's answer verbatim.
   This is the panel that answers "what already exists"; without it the adjustment is a guess.
3. **O ajuste** — the lots as an editable grid (depósito, quantity), with the ones the Everest
   already accepted dimmed and labelled "já no pedido — não reenviar". Two modes:
   **Só os que faltaram** (default) and **A conta inteira**.

## Gates

1. **A run with no confirmed outcome blocks every re-send.** Nobody knows whether the pedido
   was created or how many lots entered. The screen offers the Consulta with the question
   already built (`SELECT … FROM Pedido WHERE cd_paciente = '…' AND dt_uso = '…'`) and a
   manual "registrar o desfecho à mão".
2. **"A conta inteira" with an existing pedido demands the pedido number**, typed, of the one
   the operator cancelled in the Everest. The path exists — sometimes the pedido *was*
   cancelled — but it costs typing: the difference between a decision and a click.
3. **The Fantasma busy or without a session queues the send**, it never denies it. The robot's
   refusal stays the final truth.
4. **"Não são a mesma conta" asks for a short reason**, stored with the pair so it is never
   flagged again — and so whoever opens the case in two weeks knows why.

## Invariants

1. The comparison is crude by design (same patient, same use date) and **wrong sometimes**:
   two surgeries on one day happen. Nothing is blocked or discarded automatically.
2. The default re-send is the safe one. Accepted lots **cannot** be ticked without switching
   to "A conta inteira".
3. The amber band states the whole sentence before the click: how many lots go, into which
   pedido, what is not re-sent, and what happens to the other reading.
4. In the cold case the button is accent, not amber: nothing is written to the ERP — the
   belt is just being tidied, and the set-aside piece is restorable from Ignorados.

## Sample data

MARTA LEMOS FERREIRA, uso 01/09/2026, two readings: `lote_setembro_01.pdf` (atend.
2026004498, 7 items) and `reenvio_marta.pdf` (atend. 2026004512, 9 items). Run 1: pedido
`0000451288` created with 5 of 7 lots; `A7K2219` → "lote nao encontrado no deposito 2";
`C24B0917` → "quantidade indisponivel: saldo 0".

## What this needs that does not exist yet

1. **The Everest's answer per lot, stored.** Billing records the bill's outcome; panel 2
   needs the line for each lot — that is what says what *not* to re-send.
2. **A partial send in the Fantasma.** Today the queue takes a whole bill. "Incluir no pedido
   existente" is a new robot gesture; without it the only honest path is cancel-and-resend.
3. **The resolved pair, with a reason.** Two ids and a short text, no patient name.

Without (1) and (2) the screen still earns its place: it resolves the cold case completely,
and in the dangerous one it leads to the Consulta with the question built instead of leaving
the operator to guess.
