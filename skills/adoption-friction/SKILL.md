---
name: adoption-friction
description: Audit why users would still hesitate to adopt a product even if it were free, grounded in the repo's own docs and code, and publish the result as a ranked artifact with five mitigations per friction. Use this whenever the user asks what stops people from switching, signing up, going live or sticking with the product, asks about onboarding drop-off or activation, asks "if this were free what would still be hard", or wants a friction, churn-risk or adoption-blocker review, even if they don't say "friction" or "artifact".
metadata:
  version: "1.0.0"
  tags:
    - product
    - onboarding
    - adoption
    - artifact
---

# Adoption friction audit

Price is the easiest objection to name and the least interesting one. The
frictions that survive "it's free" are the ones that decide whether a signup
becomes a regular user: risk to their reputation, work before value, what they
must leave behind, fear of being stuck, trust in us, skills they lack, other
people who have to change, and choices they can't undo. This skill finds those
for the product in the current repo and turns them into a ranked, actionable
page.

## 1. Ground it in the repo first

Generic category lists are worthless here. The value is in frictions that are
true of this product today, stated with the fact that makes them true. Before
writing anything, read whatever explains the product to its users:

- docs, onboarding or setup flow, marketing or README copy
- changelog, roadmap or plan docs (to know what is built vs. promised)
- the settings or configuration surface (defaults, what needs support, what
  can't be changed later)

Work out who the user is, what they have to give up or change to adopt this,
and who else is affected when they do. Where the code or docs show something
specific (a capability that exists in only one mode, a step that needs a human
on our side, a feature marked "coming soon" or "on request", a default that
hurts new users), cite it in plain words. If you're unsure whether something
exists, check the code rather than guess. Never invent features.

## 2. Find the frictions

Price is off the table, so look for the other costs of adoption:

- risk to the user's own reputation, customers, users or data if it goes wrong
- work they must do before they see any value
- things they have to give up, migrate or leave behind
- dependence on us: exit, export, deletion, portability, what happens if we stop
- trust in us: security, privacy, compliance, who else touches the data
- skills, access or people they need that they may not have
- behaviour change for others, not just the person who signed up
- irreversible or hard-to-undo choices made early
- anything else the repo reveals that a competitor would use against us

Keep only the ones that apply. Pick the eight that matter most and rank them by
how often each would stop a first-time user from becoming a regular one.

## 3. Answer in chat first

Give a brief version in chat before building anything, so the user can react
and reorder. One short paragraph per friction: the title, why it bites, what
the product does today, and the one or two mitigations that matter most. Close
with the two you'd ship first. Then build the page.

If the user has already reacted or clearly only wants the page, skip straight
to the artifact.

## 4. Build the artifact

Publish one readable page. Utilitarian, not flashy. For every friction:

1. A one-line title in plain language.
2. Two sentences on why it bites, written from the user's side.
3. A "Today" box: what the product currently does, factually, from the repo,
   visually set apart from the mitigations.
4. Exactly five mitigations, numbered 1.1, 1.2 and so on. Each is a bold
   three-to-five-word label plus one or two sentences of what we'd build,
   change or remove. Mix cheap and expensive. Where a mitigation is "make an
   existing feature the default" or "surface something already built", say
   so. Tag each with effort as a small chip: S (days), M (a sprint),
   L (roughly a quarter).

Add a sticky jump bar to the eight sections and an effort legend at the top.
Close with "If we only did two": the two mitigations to ship first and one
sentence on why. Title it "<Product name> Adoption Friction". It must work in
light and dark and at phone width; the artifact-design guidance covers the
mechanics.

## Writing rules

Short sentences. No em-dashes, no marketing tone, no hedging. Write from the
user's side of the screen and name things the way they would. A mitigation
should be concrete enough that an engineer could scope it from the sentence.
