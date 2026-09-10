# Command: explain — teach a human → `explain/<topic>.md`

The only artifact in the manual written for a person. Everything else optimises for an agent that
can afford to be told facts in a table. A person cannot start from a table, because they have no
place to put the facts yet.

The reader is a competent engineer who does not know **this** system, and who does not necessarily
know the technology it leans on. Assume the profession, not the stack.

## What makes this different

| Agent docs | Explainer |
|---|---|
| Facts in tables | One idea per section, in order |
| `file:line Symbol` on every claim | Citations at the end, or a link to the flow doc |
| Complete | Only the spine, with detours cut |
| Status tags everywhere | Says plainly what does not exist yet |
| Sequence diagrams | Flowcharts and drawn data structures |

## Structure

1. **The problem.** What breaks without this thing. Three or four concrete situations from the
   actual product, not an abstraction of them. A reader who cannot see the problem cannot judge the
   solution, and will memorise it instead.
2. **The primitive.** The one mechanism the solution rests on: a data structure, a protocol, a
   guarantee. Explain it standalone, with a worked example carrying real numbers. Nobody understands
   a mechanism from its definition; they understand it from watching one instance run.
3. **The shape.** How the pieces fit. One diagram.
4. **The walkthrough.** One request or one job, start to end, in prose, with the state drawn before
   and after. Show the values changing.
5. **What goes wrong.** The failure the design exists to prevent, and the ordering or invariant that
   prevents it. Show the wrong version next to the right one and name why the wrong one loses.
6. **What is not built yet.** Plainly. A person reading about a system half of which is imaginary
   needs to know which half.
7. **Where to go next.** Links to the flow and unit docs for the same topic.

## Devices that work

- **Draw the state.** A table of a queue, a cache or a row, before and after, beats any sentence
  describing the transition.
- **Real numbers.** `score = 1030` and `now = 1000` are followable. `score = t + delay` is algebra
  the reader must run themselves.
- **Wrong-then-right.** Two small diagrams, one labelled as the mistake, is the fastest way to make
  an ordering constraint stick.
- **Answer the question that was just raised.** Each section should provoke the next section's
  question. When it does not, the section is out of order.

## Rules

- No `file:line` in the body. It breaks the reading rhythm for a reader who is not going to open
  the file mid-paragraph. Put the coordinates in a closing section, or link the flow doc.
- Define a term the first time it is used, in the sentence that uses it. A reader who has to scroll
  back has stopped reading.
- Do not restate the unit doc. If the explainer is a paraphrase of a table, delete it: the table was
  already better.
- One topic per file, and the topic is a thing the reader wants to understand, not a component in
  the architecture. "How scheduled work survives a restart" is a topic. "The scheduler package" is
  a unit doc.
- Read [references/diagrams.md](diagrams.md) first. Explainers use flowcharts and drawn structures,
  rarely sequence diagrams.
