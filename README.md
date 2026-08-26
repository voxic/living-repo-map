# Living repo map

A Cursor skill. Kick it on a connected repo. It bounds a slice, writes a durable map into the tree (`MAP.md` plus a clickable `repo-map/`), then turns each blast-radius row into a Cloud Agent job.

Understanding stays in the repo. The work leaves as a PR a human reviews.

This is not a wiki and not a local CLI dump. Cloud Agents clone the target repo. Do not clone the target onto your laptop as the authoring workspace.

## Use it

1. Connect the target repo in Cursor (GitHub or Origin, read-write).
2. Open `SKILL.md` (or `.cursor/skills/living-repo-map/SKILL.md`) and run it in Agent or a Cloud Agent.
3. Name the slice, or let it infer the smallest honest one. Optional: name the work you already care about.
4. The map Cloud Agent opens a PR with `repo-map/` and `MAP.md`. Spot-check paths. Do not invent files.
5. Open `repo-map/index.html` (GitHub Pages or a local file). Each blast-radius row has a **Run in Cloud Agent** button. It copies the `kickoff` prompt and opens [cursor.com/agents](https://cursor.com/agents). There is no public URL that starts the agent for you.
6. In Agents: pick the repo, set starting ref to the map branch, paste, run. One hotspot per agent. Bugbot on the PR. Do not merge until a human says so.

## What lands in the target repo

- `repo-map/index.html` and zone pages (vanilla HTML, no build)
- `repo-map/graph.json`
- `MAP.md` at repo root, machine-readable, with `kickoff` blocks

The skill does not compile the target and does not touch product code except those files.

## Rules

- Do not swallow the monorepo.
- Do not create GitHub PATs.
- Do not claim Cursor hosted the map (Pages or local HTML).
- Nothing merges without a human.
