---
name: code-explainer
description: Explain a code change visually with Vizzy — take the diff of the current branch (vs the default branch), a PR number/URL, a branch name, or a commit range, and author a *.vizzy.md explainer doc covering the interesting parts of the change (not the full diff — GitHub has that) with tinted change-map diagrams, flow diagrams, and a curated key-files table. Use when asked to explain what changed on a branch or in a PR, visualize a diff, or make a visual change summary / review companion.
metadata:
  version: "1.0.0"
  tags:
    - vizzy
    - diagrams
    - code-review
    - git
    - diff
---

# Code explainer

Turn a diff into a picture. This skill reads the change set — current branch, a PR, or a
commit range — figures out what it *means* (not just which lines moved), and writes a
`*.vizzy.md` explainer document the Vizzy app renders: a change map of the affected
architecture, the runtime flow that changed, and a table of the key files.

The audience is a reviewer or teammate who hasn't seen the work. **This is not a diff
viewer** — GitHub already shows every hunk. Cover only the *interesting* bits: the parts
that change behavior, introduce or remove a component, or are subtle enough that a
reviewer would want them explained. The value you add is the mental model — what the
change does, where it lands in the system, how behavior differs afterwards — not
coverage.

**This skill builds on the `vizzy` skill.** If the vizzy skill's instructions aren't
already loaded in this conversation, invoke it (Skill tool → `vizzy`) before authoring —
it carries the diagram grammar, and its `references/annotations.md` covers every directive
used below (`style`, `note`, `desc`, `hint`, `cellstyle`, `frame`, `phase`).

## 1 — Scope the change set

Work out what to explain from the argument given:

- **No argument → the current branch.** Find the default branch
  (`git symbolic-ref --short refs/remotes/origin/HEAD`, falling back to `main`/`master`),
  then:
  ```
  git merge-base origin/<default> HEAD          # the base
  git log --oneline <base>..HEAD                # commit intents
  git diff --stat <base>...HEAD                 # committed shape
  git status --porcelain && git diff HEAD --stat  # uncommitted work counts too
  ```
  Include uncommitted/staged changes — "what we changed" means the working tree too. If
  you're *on* the default branch with no commits ahead, explain just the uncommitted
  changes; if there's nothing at all, say so and stop.
- **A PR number or URL** → `gh pr view <n> --json title,body,baseRefName,headRefName,files`
  and `gh pr diff <n>`. When you need more context than the diff shows and the branch
  isn't checked out, fetch it read-only — `git fetch origin pull/<n>/head` then
  `git show FETCH_HEAD:<path>` — never check it out over the user's working tree.
- **A branch name or explicit range** (`feature-x`, `abc123..def456`) → diff it against
  its merge-base with the default branch, or use the range as given.

## 2 — Understand it before drawing

Same rule as all Vizzy authoring: **explore the real code first.**

- Read the commit messages and PR description — they state intent.
- Read the diff, then read *around* it: the callers of a changed function, the flow a new
  endpoint sits in, the component a deleted file used to serve. The diagram must show the
  change in its architectural context, not floating in space.
- Group hunks by **concern**, not by file — "adds retry to the payment client" is one
  concern even if it touches four files.
- **Triage hard.** Split the change set into the part that carries meaning and the churn
  that follows from it — renames, lockfiles, generated files, import shuffles, test
  scaffolding, mechanical call-site updates. The churn gets, at most, one line of prose
  ("plus the ~40 call-site renames that fall out of this"); it never gets a node, a
  table row, or a diagram.
- Classify each *interesting* component: **added**, **modified**, or **removed** — this
  drives the color coding below.

## 3 — Author the explainer doc

### Where it goes

Write to **`vizzy/changes/<slug>.vizzy.md`** in the target repo — slug from the branch
name (`feature/retry-queue` → `retry-queue`) or `pr-<n>`. The app discovers it
automatically. If the repo can't have a root `vizzy/` folder (rare; e.g. a
case-insensitive clash with an app folder named `Vizzy/`) or already keeps its vizzy docs
somewhere else, follow the repo's existing convention instead.

These docs are **review artifacts, not documentation** — don't commit them unless asked.
Suggest adding `vizzy/changes/` to `.gitignore`; the app still shows gitignored docs.

### Structure

Frontmatter title `Changes: <branch or PR title>`, with `created`/`lastEdited` set to
today. Then:

1. **A TL;DR prose card** (2–4 sentences): what the change does, why, and the scope line
   (`origin/main..HEAD, 9 commits, 14 files` or `PR #123`). State the color legend here
   in words: *green = added, orange = modified, red = removed*.
2. **The change map** — almost always the anchor diagram. A flowchart or architecture map
   of the affected slice of the system: the interesting touched components **plus their
   unchanged neighbors** for orientation. Tint by change type and annotate each changed node with
   its one-line delta:
   ````
   ```mermaid
   flowchart LR
     web[Web] --> api[API] --> queue[Retry queue] --> pay[Payment client]
   ```

   ```vizzy
   style queue fill:green
   style pay fill:orange
   note queue: new — buffers failed charges, exp. backoff
   note pay: now idempotent; accepts a retry key
   hint queue: [Source](../../src/payments/retry-queue.ts)
   ```
   ````
   Leave unchanged neighbors untinted. Add a `hint <id>: [Source](<path>)` pointing each
   changed node at its source file — paths resolve relative to the doc's folder and
   `vizzy lint` verifies them; non-markdown files open in the OS's default editor, so the
   reader can jump straight into the code.
3. **The flow that changed** — when runtime behavior changed, a sequence diagram of the
   *new* flow, with notes marking exactly which steps are new or different. For a
   materially rerouted control flow, a small **before → after** pair of flowcharts beats
   annotations on one.
4. **Key-files table** (optional) — a Markdown table (`File | Change | What`) of *only*
   the files worth opening, color-coded via a `vizzy` fence:
   ```
   cellstyle Change = added: green
   cellstyle Change = modified: orange
   cellstyle Change = removed: red
   ```
   A handful of rows, each earning its place — the five files that carry the change, not
   all 200 touched by a rename. If every interesting file already appears as a hinted
   node on the change map, skip the table entirely.

Pick **2–4 diagrams that fit this change** — a pure refactor may need only the change map
and the table; a behavior change earns the sequence diagram; a `gitGraph` of the branch is
worth it only when the commit sequence itself tells the story.

### Quality bar

- **Explain the delta, not the diff.** "Charges now retry with backoff instead of failing
  the request" — never "modified payment.ts lines 40–90".
- **Interesting beats complete.** Completeness is GitHub's job. Leaving boring changes out
  is correct, not sloppy — a doc that explains three subtle things well beats one that
  inventories fourteen files. If nothing about a change would surprise a reviewer, it
  doesn't belong in the doc.
- **Intent over mechanics** on edges and notes: show what's exchanged (`POST /charge
  {amount, retry_key}`), and *why* a thing changed when the commit message says.
- **Small enough to grok in a minute.** One concern per diagram; split a sprawling change
  set into sections by concern rather than drawing one mega-map.
- **Accurate.** Every node, edge, and note traces back to something in the diff or the
  surrounding code you actually read. No speculation about code you didn't open.
- **Consistent legend.** green = added, orange = modified, red = removed — palette names,
  never raw hex. Don't use these three colors for anything else in the doc.

## 4 — Verify and show your work

1. **`vizzy lint <file>`** — fix every error; treat `broken-node-ref` and
   `broken-file-ref` warnings (usually a `hint`/`note`/`style` id typo) as must-fix.
2. If the Vizzy app is running, drive it: `vizzy state`, `vizzy goto <doc>`, then
   `vizzy focus <id>` on the most important changed node so the reader's eye lands there.
3. If it isn't, mention the doc path and offer `vizzy render <file> --whole-doc` for a
   shareable PNG.
4. Close with a one-paragraph text summary of the change in chat — the doc is the
   deliverable, but the user shouldn't have to open it to know what you found.
