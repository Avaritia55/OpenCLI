# opencli-browser Recipes & Reference

## Compound Form Controls

Every date/time, `<select>`, and file input carries a `compound` field. Use it — do not regex attributes.

### Date family

```json
{"control": "date", "format": "YYYY-MM-DD", "current": "2026-04-21", "min": "2026-01-01", "max": "2026-12-31"}
```

`control`: `date | time | datetime-local | month | week`. Type using the exact `format` string.

### Select

```json
{"control": "select", "multiple": false, "current": "United States", "options": [...], "options_total": 137}
```

`options[]` capped at 50. `current` is always correct. If option isn't in `options[]`, call `browser select <target> "<label>"` — CLI matches against live DOM.

### File

```json
{"control": "file", "multiple": true, "current": ["report.pdf"], "accept": "application/pdf,image/*"}
```

Respect `accept` when telling the user what to upload.

`compound` appears in: `browser find`, `browser get html --as json`, `browser state` (as sidecar keyed by ref number).

---

## Recipes

### Fill a login form

```bash
opencli browser --session login open "https://example.com/login"
opencli browser --session login state
opencli browser --session login type 4 "me@example.com"
opencli browser --session login type 5 "hunter2"
opencli browser --session login get value 4                    # verify (autocomplete can eat chars)
opencli browser --session login click 6
opencli browser --session login wait selector "[data-testid=account-menu]" --timeout 15000
opencli browser --session login state                          # fresh refs on logged-in page
```

### Pick from a long native dropdown

```bash
opencli browser --session form state
# compound.options_total: 137, find ref [12] for <select name=country>
opencli browser --session form select 12 "Uruguay"
opencli browser --session form get value 12    # {value: "uy", match_level: "exact"}
```

### Pick from a custom React/Radix dropdown

```bash
opencli browser --session app state                   # find category trigger ref
# If unclear, use AX:
opencli browser --session app state --source ax       # look for combobox/button/listbox/option names
opencli browser --session app click 7                 # click trigger
opencli browser --session app state --source ax       # fresh refs after portal opens
opencli browser --session app click 12                # click option
opencli browser --session app get text 7              # verify visible selected label
```

Do NOT use `browser select` on custom dropdowns — it is for native `<select>` only.

### Scrape via network instead of DOM

```bash
opencli browser --session hn open "https://news.ycombinator.com"
opencli browser --session hn network --filter "title,score"
# find the /topstories entry, note its key
opencli browser --session hn network --detail topstories-a1b2
```

### Read a long article in chunks

```bash
opencli browser --session article open "https://blog.example.com/long-post"
opencli browser --session article extract --chunk-size 8000
opencli browser --session article extract --start 8000 --chunk-size 8000
# until next_start_char is null
```

### Cross-origin iframe

```bash
opencli browser --session checkout frames
# [{index: 0, url: "https://checkout.stripe.com/..."}]
opencli browser --session checkout eval "(() => document.querySelector('input[name=cardnumber]')?.value)()" --frame 0
```

`--source ax` may omit cross-origin iframe contents. Fallback: `browser frames` + `browser eval --frame`.

---

## Cost Guide

| command | rough cost | when to use |
|---------|-----------|-------------|
| `state` | medium | First call on any page, after every nav |
| `find --css <sel>` | small | Already know the selector |
| `get title` / `get url` | tiny | Sanity checks |
| `get text/value/attributes` | tiny | Verify one field |
| `get html` (raw) | can be huge | Avoid on unbounded pages; always add `--selector` + depth budget |
| `get html --as json --depth 3 --children-max 20` | medium | Reasoning about structure |
| `screenshot` | large | Only for visual pages (CAPTCHA, charts) |
| `extract` | medium/chunk | Long-form reading |
| `network` (default) | small | First look at APIs |
| `network --detail <key>` | varies | Pull one body |
| `network --raw` | huge | Only after `--filter` narrowed candidates |
| `eval "JSON.stringify(...)"` | controlled | Targeted extraction |

Rule: one `state` per page transition, one `find` per follow-up query, one `get`/`click`/`type` per action. >10 calls/page → consider `extract` or `network`.

---

## Pitfalls

- **Do not submit via `eval "document.forms[0].submit()"`** — modern sites intercept with JS handlers. Use `click` on submit button, or `open` the GET URL directly.
- **Do not reuse refs across a page transition.** `wait` then re-`state`. Old refs may `reidentify` onto a similarly-shaped element.
- **`match_level: reidentified` is a warning, not error.** Verify with `get text`/`get value` before chaining more writes.
- **Budget-aware commands silently cap.** `get html --as json` returns `truncated: {...}`. Raise `--depth`/`--children-max` or tighten selector.
- **`autocomplete: true` on `type` is not an error.** Suggestion popup open, value not committed. Send `keys Enter` or `click` the suggestion.
- **`network --filter` is AND-semantics.** `--filter "title,score"` keeps entries containing BOTH fields as path segments at any depth.
- **Screenshots are for humans, not agents.** Use `state` + `find` unless the page is genuinely visual.

## Troubleshooting

| symptom | fix |
|---------|-----|
| `doctor` red: "Browser not connected" | Start Chrome with `--remote-debugging-port=9222`, or install the extension |
| `attach failed: chrome-extension://...` | Disable 1Password / other CDP-hungry extensions |
| `selector_not_found` right after `state` | Page mutated; `wait selector "..."` then retry |
| `stale_ref` across every command | Reusing refs from prior page; re-`state` |
| `click` succeeds but nothing happens | Decorative wrapper; use `find` with narrower selector on inner element |
| `type` finishes but value wrong | Autocomplete/masked/React input; verify with `get value`, add `keys Enter` or re-type |
| Giant `get html` output | Add `--selector` + `--as json --depth 3 --children-max 20 --text-max 200` |
| Network cache seems stale | Bump `--ttl` down or clear `~/.opencli/cache/browser-network/` |
