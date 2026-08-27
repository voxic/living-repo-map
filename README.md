# Living repo map

A Cursor skill that walks an unfamiliar repo slice in the checkout you already have, instead of launching Cloud Agents or asking an agent to narrate it.

Name a slice. It writes `MAP.md` plus a clickable `repo-map/` into the working tree and stops. No branch, no commit, no PR. You open `repo-map/index.html` and step the real flows: where work enters, what it hits next, where state is owned, what must not break. Every node opens a real file.

This is exploration, not a change to ship. Keep the files if you want them; otherwise they stay uncommitted.

This is not a wiki and not a local CLI dump. It also does not hand out work: the skill explains the slice and stops. No job prompts, no agents launched on your behalf.

## Use it

1. Open the target repo in Cursor, on the branch you want to look at.
2. Run `SKILL.md` (or `.cursor/skills/living-repo-map/SKILL.md`) in that same session. Do not send it to a Cloud Agent.
3. Name the slice, or let it infer the smallest honest one. If you are about to make a change, say so; it decides which flows are worth walking.
4. The first reply should bound the slice, then read this checkout. If it talks about remotes, the default branch, or launching a Cloud Agent, stop it and point it back at the skill.
5. It writes `MAP.md` and `repo-map/` locally and reports a verification table. It does not open a PR.
6. Open `repo-map/index.html` from the working tree (`file://` is enough) and walk. Spot-check that one node opens a real file and one walk has at least three real hops.

## What the walk shows

For the bounded slice only:

- **Entry points** — how work starts: HTTP or RPC, CLI, jobs, queue consumers, UI routes.
- **Two to five flows** — entry to handler to domain to the thing that owns state, in order, each hop with why it exists.
- **Sources of truth** — which table, package, config, or API owns the state, and which nearby copies are caches.
- **Traps** — the invariant on the node it constrains, drawn from code or in-slice docs.
- **Touches** — what moves when that flow changes: callers, tests, docs.

## What lands in the working tree

- `MAP.md` at repo root: the structured source, one block per walk, machine-readable
- `repo-map/index.html` (hub) and one page per walk, vanilla HTML with no build
- `repo-map/graph.json`, generated from `MAP.md` so the two cannot drift

The skill does not compile the project and touches no product code beyond those files.

## Rules

- Stay in this checkout. Do not launch Cloud Agents. Do not inspect remotes or the default branch to "honestly" bound the slice.
- Do not swallow the monorepo.
- No invented paths, no invented edges, no invented incidents.
- No job prompts written into the map.
- Do not create GitHub PATs.
- Do not open a PR for the map unless the user asks to keep it.
