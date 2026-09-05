<!-- `iphone-use-mcp flow publish` fills this in automatically; for a manual PR: -->

## `<app>/<flow>` — <name>

| | |
|---|---|
| app | `<bundle id>` |
| category | |
| risk | read_only · navigation · side_effect |
| locale | |

### Verified on
- <device> · iOS <version> · app <version> · <date>   (or: not yet — draft)

### Checklist
- [ ] `iphone-use-mcp flow validate` passes
- [ ] no literal typed text, credentials, or private content in the file
- [ ] `risk` is honest (`side_effect` for send/publish/pay/delete)
- [ ] `index.json` regenerated with `scripts/build-index.py`
- [ ] `app.json` has `aliases` for every UI language the app label was seen in
