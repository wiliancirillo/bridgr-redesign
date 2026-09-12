# Bandeja de trabalho — the status-bar tray

> Where: a handle in the status bar, right of the service blocks; it opens a drawer
> above the toast stack. No sidebar item and no route — the tray is part of the shell.
>
> **Status: proposed.** Designed in the reborn deck (`Bridgr Reborn - Bandeja de trabalho.dc.html`), not built. Nothing in `docs/04-shell.md` describes it yet.

## The question

**O que está correndo agora, e o que está esperando por mim?**

Today that answer is spread across the screens that own each gesture: a collection
lives in Catálogos, a run lives in Faturamento, a portal sweep lives in OPMENEXO, and
the status bar only says the Fantasma is "ocupado". An operator who starts a
collection and walks to another screen has no way back to it except by remembering
where it was.

## The handle

Sits in the status bar, after the five service blocks, before the PIN block. Three
states, and only these:

| State                                | Reads                        | Tone                       |
| ------------------------------------ | ---------------------------- | -------------------------- |
| something in flight                  | "2 em voo · 3 pedem gente"   | accent, with a tinted pill |
| nothing in flight, something waiting | "3 pedem gente"              | amber text, no fill        |
| nothing at all                       | "Em voo · nada em voo agora" | muted, dimmed              |

The handle is the only part of the status bar that keeps its words in a narrow window
(the service blocks drop to icons below 980 px): it is the one block that counts work
that needs a person.

## The drawer

330 px wide, anchored bottom-right, pushing nothing — it sits over the content with the
toast stack moving above it. Four sections, always in this order, each hidden when empty:

1. **Em voo agora** — what the app is doing right now: the gesture, the subject, how far
   it got, how long ago it started. One line per gesture.
2. **Esperando a trava** — gestures queued behind the Fantasma's single lock, naming who
   is ahead ("atrás da coleta de Materiais").
3. **Pedem gente** — a gate holding a bill, a stalled run with no outcome, a "provável".
   Each line is a door to the screen that owns the decision.
4. **Terminou agora** — the last **2 hours**, at most **5 lines**, cleared when the app
   closes. Each line carries its verdict and how long ago.

## Invariants

1. **The tray never acts.** Every line is a door to the screen that owns the gesture;
   the tray itself has no verbs. A second place to approve a bill would be a second
   source of truth about it.
2. **The handle counts people, not machines.** "3 pedem gente" counts decisions waiting
   for a human — not services that happen to be busy.
3. **Nothing in flight is a state, not an absence.** The handle still reads, dimmed; it
   never disappears, because an operator learns where it lives.
4. **Terminou agora is not a journal.** 2 hours and 5 lines, cleared on restart; the full
   story lives in [Logs](11-logs.md). The tray is what is happening, not what happened.
5. **The remembered height is per station**, between 220 px and 70% of the window. A
   window that shrinks trims the height without forgetting the value.
6. **Below 700 px of window height** the drawer stops being a drawer and becomes a
   full-height sheet on the right, 320 px wide.

## Keyboard

`Ctrl J` opens and closes it, and the binding lives in **Configurações → Atalhos**
(see [10-settings](10-settings.md)) — not hard-coded in this surface.

## Sample data

Handle: "2 em voo · 3 pedem gente". In flight: "Coletando Materiais · página 4 · há 3 min
· 62%". Waiting: "Consulta ao Everest — atrás da coleta de Materiais". Needs a person:
"JOÃO CARLOS PRADO — corrida parada, desfecho desconhecido". Finished: "MARTA LEMOS
FERREIRA — pedido criado · há 6 min"; "Coleta de Convênios — recusada · há 1 h".

## What this needs that does not exist yet

- A single place that knows every gesture in flight. Today each store knows its own, and
  the status bar reads a boolean per service.
- A short-lived ring for finished gestures (2 h, 5 entries, memory only).
