# Worked examples

Concrete patterns the agent should follow when invoking Ditto from Hermes.

> All examples invoke `heyditto` (the npm-installed binary from `@heyditto/cli` 1.1.3+). It's identical to `ditto` but never collides with Apple's `/usr/bin/ditto` on macOS.

## Pattern 1 — recall: "what did I say about X"

```bash
# 1. broad semantic search
heyditto search "X" --output json

# 2. fetch full text for the top hit(s)
heyditto fetch <pairId from step 1> --output json
```

Summarize the fetched memory text for the user, then optionally save a follow-up if they react with a clarification or correction.

## Pattern 2 — opinion / preference

```bash
# 1. find subjects matching the topic
heyditto subjects "X" --top-k 5 --output json

# 2. expand the strongest subject(s) into memories
heyditto memories <subjectId from step 1> --output json

# 3. optionally fetch full text for the most relevant memory
heyditto fetch <pairId> --output json
```

## Pattern 3 — explicit save

```bash
heyditto save "<durable fact, 1-3 sentences>" \
  --source hermes \
  --source-context "<optional: project, file, channel>" \
  --output json
```

Confirm to the user: "Saved."

## Pattern 4 — targeted update

When the user asks to correct or extend an existing saved memory, fetch an
outline first, then patch block IDs with a base revision.

```bash
# 1. get stable block IDs
heyditto fetch <pairId> --memory-format outline --output json

# 2. apply a precise block edit using the revision from the prior save/update response
heyditto update <pairId> \
  --edits-json '[{"op":"replace_text","blockId":"2","find":"old","replace":"new","expectedCount":1}]' \
  --base-revision <revision> \
  --output json
```

If the current revision is unknown, use full replacement instead:
`heyditto update <pairId> --content-file revised.md --output json`.

## Pattern 5 — publish only on explicit request

```bash
heyditto publish <pairId> --title "<optional title>" --privacy-mode scan_and_block --output json
heyditto unpublish --share-id <shareId> --output json
```

## Pattern 6 — graph traversal

```bash
# 1. find seed memory
heyditto search "X" --output json

# 2. expand its network
heyditto network <pairId from step 1> --limit 30 --output json
```

## Pattern 7 — first-run, no key configured

If `heyditto status --output json` reports `"apiKey": { "present": false, ... }` or any command exits with `error: no Ditto API key configured`:

1. Hermes should have prompted at install time, but if the prompt was skipped:
   ```bash
   hermes skills config ditto
   ```
   Re-runs the prompt for `DITTO_API_KEY`.
2. If the user prefers to paste directly, fall back to:
   ```bash
   heyditto login <key>
   ```
   Writes `~/.config/heyditto/cli/config.json` (mode 0600) — persists across shells without `.zshrc` editing.
3. Confirm with `heyditto status --output json` (should show `"source": "env"` or `"source": "config"`), then retry the original command.

## Common args reference

| Command | Required | Optional |
|---|---|---|
| `heyditto save <content>` | content | `--source <s>`, `--source-context <c>` |
| `heyditto search <q>...` | one or more queries | `--include-public`, `--filter-username <u>` |
| `heyditto fetch <id>...` | one or more pair/share ids | `--memory-format full\|outline\|blocks` |
| `heyditto list` | — | `--username <u>`, `--limit <n>`, `--offset <n>`, `--source <s>` |
| `heyditto update <id>` | memory id plus content or edits | `--content`, `--content-file`, `--edits-json`, `--edits-file`, `--base-revision`, `--title` |
| `heyditto publish <id>` | memory id | `--title <t>`, `--privacy-mode <mode>` |
| `heyditto unpublish` | one id | `--memory-id <id>`, `--share-id <id>` |
| `heyditto subjects <q>` | query | `--top-k <n>` (default 10, max 100) |
| `heyditto memories <id>...` | one or more subject ids | `--query <q>` |
| `heyditto network <id>` | pair id | `--limit <n>` (default 20, max 50) |

All commands accept `--output text|markdown|json|raw`. **Always pass `--output json`** from Hermes — the agent gets structured data instead of human-formatted text.

## When something fails

- **`heyditto: command not found` or `Unknown option '--output'`** → `@heyditto/cli` is missing or older than 1.1.3. `npm install -g @heyditto/cli@latest`.
- **`Unknown option '--memory-format'` or `Unknown command: update/publish`** → `@heyditto/cli` is older than 1.2.0. `npm install -g @heyditto/cli@latest`.
- **`ditto: unrecognized option '--output'` or Apple's `Usage:` line** → only seen if you typed `ditto` instead of `heyditto` and hit `/usr/bin/ditto` on macOS. Switch back to `heyditto` (no collision); see [`setup.md`](setup.md) step 1 to also fix PATH.
- **`error: no Ditto API key configured`** → see Pattern 5.
- **Connection failed** → run `heyditto status --output json`; rotate via `heyditto logout && heyditto login <new-key>`.
- **Empty results** → no matching memories. Suggest `heyditto save` if the fact is worth keeping.
- **Anything else** → support@heyditto.ai
