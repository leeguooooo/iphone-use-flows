# Contributing a flow

1. **Explore once with the agent** (or by hand in the `/phone` browser UI) and note, for
   every step, the exact accessibility label/identifier that worked and what proved the
   step succeeded. The browser **流程** panel records acknowledged actions and can
   download a v1 JSON file directly.
2. **Compile it into a flow file** under `<app>/<flow>.json`. If `<app>/app.json` does
   not exist yet, add it (`id` = directory name, `bundle`, `name`, `category`).
   Add `app`, `category`, `risk`, `locale`; put `"draft"` in `tags` until step 4 is done.
   If the flow has inputs, add `example_inputs` with harmless values so the nightly canary
   can run it; if it changes anything on the device (creates files, sends, toggles), tag it
   `no-canary`. Set `app_version_min` only when you know an older version lacks the UI.
3. **Validate offline:**
   ```bash
   chmod 600 <app>/<flow>.json
   iphone-use-mcp flow validate <app>/<flow>.json
   python3 scripts/build-index.py
   ```
4. **Run it on a real iPhone** from the file path, then from the store:
   ```bash
   PHONE_REMOTE_TOKEN=… iphone-use-mcp flow run <app>/<flow>.json [--input k=v]
   IPHONE_USE_FLOWS_SOURCE="$PWD" IPHONE_USE_FLOWS_DIR=/tmp/flows iphone-use-mcp flow update
   IPHONE_USE_FLOWS_DIR=/tmp/flows PHONE_REMOTE_TOKEN=… iphone-use-mcp flow run <app>/<flow>
   ```
   Then fill in `verified_on` (device, iOS, app version, date) and drop the `draft` tag.
   For Apple system apps put the **iOS version** in `app_version`; that is what compat compares.
5. **Open a PR** — one command does the fork, branch, `app.json`, index rebuild, and PR:
   ```bash
   iphone-use-mcp flow publish <flow>.json --as <app>/<flow> --alias Health --alias 健康 \
     --note "iPhone 17 Pro Max · iOS 26 · zh-CN, ran 3×"
   ```
   (`aliases` are the app's foreground label in each language; they let the MCP surface
   your flow the moment an agent looks at that app.) CI checks that `index.json` is
   current and that every flow passes the CLI validator. An unverified file opens as a
   draft PR.

## Reporting a broken flow

`iphone-use-mcp flow report <app>/<flow> --result @run.json --note "..."` files an issue
with the failed step and a redacted daemon result (typed text, screen labels, and element
lists are stripped). From an MCP client, `phone_flow_report` reuses the last failed
`phone_flow_run` automatically. Issues without a flow id or with private content are
closed.

## What will not be merged

- Steps that send, publish, pay, follow, like, comment, or delete without
  `"risk": "side_effect"`.
- Inputs meant to carry passwords, codes, tokens, or private message content.
- Coordinate-only taps without a following `wait_for` postcondition.
- Anything that automates around a platform's rate limits, risk controls, or consent
  screens (unknown alerts stop the flow; they are not tapped through).
- Flows whose labels are hard-coded in one language without a `locale` field.

## Repairing a broken flow

App updates break labels. Run the flow, read the failed step from the result, read
`/agent/elements`, fix that locator only, bump `verified_on` with the new app version,
and re-run before opening the PR. Do not add "tap again if nothing changed" retries —
the first tap may already have acted.
