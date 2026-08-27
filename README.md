# Living repo map

A Cursor skill that gives you a visual walk of an unfamiliar repo slice, instead of asking an agent to narrate it every time.

Point it at a connected repo and name a slice. It writes `MAP.md` plus a clickable `repo-map/` into the tree, then opens a PR. You open `repo-map/index.html` and step the real flows: where work enters, what it hits next, where state is owned, what must not break. Every node opens a real file.

The walk stays in the repo, so it is still there tomorrow and the next Cloud Agent can read it instead of rediscovering the tree.

This is not a wiki and not a local CLI dump. Cloud Agents clone the target repo. Do not clone the target onto your laptop as the authoring workspace.

## Use it

1. Connect the target repo in Cursor (GitHub or Origin, read-write).
2. Open `SKILL.md` (or `.cursor/skills/living-repo-map/SKILL.md`) and run it in Agent or a Cloud Agent.
3. Name the slice, or let it infer the smallest honest one. If you are about to make a change, say so; it decides which flows are worth walking.
4. The map agent opens a PR with `MAP.md`, `repo-map/`, and a verification table for every path.
5. Open `repo-map/index.html` (local file or GitHub Pages) and walk. Spot-check that one node opens a real file and one walk has at least three real hops.

## What the walk shows

For the bounded slice only:

- **Entry points** — how work starts: HTTP or RPC, CLI, jobs, queue consumers, UI routes.
- **Two to five flows** — entry to handler to domain to the thing that owns state, in order, each hop with why it exists.
- **Sources of truth** — which table, package, config, or API owns the state, and which nearby copies are caches.
- **Traps** — the invariant on the node it constrains, drawn from code or in-slice docs.
- **Touches** — what moves when that flow changes: callers, tests, docs.

## What lands in the target repo

- `MAP.md` at repo root: the structured source, one block per walk, machine-readable
- `repo-map/index.html` (hub) and one page per walk, vanilla HTML with no build
- `repo-map/graph.json`, generated from `MAP.md` so the two cannot drift

The skill does not compile the target and touches no product code beyond those files.

## Rules

- Do not swallow the monorepo.
- No invented paths, no invented edges, no invented incidents.
- Do not create GitHub PATs.
- Do not claim Cursor hosted the map (Pages or local HTML).
- Nothing merges without a human.

Handing a walk to a Cloud Agent is optional and lives in the appendix of `SKILL.md`. The map is done without it.
