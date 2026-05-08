# Worked examples

Concrete patterns the agent should follow when invoking Ditto from Hermes.

## Pattern 1 — recall: "what did I say about X"

```bash
# 1. broad semantic search
ditto search "X" --output json

# 2. fetch full text for the top hit(s)
ditto fetch <pairId from step 1> --output json
```

Summarize the fetched memory text for the user, then optionally save a follow-up if they react with a clarification or correction.

## Pattern 2 — opinion / preference

```bash
# 1. find subjects matching the topic
ditto subjects "X" --top-k 5 --output json

# 2. expand the strongest subject(s) into memories
ditto memories <subjectId from step 1> --output json

# 3. optionally fetch full text for the most relevant memory
ditto fetch <pairId> --output json
```

## Pattern 3 — explicit save

```bash
ditto save "<durable fact, 1-3 sentences>" \
  --source hermes \
  --source-context "<optional: project, file, channel>" \
  --output json
```

Confirm to the user: "Saved."

## Pattern 4 — graph traversal

```bash
# 1. find seed memory
ditto search "X" --output json

# 2. expand its network
ditto network <pairId from step 1> --limit 30 --output json
```

## Pattern 5 — first-run, no key configured

If `ditto status` reports `MISSING (source: none)` or any command exits with `error: no Ditto API key configured`:

1. Hermes should have prompted at install time, but if the prompt was skipped:
   ```bash
   hermes skills config ditto
   ```
   Re-runs the prompt for `DITTO_API_KEY`.
2. If the user prefers to paste directly, fall back to:
   ```bash
   ditto login <key>
   ```
   Writes `~/.config/heyditto/cli/config.json` (mode 0600) — persists across shells without `.zshrc` editing.
3. Confirm with `ditto status --output json` (should show `source: env` or `source: config`), then retry the original command.

## Common args reference

| Command | Required | Optional |
|---|---|---|
| `ditto save <content>` | content | `--source <s>`, `--source-context <c>` |
| `ditto search <q>...` | one or more queries | — |
| `ditto fetch <id>...` | one or more pair ids | — |
| `ditto subjects <q>` | query | `--top-k <n>` (default 10, max 100) |
| `ditto memories <id>...` | one or more subject ids | — |
| `ditto network <id>` | pair id | `--limit <n>` (default 20, max 50) |

All commands accept `--output text|markdown|json|raw`. **Always pass `--output json`** from Hermes — the agent gets structured data instead of human-formatted text.

## When something fails

- **`error: no Ditto API key configured`** → see Pattern 5.
- **Connection failed** → run `ditto status`; rotate via `ditto logout && ditto login <new-key>`.
- **Empty results** → no matching memories. Suggest `ditto save` if the fact is worth keeping.
- **Anything else** → support@heyditto.ai
