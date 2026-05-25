---
name: ditto
description: Save, search, fetch, and traverse the user's Ditto memory graph (https://heyditto.ai). Use whenever the user references "remember", "recall", "what did I", "from my notes", or asks about past conversations and saved knowledge.
version: 1.0.0
author: ditto-assistant
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [Memory, Knowledge, Search, MCP, Personal]
    category: productivity
    homepage: https://heyditto.ai
prerequisites:
  commands: [heyditto]
required_environment_variables:
  - name: DITTO_API_KEY
    prompt: Ditto API key (paste from https://app.heyditto.ai/connect/hermes)
    help: Open https://app.heyditto.ai/connect/hermes — one-page sign-in (GitHub / Google / email), click "New key", and copy. The key starts with "ditto_mcp_".
    required_for: all skill functionality
---

# Ditto

Use the `heyditto` CLI to save, search, fetch, update, publish, and traverse the user's long-term memory graph at [heyditto.ai](https://heyditto.ai). Memories are conversation **pairs** identified by a private `pair_id`; public DittoHub shares use a `share_id`. **Subjects** are graph nodes for topics, identified by `subject_id`.

> The package `@heyditto/cli` ships **two interchangeable binaries**: `heyditto` and `ditto`. This skill standardizes on **`heyditto`** in every example because on macOS Apple ships its own `/usr/bin/ditto` (a file-copy utility) that can shadow the npm CLI; `heyditto` has no such collision. If you're on a system where plain `ditto` resolves to the npm bin, the two names are identical.

## When to Use

Reach for Ditto memory whenever the user:

- Says "remember…", "save this", "note that…", "for later".
- Says "what did I…", "recall…", "have I told you about…", "from my notes".
- Asks a question best answered from their prior context, not general knowledge.
- References a topic, person, project, or thread that isn't in this conversation but might be in their memory.

## Prerequisites

The `heyditto` CLI ([`@heyditto/cli`](https://www.npmjs.com/package/@heyditto/cli)) must be on PATH at version **1.2.0 or newer** (the version that includes current MCP fetch formats and memory update/publish commands). Hermes installs Node.js as part of its install, so:

```bash
npm install -g @heyditto/cli@latest
heyditto --version    # should print 1.2.0 or newer
```

`DITTO_API_KEY` is prompted by Hermes at skill-install time (declared above). Hermes passes it through to all `terminal` and `execute_code` calls automatically — no manual `export` needed.

## Procedure

Always call the CLI with `--output json` so output is structured and easy to summarize back to the user.

```bash
heyditto status --output json    # confirm key + endpoint + tools resolve
```

### Recall — "what did I say about X"

Search for previews, then fetch full text for the top hit(s).

```bash
heyditto search "X" --output json
heyditto fetch <pairId from step 1> --output json
```

Summarize the fetched memory text for the user.

### Opinion / preference — "what do I think about X"

Hit the subject graph first, then drill into matching memories.

```bash
heyditto subjects "X" --top-k 5 --output json
heyditto memories <subjectId> --output json
heyditto fetch <pairId> --output json   # optional, for full text
```

### Save — "remember this" / spotted durable fact

```bash
heyditto save "<the durable fact, 1–3 sentences>" \
  --source hermes \
  --source-context "<optional: project, file, channel>" \
  --output json
```

Confirm to the user: "Saved."

### Update — correct a saved memory

Use an outline fetch first when doing a targeted patch, then pass block edits
with the current revision from the prior `save` or `update` response. Use
full-content replacement when the current revision is unknown or the user asks
to rewrite the whole memory.

```bash
heyditto fetch <pairId> --memory-format outline --output json
heyditto update <pairId> \
  --edits-json '[{"op":"replace_text","blockId":"2","find":"old","replace":"new","expectedCount":1}]' \
  --base-revision <revision from save/update> \
  --output json
```

### Publish / unpublish

Only publish after an explicit user request.

```bash
heyditto publish <pairId> --title "<optional title>" --privacy-mode scan_and_block --output json
heyditto unpublish --share-id <shareId> --output json
```

### Graph traversal — "show me everything related to X"

```bash
heyditto search "X" --output json                              # find seed memory
heyditto network <pairId from step 1> --limit 30 --output json # related via shared subjects
```

## Tools (CLI surface)

| Command | Purpose |
|---|---|
| `heyditto save <content> [--source <s>] [--source-context <c>]` | Persist a memory pair from an external source. |
| `heyditto search <query>... [--include-public] [--filter-username <u>]` | Semantic search across private memories and optionally public DittoHub memories. |
| `heyditto fetch <id>... [--memory-format full\|outline\|blocks]` | Fetch full content or block structures by private pair id or public share id. |
| `heyditto list [--username <u>] [--limit <n>] [--offset <n>] [--source <s>]` | List saved memories or a public DittoHub catalog. |
| `heyditto update <id> [--content ...\|--content-file ...\|--edits-json ...]` | Edit a saved memory in place. Use `--base-revision` with block edits. |
| `heyditto publish <id> [--title <t>] [--privacy-mode <mode>]` | Publish a saved memory to DittoHub after explicit user consent. |
| `heyditto unpublish (--memory-id <id>\|--share-id <id>\|<id>)` | Disable a public share without deleting the private memory. |
| `heyditto subjects <query> [--top-k <n>]` | Search the subject graph (default 10, max 100). |
| `heyditto memories <subject-id>... [--query <q>]` | Fetch or semantically filter memory previews scoped to specific subjects. |
| `heyditto network <pair-id> [--limit <n>]` | Traverse memories connected via shared subjects (default 20, max 50). |
| `heyditto status` | Endpoint + key source + live tool list. |
| `heyditto config` | Print Claude Desktop / Cursor / generic-MCP-client config snippet. |
| `heyditto help` | Full reference. |

`heyditto status` prints the live tool list straight from the MCP — trust it over this file if anything drifts. `ditto <subcommand>` works identically when the npm bin wins on PATH.

## Pitfalls

- **`heyditto: command not found`** → `@heyditto/cli` isn't installed or is older than 1.1.3 (which introduced the `heyditto` alias). Run `npm install -g @heyditto/cli@latest` and reopen the shell.
- **`Unknown option '--memory-format'`, `Unknown command: update`, or `Unknown command: publish`** → installed `@heyditto/cli` is older than 1.2.0. Same fix: `npm install -g @heyditto/cli@latest`.
- **`Unknown option '--output'`** → installed `@heyditto/cli` is older than 1.1.3. Same fix: `npm install -g @heyditto/cli@latest`.
- **`ditto: unrecognized option '--output'` or `Usage: ditto [ <options> ] src [ ... src ] dst`** (only seen if a user typed `ditto` instead of `heyditto` on macOS) → that's Apple's `/usr/bin/ditto` shadowing the npm CLI. Stick to `heyditto` (no collision) or run `type -a ditto` and reorder PATH.
- **`error: no Ditto API key configured`** → `DITTO_API_KEY` didn't make it into the shell. Re-prompt the user via `hermes skills config ditto`, or fall back to `heyditto login <key>` to write `~/.config/heyditto/cli/config.json` directly.
- **Connection failures** → run `heyditto status --output json`; rotate via `heyditto logout && heyditto login <new-key>`.
- **Empty results** → user may not have memories matching the query. Suggest they save the fact with `heyditto save` if it's worth keeping.
- **Schema drift** → run `heyditto status --output json` and `heyditto help` for the current surface.

## Verification

```bash
heyditto status --output json | jq '.tools'                  # array of MCP tool names
heyditto subjects "test" --output json | jq '.results | length'
```

Both should succeed with a valid key.

## Source + Support

- **CLI on npm:** https://www.npmjs.com/package/@heyditto/cli
- **CLI source:** https://github.com/ditto-assistant/ditto-cli
- **Skill repo:** https://github.com/ditto-assistant/ditto-hermes
- **Get a key:** https://app.heyditto.ai/connect/hermes
- **Account / backend support:** support@heyditto.ai
