---
name: Living repo map
description: >-
  Use this when a large repo is hard to enter and you need a durable map in the
  tree, then Cloud Agents to do the work the map finds. Bound a slice, write
  MAP.md plus a clickable static map with a Run in Cloud Agent button on each
  hotspot.
---
# Living repo map

Turn Cursor's understanding of a codebase into a map that lives in the repo, then kick Cloud Agents at the work the map names.

This is not a wiki and not a local CLI dump. The map is files in the tree. Cloud Agents clone the repo, so the map is still there after the laptop closes.

## Hard rules

- Do not clone the target repo onto the operator machine as the authoring workspace. Cloud Agents against the connected GitHub or Origin repo are the checkout.
- Do not swallow the monorepo. Bound a slice before anyone walks the tree.
- Do not invent file paths. If a path does not exist in the checkout, delete the node.
- Do not compile the whole project. Do not run the full testsuite.
- Do not create GitHub PATs.
- Propose before a Cloud Agent launch if the user has not already asked for that launch.
- A human reviews the PR. Nothing merges until they say so.
- Do not invent a URL that starts a Cloud Agent. Public docs only list [cursor.com/agents](https://cursor.com/agents), the desktop Cloud dropdown, Slack / GitHub / Linear `@cursor`, and the API. A static page cannot fire the API without a secret.

## Inputs

Need at least:

1. The connected repo (owner/name and default branch).
2. The slice to map. If the user named one, use it. If not, infer the smallest honest slice from the ask (for example authentication, billing, or the module they opened) and state that assumption.
3. Optional work targets the user already cares about (deprecated APIs, a migration, a risky module). If none, the map still produces blast-radius rows. Do not invent incidents.

## Step 1. Bound the slice

Write the bound in one short block before any agent runs:

- In scope: the modules, packages, or docs trees the map may read, plus files they clearly import.
- Out of scope: tests that are not tied to those modules, operators, packaging, unrelated apps, the rest of docs.
- If an agent must peek outside scope, it lists the path as peeked in `MAP.md` and stops.

Do not say "the whole repo" as the job.

## Step 2. Kick the map Cloud Agent

Launch a Cloud Agent on the repo. Starting ref: the default branch, unless the user named another.

The agent must:

1. Create a branch for the map (name it from the work, for example `living-map`).
2. Write a static site in `repo-map/`:
   - `index.html` hub
   - one page per zone the slice needs (flows, inventory, UI routes, or the equivalent in this repo)
   - `blast-radius.html` with the rows you will actually kick work from
   - `graph.json` (and vendor any JS into `repo-map/` so `file://` works)
   - `styles.css`
   - `cta.js` (clipboard + open Agents)
3. Write `MAP.md` at repo root. Machine-readable. Sections that match the pages. Each entry: `id`, `title`, `paths`, `touches`, `notes`, and a `kickoff` fenced prompt the next Cloud Agent can paste.
4. Vanilla HTML plus CSS plus SVG. No app build. Opening `repo-map/index.html` in a browser must work.
5. Verify every path in `graph.json` and `MAP.md` exists. Print a verification table in the PR body. Fix or remove any `no`.
6. Open a PR titled for the map, with that table.
7. Touch no product code except adding `repo-map/` and `MAP.md`.

Keep the prompt tight. Repeat the in-scope and out-of-scope lists. Say: never invent files.

## Step 3. Verify before kicking work

Spot-check in the PR, not in a local clone:

1. One graph or flow node opens a real source file.
2. One inventory row opens a real interface, factory, or registry file.
3. One UI or caller row opens a real file in that tree.
4. Each blast-radius row cites a real endpoint, symbol, or doc path.
5. Each blast-radius row has a working Cloud Agent CTA (button visible, prompt copies, Agents tab opens).

If more than two nodes are wrong, send a punch list to the same Cloud Agent. Do not kick work agents on a fictional map.

Host if you need it clickable: GitHub Pages from the map branch `/repo-map`, or a zip of `repo-map/` opened as local `index.html`. Do not claim Cursor hosted the site.

## Step 4. The map is the interface

Do not build a second product UI. The map is the UI.

- Developers open `repo-map/index.html` (hosted or local).
- Each blast-radius row is a job. The `kickoff` block in `MAP.md` is the Cloud Agent prompt.
- Every hotspot card on `blast-radius.html` (and any graph node that is also a job) has one primary button: **Run in Cloud Agent**.

### Cloud Agent CTA (required)

There is no public "start this agent" URL that accepts a prompt. Do not fake one.

Each CTA must:

1. Show a short label: **Run in Cloud Agent**.
2. On click: copy that row's `kickoff` prompt to the clipboard, then open [cursor.com/agents](https://cursor.com/agents) in a new tab.
3. After click, change the button or a status line to: `Prompt copied. Pick this repo, set starting ref to the map branch, paste, run.`
4. Include `data-repo` and `data-ref` on the button (owner/name and the map branch) so the status line can name them.
5. Keep the full kickoff in a `<textarea class="kickoff" hidden>` (or `hidden` attribute) next to the button so `file://` copy still works.
6. Style the button as the one action on the card. Do not hide it behind a menu.

Vendor this behavior in `repo-map/cta.js`. Rough shape:

```html
<article class="hotspot" id="brokering-v1">
  <h2>…</h2>
  <textarea class="kickoff" hidden>Read MAP.md first. …</textarea>
  <button class="cta" type="button"
    data-repo="owner/name"
    data-ref="living-map">Run in Cloud Agent</button>
  <p class="cta-status" hidden></p>
</article>
```

```js
// cta.js — copy kickoff, open cursor.com/agents
document.querySelectorAll(".cta").forEach((btn) => {
  btn.addEventListener("click", async () => {
    const card = btn.closest(".hotspot");
    const text = card.querySelector(".kickoff").value;
    try { await navigator.clipboard.writeText(text); } catch (_) {}
    const repo = btn.getAttribute("data-repo") || "this repo";
    const ref = btn.getAttribute("data-ref") || "the map branch";
    const status = card.querySelector(".cta-status");
    status.hidden = false;
    status.textContent = "Prompt copied. Pick " + repo + ", starting ref " + ref + ", paste, run.";
    window.open("https://cursor.com/agents", "_blank", "noopener");
  });
});
```

One hotspot per Cloud Agent unless the user asked to batch. A tight PR beats a 50-file cleanup.

## Step 5. Kick work Cloud Agents

For each chosen hotspot:

1. Read `MAP.md` first, then `repo-map/blast-radius.html` and `repo-map/graph.json`. Treat those as the architecture. Do not rediscover the repo.
2. Stay inside the blast radius those files describe. Every file changed should already appear in `MAP.md` or be a direct caller of a path in it. If you must touch something outside, add it to `MAP.md` in the same PR with one line why.
3. Open one focused PR. Body must include: map paths used, files changed vs files the map predicted, what you did not change and why, and links to the upstream docs or issues you cited.
4. A findings report plus one real code change is better than an empty PR. Empty theater (whitespace-only) is worse than an honest "no remaining callers" finding that updates `MAP.md`.
5. After the PR is open, save the agent URL and the PR URL.

Do not merge. Run Bugbot on the PR (`bugbot run` or `cursor review` if nothing posts). A human reviews in the IDE.

## Step 6. What to report back

- Map PR URL, branch, and hosted or local map path
- Verification: how many paths checked, how many removed
- Blast-radius rows, whether each CTA copied and opened Agents, and which rows you kicked
- Work PR URL(s) and whether Bugbot commented
