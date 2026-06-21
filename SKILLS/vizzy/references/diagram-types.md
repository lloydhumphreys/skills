# Diagram types

Vizzy parses a **practical subset** of Mermaid. Each ` ```mermaid ` fence is one diagram;
its first keyword picks the type. Unknown lines are ignored, never fatal — partial diagrams
still render. The standalone `architecture` map is a `vizzy`-fence diagram (see below).

The examples show the diagram **body**; place each inside a ` ```mermaid ` fence (or a
` ```vizzy ` fence for `architecture`).

## Sequence — who talks to whom, and what they exchange

```
sequenceDiagram
  actor User
  participant Web as Web Client
  Web->>API: POST /login {email, pw}
  API-->>Web: 200 {token}
  API->>API: validate
  Note over API: rate limited
```

**Arrows:** `->>` call, `-->>` dashed reply, `-x`/`--x` lost (✗), `-)`/`--)` async,
`->`/`-->` line with no head, `<<->>`/`<<-->>` bidirectional. Text after `:` is the
message. `participant X as Label` aliases; `actor X` draws a person. `Note over X: text`
(also `left of` / `right of`) pins a note to a lifeline.

**Combined fragments** wrap a run of messages in a labeled box. One compartment: `loop`,
`opt`, `critical`, `break`. Multiple compartments split by a dashed divider:
`alt`…`else`, `par`…`and`, `critical`…`option`. Close every fragment with `end`; they
nest. The text after the keyword is the condition shown on the fragment's tab / divider.

```
sequenceDiagram
  participant A
  participant B
  loop every minute
    A->>B: ping
    alt healthy
      B-->>A: pong
    else timeout
      B-->>A: retry
    end
  end
```

Tint a fragment's tab, border, and dividers with a `frame` directive in the `vizzy` fence,
named by keyword + condition: `frame loop every minute color:blue`. To group consecutive
messages into labeled, tinted **swimlanes**, use `phase` directives. Both live in
[`annotations.md`](annotations.md).

## Flowchart — control / branching flow

```
flowchart TD
  start([Start]) --> check{OK?}
  check -->|yes| go[Proceed]
  check -->|no| stop[Halt]
  subgraph Backend
    api[API] --> db[(Postgres)]
  end
```

**Node shapes:** `[rect]`, `(rounded)`, `([stadium])`, `[[subroutine]]`, `[(database)]`,
`((circle))`, `{diamond}`, `{{hexagon}}`, `>banner]`, `[/parallelogram/]`, `[\alt\]`,
`[/trapezoid\]`, `[\alt/]`, `(((double circle)))`.

**Links:** `-->`, `---`, `-.->` (dashed), `==>` (thick), `<-->` (bidirectional),
`<--` (reverse), `--o` / `--x` (circle / cross head), `o--` / `x--` (circle / cross at the
*source* end), `o--o` / `x--x` (both ends), `~~~` (invisible). Labels via `-->|text|` or
inline `A -- text --> B`. Connect many at once with `&`: `A & B --> C & D`.
`subgraph Name … end` groups nodes.

**Labels** are plain text — don't put HTML in them. Vizzy tolerates `<br>` so pasted
Mermaid renders, but prefer a `desc:` subtitle or a shorter label; other tags (`<sub>`,
`<b>`, …) render literally. Quote a label to include characters that would break parsing:
`A["text with (parens) & [brackets]"]`. For bold/italic use Mermaid backtick markdown:
``A["`**bold** and _italic_`"]``. There's no thick-bidirectional edge — write `<-->` (or
`==>`), never `<==>` (it leaks a stray node).

`flowchart` and `graph` are interchangeable keywords.

## Architecture — the static service map (`vizzy` fence)

Best for "these services talk, and here's the payload." A standalone `vizzy` block whose
first word is `architecture`:

```
architecture LR My System
group data Data Layer
service web Web Frontend
service api API
service db Postgres in data
web -> api : POST /login {creds}
api -> db : SELECT user
```

`service <id> [Label] [in <group>]` (`node` is a synonym for `service`),
`group <id> [Label]`, and `A -> B : payload` (also `<->` bidirectional, `-->`). The text
after `:` renders as a chip on the edge. A `desc <id>: …` directive (declared anywhere in
the same fence) adds a subtitle.

## Class / ER / State

```
classDiagram
  class User {
    +id: UUID
    +name: String
  }
  User --> Order : places
```

```
erDiagram
  CUSTOMER ||--o{ ORDER : places
  ORDER ||--|{ ORDER_ITEM : contains
  PRODUCT }o..o{ CATEGORY : "filed under"
  CUSTOMER {
    bigint id PK
    text email UK
    text name
  }
```

Entities render as **tables**: an `ENTITY { … }` block lists attributes as
`type name [key]` rows (key ∈ `PK` / `FK` / `UK`; a trailing `"comment"` shows as an `(i)`
badge on the row). Relationships use the full crow's-foot vocabulary — left/right
cardinality `|o` `||` `}o` `}|` (and mirrors `o|` `o{` `|{`) meaning
zero-or-one / exactly-one / zero-or-many / one-or-many — on a solid identifying line (`--`)
or a dashed non-identifying line (`..`). Text after `:` labels the relationship.

```
stateDiagram-v2
  [*] --> Idle
  Idle --> Running : start
  Running --> [*]
```

To tint a node in a class/ER/state/sequence diagram, use a `vizzy`-fence `style <id> …`
override (Mermaid `style`/`classDef` only apply inside `flowchart`/`graph`). See
[`annotations.md`](annotations.md).

## Gantt — a project timeline / schedule

```
gantt
  title Q3 Launch
  dateFormat YYYY-MM-DD
  section Design
    Wireframes    :active, wires, 2026-06-08, 1w
    Visual design :crit, visual, after wires, 1w
  section Build
    Frontend      :fe, after visual, 2w
    Launch        :milestone, launch, after fe, 0d
```

- `dateFormat` sets the date parser (default `YYYY-MM-DD`). `section <Name>` groups rows.
- Task line: `Label : [status,] [id,] <start>, <end|duration>`.
  - **status** (optional): `done`, `active`, `crit`, or `milestone`.
  - **id** (optional): a handle other tasks depend on; auto-generated if omitted.
  - **start**: a date or `after <id>`. **end**: a date or a duration — `Nd` / `Nw`
    (`0d` makes a milestone point).
- `excludes weekends` / `excludes monday,tuesday` / `excludes 2026-06-15` grey-shades those
  columns. `tickInterval 1week|1day|1month` + `weekday monday` set the axis ticks;
  `axisFormat %Y-%m-%d` formats tick labels. A "today" marker shows when in range
  (`todayMarker off` hides it).

## Timeline — what happened, in order

```
timeline
  title History of Social Media
  section Early
    2002 : LinkedIn
    2004 : Facebook : Flickr
  section Growth
    2006 : Twitter
    2010 : Instagram : Pinterest
```

`section <Name>` groups periods into a tinted band (optional). Period line:
`<time> : <event> [: <event> …]` — the first `:`-segment is the time label, each following
segment is an event card below the axis. A line starting with `:` adds more events to the
previous period.

## Journey — a user's experience step by step, each scored

```
journey
  title My working day
  section Go to work
    Make tea: 5: Me
    Go upstairs: 3: Me, Cat
  section Go home
    Go downstairs: 5: Me
```

`section <Name>` groups steps. Step line: `<label> : <score> : <actor>, <actor>` —
`score` is a 0–7 satisfaction rating (the marker is colored red→amber→green by score);
actors are an optional comma-separated list.

## Pie chart — share of a whole

```
pie showData title Pets adopted
  "Dogs" : 386
  "Cats" : 85
  "Rats" : 15
```

`pie`, then one `"Label" : value` slice per line (quotes optional). Optional `showData`
(before or after `title`) appends each slice's raw value to its legend entry; `title` may
sit on the `pie` line or its own line. Non-numeric or negative values are skipped.

## XY chart — bar and line plots

```
xychart-beta
  title "Monthly revenue"
  x-axis [jan, feb, mar, apr]
  y-axis "Revenue (USD)" 0 --> 12000
  bar  [5000, 6000, 7500, 8200]
  line [4800, 6100, 7300, 9000]
```

First keyword is `xychart-beta` (or `xychart`). `x-axis [...]` are the column labels;
`y-axis "Title" min --> max` titles the axis (the numeric range is optional — both bounds
clamp the scale, otherwise it auto-scales). Add one or more `bar [...]` / `line [...]`
series of numbers, index-aligned with the x-axis — overlay a `bar` and a `line` for a
combo. `xychart-beta horizontal` flips the chart (categories down the side).

## Mind map — brainstorms and branching ideas

The "fun" map: a central topic with **colourful branches** fanning out into rounded,
branch-tinted chips. Best for brainstorms, personas, and shaping ideas.

```
mindmap
  root((Empathy Map))
    Think and feel
      Thoughts
      Emotions
    See
      Reactions
      Environment
```

The first content line is the **root** (a centred pill). **Indentation builds the
hierarchy**: a more-indented line is a child of the nearest shallower line above. Top-level
branches split left/right, each getting its own colour, inherited by its children. A mind
map leans on the tree — for labeled relationships between concepts, use a concept map.

## Concept map — domain models and relationships

The structured "diagram" sibling: boxed concepts with **labeled dashed relationships**,
laid out radially. Best for domain models, design docs, and "how X fits together."

```
conceptmap
  root((Authentication))
    Identity
      OAuth providers
      Passkeys
    Sessions
      Access token
      Refresh token
  OAuth providers --|issues|--> Access token
  Refresh token --|mints|--> Access token
```

Same indentation grammar as a mind map (root + nested branches), rendered as boxes with
straight spokes. A line containing a link operator is a **cross-link** — a labeled
relationship between two named nodes, drawn dashed and *not* part of the indentation:
`A --> B`, `A -->|rel| B`, `A -- rel --> B`, or `A --|rel|--> B`. An unknown endpoint
becomes a floating node. Root shape `((circle))`; branches default to rounded, or use
`(rounded)` / `[rect]`.

## Comments

`%%` inside a `mermaid` fence; `#` inside a `vizzy` or `architecture` fence.
