# The rules that outrank aesthetics

These hold on every screen. Each one was paid for — most of them by an incident
where the screen said something that was not true and somebody acted on it. A
new look may draw them any way it likes; it may not drop them.

Screen-specific rules live in each screen's **Business rules** and **Must
survive the redesign**.

## 1. Patient data is everywhere — mockups are fictitious

Names, procedures, surgery dates, medical records: under Brazilian law (LGPD)
they are personal health data. No mockup, preview or example uses a real one,
and no capture of the current app is reference material. The cast is in
[06-sample-data.md](06-sample-data.md).

## 2. The screen never asserts what nobody verified

The expensive error of this app is not ugliness. It once sent eleven bills to
the robot behind a green "sem registro" that was hard-coded — all eleven were
already invoiced. Since then:

- **Absence looks like absence.** A verdict nobody asked for (_não checado_)
  draws **nothing** — no pill, no grey badge. It is the normal state of every
  bill in the queue, and a warning shown on everything warns about nothing.
- **Green exists only after somebody asked** and the answer came back.
- **Unknown is a dash, never a zero.** A count nobody could read is `—`; a
  catalog never collected shows no count at all, because "0 itens" would say
  the ERP answered an empty registry.
- **"Not asked yet" is its own state**, and it asserts nothing: the screen
  says nobody has asked and what asking costs — it never draws a result.
- **What was read from elsewhere carries when it was read**, in sight, never
  behind a tooltip. A mirror or a portal reading ages, and an old one presented
  as current is an invisible error.

## 3. Colours carry meaning, and only these

| Role                  | Means                                                                   | Never used for                                    |
| --------------------- | ----------------------------------------------------------------------- | ------------------------------------------------- |
| **Warn (amber)**      | A human must decide something here, now                                 | Selection, decoration, "information"              |
| **OK (green)**        | Somebody checked, and it is fine                                        | A default, an unknown, "configured"               |
| **Danger (red)**      | Destructive gesture, or a failure that happened                         | A business ending that is not a failure           |
| **Accent**            | Interaction and selection                                               | A state of the data                               |

- **A tint is never the only signal.** A flagged row also says why, in the
  row; a state also has a word.
- **Selection is accent, never amber** — a selection is a state, not an alert.
  When selection and a flag meet on one row, the selection wins visibly.
- These roles are **not themeable**: a custom palette may repaint surfaces and
  the accent, never make danger green ([03-platform.md](03-platform.md)).

## 4. One bill, one click

No button processes the whole queue, and no gesture writes to the ERP or to a
portal in bulk. The agreement with the other side is to grow the batch slowly,
and the ERP does not cancel a closed conferência. Every writing gesture acts on
one thing, names it, and asks first.

## 5. Resume, never redo

A bill that stopped between the closed conferência and the created order is
**not** redone — redoing is the definition of a duplicated invoice. A send to
a portal that got no answer is **not** re-sent — the portal may have the file.
The screen says where the thing stopped and offers the gesture that recovers
it, never a generic "tentar de novo".

## 6. A long gesture shows a sign of life before it answers

Commands to the services answer only when they finish — sometimes minutes
later. Every long gesture marks its in-flight state **before** it leaves (a
spinner on the control itself, a label such as "Ligando…"), or the whole
operation happens invisibly and the person clicks again.

## 7. Failures are named, never dressed as empty

- A refusal never becomes an empty result, and "nothing found" never covers
  "could not look". On the Consulta, "no rows" can mean "safe to invoice"
  downstream — a failure must never wear that sentence.
- When a screen reads several sources, each fails **by name** and the others
  keep their answers.
- A partial answer says how partial ("mostrando 80 de N", "2 clientes não
  responderam").
- A table that goes blank says why **inside its own box**, with its header
  still in place — the header says the table exists and only the rows are
  missing.
- The other side's words are shown verbatim beside ours; the app never
  interprets a service's log text.

## 8. Every kind of empty has its own sentence

"Nothing saved yet", "nothing pinned yet" and "nothing matches what you
typed" are three different sentences. One sentence for all of them sends the
person looking for something that is right there under another filter.

## 9. The gate says what is missing, on the thing that is missing it

Nothing enters the robot's queue unfinished: patient, date of use,
atendimento, convênio → code (when the report names one), doctor → code (when
the report names one), the client's colinha, and a lot on every item not
struck out. When one is missing the button does not act — and the screen says
**which**, on the bill. (The quantity is always present as a number; whether a
zero should hold the gate is an open question in
[screens/03-review.md](screens/03-review.md).)

## 10. Two registries answering one name is a question

Two homonymous insurers are two different billings. When a name matches two
ERP registries, the field is flagged and the gate holds until a human picks —
getting it right by luck half the time is worse than asking.

## 11. Codes are data

ERP codes, lots, references and price tables are shown **exactly as they
are** — monospace, never re-cased, never trimmed of leading zeros (`015` is not
`15`), never reformatted. Values that came from a portal are shown as the
portal wrote them, even when they look wrong.

## 12. Secrets never come back to the screen

A stored password or key is drawn as a masked placeholder (`••••••••••`),
never as its value. A blank secret field means "I did not touch it", never
"erase it" — otherwise saving the folder beside it would erase the credentials.

## 13. Destructive gestures ask first, and never sit beside Save

Deleting asks inline (a confirmation in place of the gesture, not a second
modal), hovers in the danger role, and lives away from the button that saves —
one slip from each other is one slip too close.

## 14. Notices are pending subjects; toasts are feedback

- A **notice** (behind the bell) is something that asks for a human decision
  or happened without a hand. It leads to the screen where its subject lives
  and resolves nothing by itself.
- A **toast** confirms an act the person just did: five seconds, never stored.

## 15. Services have more than two states

A service can be up, starting, stopped, unconfigured, owned by somebody else
(usable, never stoppable from here), busy, or unreachable — and an unreachable
service is **never** drawn as a broken credential. "Busy" overrides the label:
ready _and_ working is the common case.

## 16. Dates and language

- Dates on screen are `dd/mm/aaaa` with four-digit years; the "day" is the
  operator's local day, never UTC.
- All text is Brazilian Portuguese, in the operator's vocabulary (_informe_,
  _colinha_, _lote_, _convênio_, _atendimento_ stay untranslated). The app has
  an English locale as structure, so labels must tolerate other lengths.

## 17. Keyboard

- `Ctrl K` focuses the global search; `Ctrl ⇧ K` focuses **the** filter of the
  current screen — exactly one per screen, and the hint is drawn only where the
  shortcut works.
- `Esc` closes what is open (and returns from the search to where the person
  came from).
- A gesture that exists only for the mouse also has a keyboard path (the
  column reorder grip answers the arrow keys).
