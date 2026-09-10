# Diagrams

Shared by `map`, `flow` and `explain`. Mermaid only, in fenced blocks, so it renders on GitHub, in
editors and in most wikis, and stays diffable and greppable. An image cannot be reviewed in a pull
request, cannot be searched, and cannot be corrected by the next agent.

A diagram earns its place by showing something a table cannot: **ordering, concurrency, or shape**. A
diagram that lists things is a table drawn badly.

## Draw the topology, not the code

A diagram of boxes that are units, stores and queues survives every refactor inside them — the design
changed if the boxes changed. A diagram whose boxes are functions is a call graph, and it is wrong the
first time somebody extracts a method.

| Question the reader has | Diagram | Where |
|---|---|---|
| What talks to what | `flowchart LR` | `system.md`, `architecture.md` |
| In what order, across units | `sequenceDiagram` | `flows/` |
| What states can this be in | `stateDiagram-v2` | unit doc, when a status column exists |
| How does this mechanism work | `flowchart` plus a drawn table | `explain/` |

Nothing else. Class diagrams and mind maps do not answer a question anyone brings to a codebase. ER
diagrams are excluded on purpose: they are the schema redrawn, so they go stale on the next migration
and the schema file was already correct. Cite the schema and draw the relations that span services
instead — those are the ones no single schema file shows.

## Rules

- **Real participants only.** Every box is a unit, a store or an external system that exists and can be
  named in the code. No box called "Business Logic".
- **Ceiling of fifteen arrows.** Past that the diagram is decoration. Split it: `flows/<name>.md` and
  `flows/<name>-part2.md`, the first ending where the second begins.
- **Label every arrow** with what is sent: the endpoint, the message, the query. An unlabelled arrow says
  two things are connected, which the reader already assumed.
- **Async is dashed and labelled** `async`. Where a response returns before the work finishes, the
  diagram must show it — that gap is where the reader's mental model breaks.
- **One diagram per file.** A second diagram means a second question, which means a second file.
- Participant names equal unit names. A diagram that renames a service to fit the box costs every reader
  one translation.

## Sequence

```mermaid
sequenceDiagram
    autonumber
    participant C as Client
    participant API as api
    participant W as worker
    participant DB as Postgres
    C->>API: POST /orders
    API->>DB: INSERT orders
    API--)W: async order.created
    API-->>C: 202 Accepted
    W->>DB: UPDATE orders SET status
```

## Component

```mermaid
flowchart LR
    C[Client] --> API[api]
    API --> DB[(Postgres)]
    API -. order.created .-> Q[[Kafka]]
    Q --> W[worker]
    W --> DB
```

Stores use `[(…)]`, queues use `[[…]]`, services use `[…]`. Consistent shapes across every diagram mean
a reader learns the notation once.

## State

```mermaid
stateDiagram-v2
    [*] --> pending
    pending --> paid: payment.succeeded
    pending --> cancelled: timeout 30m
    paid --> shipped
    shipped --> [*]
```

Transitions carry the event or condition that fires them. A state diagram without labelled edges tells
the reader what is possible but never how to get there.
