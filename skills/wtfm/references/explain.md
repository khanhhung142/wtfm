# Command: explain — teach a human → `explain/<topic>.md`

The only artifact written for a person. Everything else optimises for an agent that can be handed facts
in a table; a person cannot start from a table, because they have no place to put the facts yet.

**Write these only if decision 6 says a human reads this manual.** For an agents-only manual, `explain/`
stays empty, and that is a correct outcome rather than an unfinished one.

The reader is a competent engineer who does not know **this** system, and does not necessarily know the
technology it leans on. Assume the profession, not the stack.

## Teach the idea, not the current call chain

This is the one mode where prose is the point, so it is also where staleness sneaks back in. The rule
that keeps an explainer true for years: **explain the problem and the mechanism, not today's code.**

| Rots in a month | Lasts for years |
|---|---|
| "The handler calls `validate`, then `enqueue`, then returns 202" | "The response returns before the work runs, so the client cannot rely on it having happened" |
| A transcript of the current call chain | The primitive the design rests on, worked through |
| Function names in the narrative | The failure the ordering exists to prevent |

Both sentences take the same effort to write. The second one survives three refactors, because it is
about the design and not the code. Someone who understands the primitive can read the current call
chain themselves in five minutes; someone handed the call chain never learns the primitive.

## Structure

1. **The problem.** What breaks without this thing. Three or four concrete situations from the actual
   product, not abstractions of them. A reader who cannot see the problem cannot judge the solution and
   will memorise it instead.
2. **The primitive.** The one mechanism the solution rests on: a data structure, a protocol, a
   guarantee. Explain it standalone, with a worked example carrying real numbers. Nobody understands a
   mechanism from its definition; they understand it from watching one instance run.
3. **The shape.** How the pieces fit. One diagram.
4. **The walkthrough.** One request or job, start to end, with the state drawn before and after. Show
   the values changing. Name units and stores, not functions — the units are what the reader is
   building a model of.
5. **What goes wrong.** The failure the design exists to prevent, and the invariant that prevents it.
   Show the wrong version beside the right one and name why the wrong one loses.
6. **What is not built yet.** Plainly, with the revision. A person reading about a system half of which
   is imaginary needs to know which half.
7. **Where to go next.** Links to the flow and unit docs for the same topic.

## Devices that work

- **Draw the state.** A table of a queue, a cache or a row, before and after, beats any sentence
  describing the transition.
- **Real numbers.** `score = 1030` and `now = 1000` are followable. `score = t + delay` is algebra the
  reader has to run themselves.
- **Wrong-then-right.** Two small diagrams, one labelled as the mistake, is the fastest way to make an
  ordering constraint stick.
- **Answer the question you just raised.** Each section should provoke the next one's question. When it
  does not, the section is in the wrong place.

## Rules

- Frontmatter is `kind: explainer` plus `written:`. No `read:` revision: an explainer about a
  primitive is not a claim about a revision, and stamping one invites re-verification it does not need.
- No code coordinates in the body. They break the rhythm for a reader who is not going to open a file
  mid-paragraph. Put links in a closing section, or link the flow doc.
- Define a term the first time it is used, in the sentence that uses it. A reader who has to scroll back
  has stopped reading.
- Do not restate the unit doc. If the explainer is a paraphrase of a table, delete it — the table was
  better.
- One topic per file, and the topic is something a reader wants to understand, not a component in the
  architecture. "How scheduled work survives a restart" is a topic. "The scheduler package" is a unit
  doc.
- Read [diagrams.md](diagrams.md) first. Explainers use flowcharts and drawn structures, rarely
  sequence diagrams.
