# Living repo map

A Cursor skill. Kick it on a connected repo. It bounds a slice, writes a durable map into the tree (`MAP.md` plus a clickable `demo-map/`), then turns each blast-radius row into a Cloud Agent job.

Understanding stays in the repo. The work leaves as a PR a human reviews.

This is not a wiki and not a local CLI dump. Cloud Agents clone the target repo. Do not clone the target onto your laptop as the authoring workspace.

## Use it

1. Connect the target repo in Cursor (GitHub app, read-write).
2. Run [Living repo map](sand-workflow:living-repo-map) or paste `SKILL.md` into Agent / a Cloud Agent.
3. Name the slice (or let it infer the smallest honest one). Optional: name KTLO targets.
4. The map Cloud Agent opens a PR with `demo-map/` and `MAP.md`. Spot-check paths. Do not invent files.
5. Open `demo-map/index.html` (GitHub Pages or local file). Each blast-radius row has a `kickoff` prompt in `MAP.md`.
6. Kick one Cloud Agent per hotspot, starting ref = the map branch. Bugbot on the PR. Do not merge until a human says so.

## What lands in the target repo

- `demo-map/index.html` and zone pages (vanilla HTML, no build)
- `demo-map/graph.json`
- `MAP.md` at repo root, machine-readable, with `kickoff` blocks

The skill does not compile the target and does not touch product code except those files.

## Proof

The EMEA FE Demo Bash 2026 tape runs this skill on a fork of [keycloak/keycloak](https://github.com/keycloak/keycloak): authentication flows, SPIs, `js/apps/admin-ui`, then Cloud Agents for deprecated Identity Brokering API V1 and Organizations `getAll()`.

## Rules

- Do not swallow the monorepo.
- Do not create GitHub PATs.
- Do not create Slack bots.
- Do not claim Cursor hosted the map (Pages or local HTML).
- Nothing merges without a human.
