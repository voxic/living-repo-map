---
name: Living repo map
description: >-
  Use this when a developer wants to explore an unfamiliar repo slice in the
  checkout they already have. Bound a slice, then write MAP.md plus a
  clickable repo-map/ locally. Do not launch Cloud Agents. Do not open a PR.
---
# Living repo map

A local walk of this checkout. You open `repo-map/index.html` and follow the slice: where work enters, what it hits next, where state is owned, what must not break. Every node opens a real file.

That is the product. It is how you learn the tree without asking an agent to narrate it. Leave the files in the working tree. Do not treat this as a change to ship.

It is not a wiki, not a job board, and not a reason to launch another agent.

## Done means

Someone who has never opened this slice can follow two to five real flows end to end by opening the local HTML.

Not done: a prettier `ls`, a package inventory, a sitemap of zone pages, a list of prompts, a Cloud Agent launch, or a pull request.

## Hard rules

- Do this in the checkout you are already in. Do not clone a second copy of the target. Do not create GitHub PATs.
- Do not launch Cloud Agents, background agents, or any other agent to walk the tree or to act on the map. Read the files yourself.
- Do not create a branch, commit, or pull request for the map. Write `MAP.md` and `repo-map/` as working-tree files and stop. Commit only if the user explicitly asks to keep the map.
- Do not swallow the monorepo. Bound a slice before you walk the tree.
- Do not invent file paths. If a path does not exist in the checkout, delete the node.
- Do not invent edges. A hop must be a real import, call, route registration, or queue publish you saw in the checkout.
- Do not invent incidents. Traps come from the code or from docs inside the slice.
- Do not write prompts, kickoff blocks, or Copy-prompt CTAs into the map. The map explains the slice; it does not hand out jobs.
- Do not compile the whole project. Do not run the full testsuite.
- Touch no product code. The map adds `MAP.md` and `repo-map/` and nothing else.

## Inputs

1. This checkout. Stay on the branch you are already on.
2. The slice to walk. If the user named one, use it. If not, infer the smallest honest slice from the ask (authentication, billing, the module they opened) and state that assumption.
3. Optional: a change they are about to make. It decides which flows are worth walking. If there is none, walk the flows the entry points make obvious.

## Step 1. Bound the slice

Write the bound in one short block before you read the tree:

- In scope: the modules, packages, or docs trees the map may read, plus files they clearly import.
- Out of scope: tests not tied to those modules, operators, packaging, unrelated apps, the rest of docs.
- If you must peek outside scope to understand a hop, list the path as peeked in `MAP.md` and stop there.

Never write "the whole repo" as the job.

## Step 2. Find the walks

Work outward from where work starts, not down the directory tree.

1. **Entry points.** How does work enter this slice? HTTP or RPC handlers, CLI commands, scheduled jobs, webhook or queue consumers, UI routes. Each entry point is a door into a walk.
2. **Follow each door.** Entry to handler to domain logic to the thing that owns state. Stop when you leave the slice or reach storage, an external API, or a rendered screen.
3. **Keep two to five walks.** Pick the load-bearing ones. Two honest walks beat nine guesses.
4. **Mark where truth lives.** On the hop that owns state (table, package, config, external API). Say which nearby copies are caches or generated, because that is what strangers get wrong.
5. **Attach traps to the node they constrain.** Ordering or monotonicity requirements, authz checks, HA or retry assumptions, anything a future reader must not break. One line, and only when the code or in-slice docs actually say so.

Every hop carries a `why`: what this node does for the flow, or what breaks without it. Never restate the filename.

## Step 3. Write `MAP.md`

`MAP.md` at repo root is the structured source. `repo-map/` renders it. The HTML must not contain a node that is not in `MAP.md`.

Start with the slice bound, then one fenced `yaml` block per walk:

```yaml
walk: token-refresh
title: Refresh an expired session token
entry: POST /api/v1/session/refresh
steps:
  - id: route
    paths: [src/api/routes/session.ts]
    why: Registers the refresh route and rejects requests without a cookie.
  - id: handler
    paths: [src/api/handlers/refresh.ts]
    why: Validates the refresh token and mints the new pair.
    trap: The old token must be revoked before the new one is returned, or a replay wins the race.
  - id: store
    paths: [src/auth/store/tokens.ts]
    why: Writes the new pair and marks the old one revoked in one transaction.
    truth: Tokens table is the only owner of session state. The in-process cache is a copy.
touches: [src/api/handlers/refresh.test.ts, docs/auth/sessions.md]
```

Rules for the file:

- `paths` are checkout-real. `steps` are ordered; that order is the walk.
- `why` is required on every step. `truth` and `trap` only where they apply.
- `touches` lists what moves when that walk changes: callers, tests, docs. It is how a reader sizes a change.
- Peeked paths go in their own short section.

## Step 4. Build the walk

Vanilla HTML plus CSS plus SVG in `repo-map/`. No app build, no JavaScript beyond what the walk itself needs. Opening `repo-map/index.html` from `file://` must work, so vendor anything you need.

- `index.html` — the hub. The slice bound in one block, then the walks, each with its entry point and a one-line summary. This is a list of walks, not a sitemap.
- `walk-<id>.html` — one page per walk. The flow as a directed SVG: entry on the left or top, each hop in order. Clicking a node shows its `why`, its paths, and any `truth` or `trap`. Give every page next and previous controls so a reader can step the flow without going back to the hub.
- `graph.json` — nodes and edges generated from `MAP.md`, so the pages and the machine-readable file cannot drift.
- `styles.css`.

Node paths link to the file in this checkout (relative from `repo-map/`) and show the plain path as text so the page still reads from `file://`. A GitHub blob URL on the current branch is optional extra, not the primary link.

Add zone or inventory pages only when they render the same walks. Do not introduce a second taxonomy.

## Step 5. Verify locally

Check every path in `MAP.md` and `graph.json` against the checkout. Print a verification table in the reply. Fix or delete every `no`.

Then spot-check the walk itself, by opening the HTML:

1. The hub lists the walks and the slice bound.
2. One walk has at least three real hops in a sensible order.
3. One node opens a real source file in this checkout.
4. Each `truth` names an owner, not a directory.
5. Each `trap` is traceable to code or an in-slice doc.

Fix what fails before you stop. A map with fictional nodes is worse than no map, because the next reader trusts it.

Do not open a PR. Do not push. Point the user at `repo-map/index.html` in this working tree.

## Step 6. Report back

- Local path to `repo-map/index.html`
- The slice bound, and the walks with their entry points
- Verification: paths checked, nodes removed
- Anything you could not resolve inside the slice, listed as peeked
- That the map is uncommitted working-tree files, unless the user asked to keep it
