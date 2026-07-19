# The `vizzy` CLI

If the Vizzy app is installed, it ships a `vizzy` command-line tool. Install it once from
the app menu: **Vizzy ▸ Install Command Line Tool…**. Everything here is optional — the app
live-reloads as you edit — but `vizzy lint` is worth running after authoring.

> **If a `vizzy` command fails with "command not found" (or otherwise looks uninstalled),
> the CLI just hasn't been linked onto your PATH yet — don't abandon it.** The tool ships
> inside the app. Tell the user to open the running Vizzy app and choose
> **Vizzy ▸ Install Command Line Tool…**, then re-run the same command. Only after that
> install step still fails should you treat the CLI as unavailable.

## Keeping this skill up to date

The Vizzy app can install and update *this very skill* for you — no need to copy an `npx`
line out of the docs:

```
vizzy skill install     # add the vizzy skill to your AI agents (Claude Code, Codex, Cursor, …)
vizzy skill update      # pull the latest version
```

Both shell out to the `skills` CLI (`github.com/lloydhumphreys/skills`) via `npx`, so Node.js
must be installed. Extra flags pass straight through — e.g. `vizzy skill install -g` for a
global install.

## Rendering diagrams to images

Render any file to PNGs without opening a window — handy for a README or the web:

```
vizzy render vizzy/architecture.vizzy.md            # one PNG per diagram → vizzy/assets/
vizzy render vizzy/                                  # every *.vizzy.md in a folder (recursive)
vizzy render auth.vizzy.md --whole-doc              # the whole file as one tall image
vizzy render auth.vizzy.md --scale 3 --dark        # hi-dpi, dark-mode render
vizzy render auth.vizzy.md -f jpeg --quality 0.85  # lossy export
vizzy render auth.vizzy.md -o docs/img             # write somewhere other than ./assets/
```

Options:

- `-o` / `--out-dir <dir>` — output directory (default `<file's dir>/assets`)
- `-f` / `--format <png|jpeg|tiff|heic|avif>` — image format (default `png`)
- `--quality <0…1>` — compression quality for the lossy formats (default `0.9`)
- `--scale <n>` — pixel scale factor (default `2`)
- `--whole-doc` — one image of the whole file instead of one per diagram
- `--dark` / `--light` — render in dark or light mode (default light)

Each diagram is written to `vizzy/assets/<file>-<slug>.png`; reference it from Markdown like
any other image.

## Linting

`vizzy lint <file.vizzy.md | dir>` checks for problems the renderer silently swallows (it
never fails a render — it just draws what it can, so mistakes are easy to miss). It prints
`path:line:col: severity: message [rule]` and **exits non-zero only on errors** — so an
agent or CI step can gate on it. Run it after authoring, especially to confirm every `hint`
file path resolves.

| Rule | Severity | Meaning |
|------|----------|---------|
| `broken-file-ref` | error | A `hint`, link, or image points at a file that doesn't exist, relative to the doc's folder |
| `unterminated-fence` | error | A ` ``` ` fence with no closing ` ``` `; the rest of the file gets swallowed into it |
| `unsupported-diagram` | warning | A ` ```mermaid ` type Vizzy can't render |
| `unsupported-html` | warning | An HTML tag other than `<br>` in a label |
| `phantom-edge` | warning | An unrecognized edge like `<==>` that leaks a stray node |
| `untagged-diagram` | warning | A ` ``` ` fence with diagram syntax but no `mermaid`/`vizzy` language tag, so it renders as a code block |
| `detached-annotation` | warning | A `vizzy` annotation block with no diagram before it to attach to; the whole block is dropped |
| `unknown-directive` | warning | A `vizzy` directive line whose keyword isn't recognized (a typo like `position` for `pos`) |
| `broken-node-ref` | warning | A `style`/`desc`/`pos`/`hint`/`payload`/`phase` directive targeting a node/edge/group/label that doesn't exist |
| `skipped-block` | info | A line a diagram parses but never draws (e.g. `autonumber`, `activate`/`deactivate`, `box`) |
| `prefer-desc` | info | A `<br>` a `desc:` directive would express more cleanly |
| `prefer-theme-color` | info | A raw `#hex` instead of a palette name / token |

## Publishing to the web

Share a diagram — or a whole folder — as a public page on usevizzy.com, the same links the
app's **Publish** menu creates:

```
vizzy login                        # sign in with GitHub (shared with the app); opens a browser
vizzy publish vizzy/auth.vizzy.md  # publish one document; prints + copies its link
vizzy publish vizzy/ --name "Acme Architecture"   # publish a whole folder as one workspace
vizzy logout                       # sign out (drop the stored token)
```

A single file publishes as its own page; a folder publishes every `.vizzy.md`/`.md` beneath
it as one browsable workspace with a file-tree sidebar. Republishing updates the same link.
Options: `--name`, `--dry-run`, `--open`, `--no-copy`, and `--token` (or set
`VIZZY_PUBLISH_TOKEN` for CI, where there's no browser to sign in through). **Everything you
publish is public** — don't publish anything confidential.

## Driving the live Vizzy window

When the Vizzy app is open, the same `vizzy` CLI can **drive the window on screen** — switch
documents, frame a node, open a hint, clear review comments, and read back what's showing.
This is how you *show your work*: after editing a diagram, jump the window straight to the
node you changed instead of describing it. The commands print JSON and no-op with a clear
message when the app isn't running.

```
vizzy state                  # what the window shows: current doc, zoom/pan, every diagram's
                             #   nodes and groups (with ids), review comments, and other docs
vizzy goto <document>        # switch the window to another doc — a name (`auth`) or a path
vizzy focus <id> [--zoom N] [--no-pulse]   # centre on a node or group and pulse a highlight
vizzy open-hint <nodeId>     # open a node's (i) hint window
vizzy comment delete <n>     # delete a review comment by its number
```

Always start with `vizzy state` to discover what you can act on — the node and group ids it
lists are what `focus` takes (a label works too; a group/subgraph frames the whole cluster),
`open-hint` takes a node id, and the doc names are what `goto` takes. A typical loop:

```
vizzy state                  # see what's open; grab a node id
vizzy focus api --zoom 1.5   # frame the API node you just edited
vizzy goto auth              # switch to auth.vizzy.md
vizzy open-hint api          # pop the API node's hint
```

Good moments to reach for it:

- **After an edit** — `vizzy focus <node>` so the reader's eye lands on what changed.
- **Working through review comments** (the window's numbered pins) — read them from
  `vizzy state`, fix each in the diagram, then `vizzy comment delete <n>` to clear it.
- **Walking someone through a flow** — `vizzy goto` + `vizzy focus` to drive the canvas as
  you narrate.
