# opencli-browser Command Reference

## Inspect

| command | purpose |
|---------|---------|
| `browser state` | Snapshot: text tree with `[N]` refs, scroll hints, hidden-interactive hints, `compounds (N):` sidecar. |
| `browser state --source ax` | Accessibility-tree snapshot. Use for custom controls, portals, iframe contents, stale React re-renders. Cross-origin iframes are best-effort. |
| `browser state --compare-sources` | Metrics-only DOM vs AX comparison (counts + sizes, no page text). |
| `browser find --css <sel> [--limit N] [--text-max N]` | CSS query, returns `{nth, ref, tag, role, text, attrs, visible, compound?}` per match. |
| `browser find --role <r> --name <n>` | Semantic locator (also `--label`, `--text`, `--testid`). Prefer over raw CSS when accessible labels exist. |
| `browser frames` | List cross-origin iframe targets for `--frame` on `eval`. |
| `browser screenshot [path]` | Viewport PNG. No path → base64. Prefer `state` for structure. |
| `browser screenshot --annotate [path]` | Visual ref map with `[N]` overlays. Use for icon-only controls or visual layouts. |

## Get (read-only)

| command | returns |
|---------|---------|
| `browser get title` / `get url` | plain text |
| `browser get text <target> [--nth N]` | `{value, matches_n, match_level}` |
| `browser get value <target> [--nth N]` | `{value, matches_n, match_level}` |
| `browser get attributes <target> [--nth N]` | `{value: {attr: val}, matches_n, match_level}` |
| `browser get text --role option --name Travel` | Semantic locator read (same flags as `find`). |
| `browser get html [--selector <css>] [--as html\|json] [--depth N] [--children-max N] [--text-max N]` | Raw HTML or structured tree. JSON tree nodes: `{tag, attrs, text, children[], compound?}`. Truncation reported via `truncated:`. |

## Interact

| command | notes |
|---------|-------|
| `browser click <target> [--nth N]` | Returns `{clicked, target, matches_n, match_level}`. |
| `browser click --role button --name Submit` | Semantic click. Ambiguous locators return candidates instead of clicking. |
| `browser hover [target]` | Trigger hover menus/tooltips. Returns `{hovered, ...}`. |
| `browser focus [target]` | Focus without typing — useful before `keys`. Returns `{focused, ...}`. |
| `browser dblclick [target]` | Double-click via native mouse events. |
| `browser check [target]` / `browser uncheck [target]` | Ensure checkbox/radio/aria-checked state. `check` returns `{checked, changed, kind, ...}`. |
| `browser upload [target] <file...>` | Attach local file(s) to `input[type=file]` via CDP. |
| `browser drag [source] [target]` | Mouse drag. Works for mouse-listener libs; native HTML5 `dataTransfer` may need fallback. |
| `browser type [target] <text>` | Click then type. Returns `{typed, autocomplete, ...}`. `autocomplete: true` → combobox opened; send `keys Enter` or click suggestion. |
| `browser fill [target] <text>` | Exact replacement (no keyboard events). Returns `{filled, verified, actual, ...}`. |
| `browser select [target] <option>` | Native `<select>` only. Match by label then value. For custom dropdowns: `state → click trigger → state → click option`. |
| `browser keys <key>` | `Enter`, `Escape`, `Tab`, `Control+a`, etc. Runs on focused element. |
| `browser scroll <direction> [--amount px]` | `up` / `down`. Default 500px. |

## Wait

```bash
browser wait selector "<css>" [--timeout ms]    # until selector matches
browser wait text "<substring>" [--timeout ms]  # until text appears
browser wait download [pattern] [--timeout ms]  # until Chrome download; reports {downloaded, filename, url, state}
browser wait time <seconds>                     # hard sleep — last resort only
```

Default timeout 10000ms. SPA routes, login redirects, lazy-loaded lists all need `wait` before `state`/`get`.
`browser wait download` requires Browser Bridge extension 1.0.8+.

## Extract

- **`web read --url <url>`** — One-shot Markdown reader. Expands relevant same-origin iframes. For AJAX pages: add `--wait-for "<selector>" --wait-until networkidle --diagnose`.
- **`browser eval <js> [--frame N]`** — IIFE returning JSON. Read-only: no form submits, clicks, or navigations.
- **`browser extract [--selector <css>] [--chunk-size N] [--start N]`** — Chunked Markdown extraction. Returns `{url, title, total_chars, next_start_char, content}`. Loop on `next_start_char` until null.

## Network

```bash
browser network                           # shape preview + cache key list
browser network --detail <key>            # full body for one cached entry
browser network --filter "field1,field2"  # AND-semantics: keep entries whose shape contains ALL fields
browser network --all                     # include static resources
browser network --raw                     # full bodies inline — use sparingly
browser network --ttl <ms>               # cache TTL (default 24h)
```

Entry shape: `{key, method, status, url, ct, size, shape, body_truncated?}`.
Cache at `~/.opencli/cache/browser-network/`.

## Tabs & Session

| command | purpose |
|---------|---------|
| `browser tab list` | `[{index, page, url, title, active}]`. Use `page` as `targetId`. |
| `browser tab new [url]` | Open tab; prints `page` string. |
| `browser tab select [targetId]` | Set default tab. Use `--tab <targetId>` per-command without changing default. |
| `browser tab close [targetId]` | Close by `page`. |
| `browser back` | History back on active tab. |
| `browser close` | Release owned browser session. |
| `browser bind --session <n>` | Bind current tab to session. |
| `browser unbind --session <n>` | Detach bound session without closing tab. |

## Target Contract & match_level

```
<target> ::= <numeric-ref> | <css-selector>
```

| match_level | meaning | action |
|-------------|---------|--------|
| `exact` | All strong attrs agreed | Proceed |
| `stable` | Tag+IDs agree, soft attrs drifted | Proceed; verify with `get value` if chaining writes |
| `reidentified` | Original ref gone; CLI found unique replacement | Double-check before chaining more writes |

Error codes: `not_found` / `stale_ref` → re-`state`; `selector_not_found` → check `error.candidates[]`; `option_not_found` → check `error.available[]`.
