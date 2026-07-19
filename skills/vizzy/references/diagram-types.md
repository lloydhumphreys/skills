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

## Quadrant — a 2×2 prioritisation grid

```
quadrantChart
  title Reach vs Effort
  x-axis Low Reach --> High Reach
  y-axis Low Effort --> High Effort
  quadrant-1 Expand
  quadrant-2 Promote
  quadrant-3 Reconsider
  quadrant-4 Improve
  Campaign A: [0.3, 0.6]
  Campaign B: [0.45, 0.23]
```

First keyword is `quadrantChart` (everyday aliases `2x2` / `2-up`, which parse identically).
`x-axis <left> --> <right>` (the `--> <right>` half optional) and `y-axis <bottom> --> <top>`
label the axis ends; `quadrant-1`…`quadrant-4` title the four quadrants (numbered
anticlockwise from the top-right). `Label: [x, y]` plots a point — both coordinates run `0`–`1`
from the bottom-left corner, out-of-range values clamped. In a `vizzy` fence below,
`hint <label>: …` attaches a hover note (the marker itself is the target, no badge),
`style <label> fill:…` tints one marker, and `style <quadrant-title>` / `style quadrant-N
fill:…` tints a whole quadrant's background. See [`annotations.md`](annotations.md).

## Data table — reference data as a sortable grid

No special keyword: a standard **GFM Markdown table** in the prose (a header row, a
delimiter row, data rows) renders as a spreadsheet-style card the reader can sort and
filter. Alignment comes from the delimiter row (`:---` left, `:--:` center, `---:` right);
numeric columns are detected and right-aligned automatically. A table has no in-fence
title line, so title it with an id-less `title` / `desc` in a `vizzy` fence directly below:

````
| Service | Status    | Latency |
| ------- | --------- | ------: |
| api     | Done      |     250 |
| web     | In Review |      45 |
| db      | Blocked   |     120 |

```vizzy
title Rollout status
cellstyle Status = Done: green
cellstyle Status = Blocked: red
cellstyle Latency > 200: orange
```
````

`cellstyle` color-codes matching cells — the right way to make enums, tags, and statuses
scannable (see [`annotations.md`](annotations.md) for the full rule grammar); `tablewidths
120 80 160` pins column widths in source order.

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
map leans on the tree — for labeled relationships between concepts, use a concept map. To
pin a top-level branch to a chosen side of the root, add a `side <id> left` / `side <id>
right` directive in a `vizzy` fence below (unpinned branches auto-balance around it).

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
`A --> B`, `A --- B` (unlabeled), `A -->|rel| B`, `A -- rel --> B`, or `A --|rel|--> B`. An
unknown endpoint becomes a floating node. Root shape `((circle))`; branches default to
rounded, or use `(rounded)` / `[rect]`.

## Git graph — branching / merging history

```
gitGraph
  commit id: "init"
  branch develop
  checkout develop
  commit tag: "v0.2"
  checkout main
  merge develop
  commit type: HIGHLIGHT
```

First keyword is `gitGraph`; each line builds the history in order. `commit` adds one to the
current branch (options `id: "name"`, `tag: "v1.0"`, `type: NORMAL | HIGHLIGHT | REVERSE`);
`branch <name>` forks a new lane and switches to it; `checkout <name>` (or `switch`) makes
`<name>` current; `merge <name>` joins a branch back as a two-parent commit (accepts a `tag:`);
`cherry-pick id: "…"` references another commit. Time runs left-to-right; add `gitGraph TB:`
(or `TD:`) on the first line to run it downward with lanes side by side.

## Treemap — proportional-area breakdowns

```
treemap-beta
"Documents"
    "Reports": 40
    "Notes": 15
"Media"
    "Photos": 60
    "Videos": 25
```

First keyword is `treemap-beta` (the bare `treemap` alias works too); an optional
`title <text>` sits on its own line. Each remaining line is a node — `"Label": value` is a
**leaf** (quotes optional, value numeric), `"Label"` alone is a **section**; indentation
nests a line under the nearest less-indented one, and multiple top-level nodes are fine (a
forest, not a single root like a mind map). Tile area is proportional to value — a section's
area is the sum of its descendant leaves — laid out squarified (largest tiles first). Each
top-level node gets its own colour in source order, descendants fading with depth so a
subtree reads as one family. A zero, negative, or non-numeric value just makes the line a
valueless section, never an error. Best for disk-usage-style breakdowns: storage, budgets,
bundle sizes.

## Diff — "what changed"

Paste `git diff` output straight into a ` ```diff ` fence — no special syntax:

````
```diff
--- a/src/auth.ts
+++ b/src/auth.ts
@@ -12,7 +12,9 @@ export async function login(
   const user = await findUser(email)
-  const token = sign(user.id)
+  const token = sign(user.id, { expiresIn: '1h' })
   return respond(token)
```
````

Standard **unified diff**, exactly as `git diff` emits it: `diff --git` / `index` / mode
lines are understood, `---` / `+++` name the file, `@@` headers set the line numbers, and
`+` / `-` / space lines are the content. Each file in the fence becomes its own card with a
path + `+N −M` header, old/new line-number gutters, red/green row washes, and word-level
highlights on changed line pairs; new files, deletions, renames, and binary entries are
labelled on the header chip. Nothing is fatal — a bare hunk, or even loose `+`/`-` lines,
still renders. A `vizzy` fence directly below opts into `layout split` (side-by-side old/new
columns; `layout unified` switches back) and `collapse <N>` (fold unchanged runs longer than
N lines into a "⋯ N unchanged lines" pill). It degrades perfectly — GitHub and any Markdown
host already tint ` ```diff ` blocks red/green. Best for code-review notes, migration
walkthroughs, and "here's the fix".

## Comments

`%%` inside a `mermaid` fence; `#` inside a `vizzy` or `architecture` fence.
