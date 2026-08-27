---
name: Living repo map
description: >-
  Use this when a developer wants a visual walk of a large or unfamiliar repo
  slice instead of asking an agent to narrate it. Bound a slice, then write
  MAP.md plus a clickable repo-map/ that steps through the real flows: entry
  point, each hop, where truth lives, what breaks.
---
# Living repo map

A developer opens `repo-map/index.html` and walks the slice: where work enters, what it hits next, where state is owned, what must not break. Every node opens a real file.

That walk is the product. It lives in the repo, so it is still there after the laptop closes and the next Cloud Agent can read it instead of rediscovering the tree.

It is not a wiki, not a local CLI dump, and not a list of jobs.

## Done means

A developer who has never opened this slice can follow two to five real flows end to end without asking an agent to explain the repo.

Not done: a prettier `ls`, a package inventory, a sitemap of zone pages, or a board of Cloud Agent prompts.

## Hard rules

- Do not swallow the monorepo. Bound a slice before anyone walks the tree.
- Do not invent file paths. If a path does not exist in the checkout, delete the node.
- Do not invent edges. A hop must be a real import, call, route registration, or queue publish you saw in the checkout.
- Do not invent incidents. Traps come from the code or from docs inside the slice.
- Do not clone the target repo onto the operator machine as the authoring workspace. Cloud Agents against the connected GitHub or Origin repo are the checkout.
- Do not compile the whole project. Do not run the full testsuite.
- Do not create GitHub PATs.
- Touch no product code. The map adds `MAP.md` and `repo-map/` and nothing else.
- A human reviews the PR. Nothing merges until they say so.

## Inputs

1. The connected repo (owner/name and default branch).
2. The slice to walk. If the user named one, use it. If not, infer the smallest honest slice from the ask (authentication, billing, the module they opened) and state that assumption.
3. Optional: a change they are about to make. It decides which flows are worth walking. If there is none, walk the flows the entry points make obvious.

## Step 1. Bound the slice

Write the bound in one short block before any agent runs:

- In scope: the modules, packages, or docs trees the map may read, plus files they clearly import.
- Out of scope: tests not tied to those modules, operators, packaging, unrelated apps, the rest of docs.
- If an agent must peek outside scope, it lists the path as peeked in `MAP.md` and stops.

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

Vanilla HTML plus CSS plus SVG in `repo-map/`. No app build. Opening `repo-map/index.html` from `file://` must work, so vendor anything you need.

- `index.html` — the hub. The slice bound in one block, then the walks, each with its entry point and a one-line summary. This is a list of walks, not a sitemap.
- `walk-<id>.html` — one page per walk. The flow as a directed SVG: entry on the left or top, each hop in order. Clicking a node shows its `why`, its paths, and any `truth` or `trap`. Give every page next and previous controls so a reader can step the flow without going back to the hub.
- `graph.json` — nodes and edges generated from `MAP.md`, so the pages and the machine-readable file cannot drift.
- `styles.css`.

Node paths link to the file. Use a GitHub blob URL on the map branch when the repo is hosted there, and show the plain path as text either way so the page still reads from `file://`.

Add zone or inventory pages only when they render the same walks. Do not introduce a second taxonomy.

## Step 5. Verify, then PR

Before opening the PR, check every path in `MAP.md` and `graph.json` against the checkout. Print a verification table in the PR body. Fix or delete every `no`.

Then spot-check the walk itself, in the PR and by opening the HTML:

1. The hub lists the walks and the slice bound.
2. One walk has at least three real hops in a sensible order.
3. One node opens a real source file.
4. Each `truth` names an owner, not a directory.
5. Each `trap` is traceable to code or an in-slice doc.

If more than two nodes are wrong, send a punch list back to the same agent. A map with fictional nodes is worse than no map, because the next reader trusts it.

Open a PR titled for the map with the verification table. Run Bugbot (`bugbot run`, or `cursor review` if nothing posts). A human reviews.

If it needs to be clickable outside the IDE: GitHub Pages from the map branch `/repo-map`, or a zip of `repo-map/` opened as a local `index.html`. Do not claim Cursor hosted the site.

## Step 6. Report back

- Map PR URL, branch, and hosted or local map path
- The slice bound, and the walks with their entry points
- Verification: paths checked, nodes removed
- Anything you could not resolve inside the slice, listed as peeked

## Appendix. Handing a walk to a Cloud Agent

Optional. The map is done without this.

When a reader wants the work done rather than explained, add a `kickoff` fenced prompt to that walk in `MAP.md` and one **Copy prompt** button on the walk page. The button only copies. It does not open a tab, a deeplink, or Agents, and you must never invent a URL that starts a Cloud Agent.

Keep the full prompt in a hidden `<textarea>` next to the button so copying works from `file://`:

```html
<article class="walk" id="token-refresh">
  <h2>…</h2>
  <textarea class="kickoff" hidden>Read MAP.md first, then walk-token-refresh.html. …</textarea>
  <button class="cta" type="button">Copy prompt</button>
  <p class="cta-status" hidden></p>
</article>
```

```js
// repo-map/cta.js — copy kickoff only
document.querySelectorAll(".cta").forEach((btn) => {
  btn.addEventListener("click", async () => {
    const card = btn.closest(".walk");
    const box = card.querySelector(".kickoff");
    let copied = false;
    try {
      await navigator.clipboard.writeText(box.value);
      copied = true;
    } catch (_) {
      // navigator.clipboard is unavailable on file:// in some browsers.
      box.hidden = false;
      box.select();
      copied = document.execCommand("copy");
      box.hidden = true;
    }
    const status = card.querySelector(".cta-status");
    status.hidden = false;
    status.textContent = copied ? "Copied." : "Copy failed. Select the prompt manually.";
  });
});
```

An agent that takes such a prompt reads `MAP.md` and the walk page first and treats them as the architecture. It stays inside that walk and its `touches`; anything else it must change gets added to `MAP.md` in the same PR with one line why. One walk per agent, one focused PR, and no merge without a human.
