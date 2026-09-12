# Trilha da conta — `/bill/[id]`

> Where: no sidebar item. Entered from a patient name in any list, and from
> [Busca](18-search.md). One bill per page, like the account screen in Revisão.
>
> **Status: proposed.** Designed in the reborn deck (`Bridgr Reborn - Trilha da conta.dc.html`),
> not built.

## The question

**O que aconteceu com esta conta?**

Today the answer needs five screens and a log read: the file in Triagem, the review in
Revisão, the runs in Faturamento, the invoice in NFe Pack, the events in Logs. Fragments of
this page have already been drawn three times — panel 2 of [Duplicidades](20-duplicates.md),
the outcome recovery in [Faturamento](04-billing.md), and the notice landing in
[Logs](11-logs.md) — which is the clearest sign the page exists and is scattered.

## The trail

Newest first, grouped by calendar day (the same ordering as Logs and Avisos; the frequent
question is "e agora?"). A **Do começo** control reverses it and keeps the day separators.

Each line is four columns:

| Column | Width | Holds                                                   |
| ------ | ----- | ------------------------------------------------------- |
| time   | 56 px | `HH:MM`, mono                                           |
| actor  | 83 px | initials + name for a person, icon + name for a service |
| spine  | 23 px | the dot, with a halo in the page colour                 |
| body   | rest  | what happened, plus an embedded block when there is one |

**The actor comes before the act.** It is the first question in front of a strange bill: did
the robot do this alone, or did somebody tell it to?

Dot tones, the same four as everywhere: neutral _something happened_, accent _something
arrived_, amber _a person decided_, green _a person checked_.

## Embedded blocks

- **A run** — the house grid, lot by lot, with the Everest's words untranslated.
- **Corrections** — field, before (struck through), after, and the adapter they fed.
- **A retired correction** — an amber block listing what the re-extraction discarded.

## The hole

When a run loses its outcome, the spine **stops**: three loose dots follow, no line after it
is invented, and the body says "não se sabe se o pedido foi criado nem quantos lotes
entraram", with a door to ask the Everest. Drawing a hole as a hole is the opposite of
closing the story with "provavelmente deu certo".

## Invariants

1. **This screen only reads.** No gesture that writes lives here — only doors to the screen
   that owns the decision at that moment. A trail that also acted would be a second source
   of truth about the same bill.
2. **It never says "só isto aconteceu".** What a person did inside the Everest, through its
   own screen, does not pass through here; the right rail says so in words.
3. **A short trail is not an empty state.** A bill that arrived today has two lines and no
   apology; the last line says where it stopped moving on its own.
4. **Refusals come first, verbatim** — `selado: o arquivo do registro está ilegível — o selo
não confere`; `selado: esta conta não existe mais`; `a peça desta conta saiu da doca — a
trilha para na extração`. The last is the most common: the piece was deleted from
   Ignorados and took the beginning of the story with it.

## Sample data

JOÃO CARLOS PRADO · atend. 2026004602 · Hospital Vale Verde · AMPARO SAÚDE · uso 02/09/2026 ·
7 items · faturada, nota 100246. Trail: file arrived 09/09 09:12 (MailBridgr,
faturamento@valeverde.example) → MedLens read 7 items 14:28 → gate held 14:31 (1 item with no
ERP code) → 3 corrections 14:43 → approved 14:47 → delivered 14:51 → run 1 partial 15:04 (5 of 7) → 2 lots delivered 10/09 11:40 → run 2 13:58 → note downloaded 14:02 → invoiced 14:05.

## What this needs that does not exist yet

**The bill id on every journal line.** The events already exist in Logs; today they do not know
which bill they belong to. Everything else is already stored.
