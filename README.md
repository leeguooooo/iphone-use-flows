# iphone-use-flows

The official flow registry for [iphone-use](https://github.com/leeguooooo/iphone-use):
reviewed, deterministic scripts that operate one iPhone app end to end, so an agent
can run a repeated phone task as **one command with no model and no screenshots**.

```bash
MCP="$HOME/Applications/iPhoneUse.app/Contents/MacOS/iphone-use-mcp"
"$MCP" flow update                       # mirror this repo into ~/.iphone-use/flows
"$MCP" flow list                         # what is installed (risk, verified, inputs)
"$MCP" flow list --category health
"$MCP" flow info  health/export-all      # metadata + step templates
PHONE_REMOTE_TOKEN=… "$MCP" flow run system/spotlight-search --input query=Health
```

MCP clients get the same surface as `phone_flow_list`, `phone_flow_info`,
`phone_flow_run`, and `phone_flow_update`.

## Layout

```
index.json              generated — id, path, sha256 of every flow (scripts/build-index.py)
<app>/app.json          {"id","bundle","name","category","description"}
<app>/<flow>.json       strict iphone-use flow v1 + registry metadata
```

A flow id is `<app>/<flow>`; both parts are lowercase slugs. `<app>` is the registry
directory (usually the app's short name), `category` groups apps (`system`, `health`,
`finance`, `im`, …).

## Flow file

Flow v1 is the `phone_run_steps` step list plus optional string inputs. The registry
adds optional metadata, all validated by the CLI:

| field | meaning |
|---|---|
| `app` | bundle id the flow operates (`com.apple.Health`) |
| `category` | lowercase slug; must match `app.json` |
| `risk` | `read_only` · `navigation` · `side_effect` — `side_effect` flows (send, publish, pay, delete) refuse to run without `--confirm` / `confirm=true` |
| `locale` | UI language the labels were recorded under (`en`, `zh-CN`); labels are locale-specific |
| `tags` | up to 8 slugs; `draft` marks a flow that has not been run on hardware yet |
| `verified_on` | `[{"device","ios","app_version","date"}]` — hardware runs that proved this exact file. Empty = unverified, shown as `verified: no` |

Rules every flow follows (the CLI enforces the mechanical ones):

- Prefer `tap_locator` / `tap_label` with an exact unique label or identifier. Zero or
  multiple matches send nothing. Coordinates are a last resort and must be followed by a
  `wait_for` that proves the screen changed.
- Guard every page transition with `wait_for` (application, present, absent). No long
  fixed sleeps: `after_ms` ≤ 3 s, `wait_for` ≤ 10 s, ≤ 60 s declared waiting in total,
  ≤ 24 steps. Split a long task into two flows at a natural checkpoint.
- Typed text is a named `input`; the file never contains a value. Never make an input
  carry a password, one-time code, session token, or private message.
- The flow stops at the first failed step and never retries by itself.

## Trust

`flow update` downloads `index.json`, then every listed file, checks each sha256, and
runs the same strict validator `flow run` uses before writing anything to
`~/.iphone-use/flows` (mode 0600). Files are pure JSON — there is no code in this
repository that runs on your Mac or phone. Only this official source is supported; the
CLI has no `sources add`. `IPHONE_USE_FLOWS_SOURCE=<dir|url>` overrides it for
development.

## Contributing and reporting

`flow publish` opens the PR for you (fork, branch, `app.json`, `index.json`, PR body);
`flow report` files an issue for a flow that stopped working, with the failure redacted
of private content. Agents using the bundled MCP get `phone_flow_publish` /
`phone_flow_report` and are told by `phone_elements` which installed flows fit the app
on screen (via each `app.json`'s `aliases`).

### Adding a flow by hand

See [CONTRIBUTING.md](CONTRIBUTING.md). Short version: record with the browser
**流程** panel or write JSON by hand, `flow validate` it, run it on a real phone, fill
in `verified_on`, add it under `<app>/`, run `python3 scripts/build-index.py`, open a PR.
