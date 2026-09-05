# Contributing a flow

1. **Explore once with the agent** (or by hand in the `/phone` browser UI) and note, for
   every step, the exact accessibility label/identifier that worked and what proved the
   step succeeded. The browser **流程** panel records acknowledged actions and can
   download a v1 JSON file directly.
2. **Compile it into a flow file** under `<app>/<flow>.json`. If `<app>/app.json` does
   not exist yet, add it (`id` = directory name, `bundle`, `name`, `category`).
   Add `app`, `category`, `risk`, `locale`; put `"draft"` in `tags` until step 4 is done.
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
5. **Open a PR.** CI checks that `index.json` is current and that every flow passes the
   CLI validator. Say in the PR which device and locale you verified on.

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
