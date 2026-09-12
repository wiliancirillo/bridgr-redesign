# Correções — what MedLens keeps getting wrong

> Where: sidebar › **Fora da esteira** › Correções, after Catálogos.
>
> **Status: proposed.** Designed in the reborn deck (`Bridgr Reborn - Correções.dc.html`),
> not built. The Adapters section of [10-settings](10-settings.md) already promises a
> corrections panel that was never built.

## The question

**Onde o MedLens erra sempre — e o que fazer a respeito?**

The operator corrects fields every day and those corrections feed MedLens. Nothing anywhere
shows the result of that work: not which client the model fails on, not that a successful
re-extraction **retires** the previous corrections. This is the only loop in the app a person
feeds without ever seeing the other end.

## The rail — adapters, worst first

One card per adapter: name, **rate**, readings in the period, corrected fields, last
correction. Sorted by rate descending. A separate group at the bottom: **sem leitura no
período**, with no rate at all.

**Rate is corrected fields over fields read**, not bills over bills: an informe with 7 items
has dozens of fields, and "83% of bills had a correction" does not say where it hurts.

## The main panel — fields, then patterns

A house grid, one row per field: **Campo · Corrigido em (n de m leituras) · Taxa · O padrão
mais comum**. Opening a row expands it inline with:

- **the corrections grouped by what was typed**, counted, newest first:
  `AMPARO SAUDE → AMPARO SAÚDE · 19×`, `veio vazio → AMPARO SAÚDE · 6×`;
- **the two repairs that exist**: a line in the colinha (for a field matched against a mirror)
  or an example in the adapter (for a field the model reads wrong).

## Invariants

1. **The pattern is literal; the cause is the operator's.** The screen groups "what came → what
   you typed" and counts. It never writes "the model does not understand accents" — a machine
   concluding about its own error is how this app would lose the operator's trust.
2. **Rate zero is not a celebration.** "22 readings · no corrections" gets a muted tick, never
   green: green in this app means *somebody checked*, and nobody checked that the model was
   right — only that nobody had to intervene.
3. **No rate is not zero.** An adapter with no readings in the period sits in its own group.
4. **Two sources, never in one table.** The list belongs to MedLens. When MedLens is silent the
   screen offers the app's own journal, labelled **outra fonte**, with what it cannot know
   written out: what was retired, and what came from another station.
5. **No patient name anywhere.** A correction is stored as the field, what came, and what was
   typed — which is why it exports to CSV without ceremony.
6. **Gravar na colinha is an amber gesture** and says which of the two outcomes happened:
   "gravada e o arquivo do Fantasma regenerado" or "gravada aqui — o Fantasma ainda não está
   configurado". Today the app promises the first always, and half the time that is false
   (`write_local` returns `Ok(())` without writing when the Fantasma folder is missing).
7. **Retired corrections stay listed**, struck through, with the date. A pattern that comes back
   *after* a re-extraction is the proof that the repair was the colinha, not a fresh read.

## Sample data

Adapters: `Vale Verde · informe` 41% (41 readings, 118 fields, last yesterday) ·
`Santa Aurora · planilha` 12% · `Horizonte · foto de etiquetas` 9% ·
`Sol Nascente · informe` 0 (22 readings) · `Costantini · pedido` no readings.
Fields of Vale Verde: Convênio 34/41 (83%), Médico 27/41 (66%), Quantidade 14/41 (34%),
Lote 9/41 (22%), Data de uso 2/41 (5%, no pattern).

## What this needs that does not exist yet

- MedLens must answer **which corrections exist, grouped, with counts and dates**, and which
  were retired. `retire_corrections` already marks them; nothing reads them back.
- The colinha write must report **whether the robot's file was written**.
