# Setup

After `hermes skills install ditto-assistant/ditto-hermes/ditto`:

## 1. Install the Ditto CLI

```bash
npm install -g @heyditto/cli@latest
heyditto --version    # 1.2.0 or newer
```

Hermes installs Node.js as part of its own installer, so npm should already be on PATH. If `npm` isn't found, install Node 20+ first.

**Why `heyditto` and not `ditto`:** the npm package ships two interchangeable binaries — `heyditto` and `ditto`. This skill standardizes on **`heyditto`** because on macOS Apple ships `/usr/bin/ditto` (a file-copy utility) which can shadow the npm CLI on default PATHs. `heyditto` has no such collision and works identically on every platform.

**Why 1.2.0+:** this skill uses current MCP fetch formats and memory update/publish commands. Earlier 1.1.x builds still support `--output` and the `heyditto` alias but do not expose the newer memory-management surface.

> If `heyditto --version` reports `heyditto: command not found`, the CLI is either missing or older than 1.1.3 — run the install line again. If you still want to use `ditto` (e.g. on Linux or once your PATH is sorted), `ditto` is the same binary.

### Optional: also wire up `ditto` cleanly on macOS

If you'd rather type `ditto` and have it Just Work, check what your shell resolves first:

```bash
type -a ditto
# /usr/bin/ditto             ← Apple's tool (file-copy utility)
# /opt/homebrew/bin/ditto    ← @heyditto/cli (you want this one to win)
```

If Apple's binary appears first, reorder `PATH` so the npm global bin precedes `/usr/bin` (e.g. in `~/.zshrc`), or call `/opt/homebrew/bin/ditto …` explicitly. None of this is necessary if you stick to `heyditto`.

## 2. API key

Hermes will prompt automatically (declared via `required_environment_variables` in the skill frontmatter):

```
Ditto API key (paste from https://app.heyditto.ai/connect/hermes): _
```

Open **https://app.heyditto.ai/connect/hermes**, sign in (GitHub / Google / email), click **New key**, copy, paste. Hermes stores it and passes it through to skill commands automatically — no shell-rc editing.

If you missed the prompt or want to re-enter the key:

```bash
hermes skills config ditto
```

Or use the CLI's own login flow (writes `~/.config/heyditto/cli/config.json`, mode 0600):

```bash
heyditto login <paste-key>
```

`DITTO_API_KEY` env wins over the config file — useful for one-off overrides, otherwise the file persists across shells.

## 3. Smoke test

```bash
heyditto status --output json
```

Expected (pretty-printed JSON):

```json
{
  "package": "@heyditto/cli",
  "version": "1.2.0",
  "endpoint": "https://api.heyditto.ai/mcp",
  "apiKey": { "present": true, "source": "env" },
  "tools": [
    "fetch_memories",
    "get_memory_network",
    "list_memories",
    "list_my_memories",
    "publish_memory",
    "save_memory",
    "search_memories",
    "search_memories_in_subjects",
    "search_subjects",
    "unpublish_memory",
    "update_memory"
  ],
  "connect": { "ok": true }
}
```

Then exercise a data command:

```bash
heyditto subjects "test" --top-k 1 --output json
```

Should return JSON like `{"results":[…],"metadata":{…}}` from the user's account.

> If you see `heyditto: command not found`, `Unknown option '--memory-format'`, `Unknown command: update`, or `Unknown command: publish`, the npm CLI is missing or older than the current skill expects — run `npm install -g @heyditto/cli@latest`.
>
> If you're typing `ditto` instead of `heyditto` and see `ditto: unrecognized option '--output'` or Apple's `Usage: ditto [ <options> ] src …`, you're hitting `/usr/bin/ditto` — switch to `heyditto`, or fix PATH (see step 1).

## You're done

Hermes will now use Ditto memory automatically when the conversation calls for it. See [`examples.md`](examples.md) for agent patterns, or run `heyditto help` for the full CLI reference.

## Troubleshooting

| Symptom | Fix |
|---|---|
| `heyditto: command not found` | `npm install -g @heyditto/cli@latest`; reopen shell to pick up the new bin. (`heyditto` ships in 1.1.3+.) |
| `Unknown option '--memory-format'` or `Unknown command: update/publish` | Installed `@heyditto/cli` is older than 1.2.0. `npm install -g @heyditto/cli@latest`. |
| `Unknown option '--output'` from the npm CLI | Installed `@heyditto/cli` is older than 1.1.3. `npm install -g @heyditto/cli@latest`. |
| `ditto: unrecognized option '--output'` *or* `Usage: ditto [ <options> ] src [ ... src ] dst` | You typed `ditto` and hit Apple's `/usr/bin/ditto`. Use `heyditto` (no collision) or fix PATH — see step 1. |
| `error: no Ditto API key configured` | `hermes skills config ditto` (re-prompts) or `heyditto login <key>`. |
| `heyditto status` shows `source: env` but you wanted `config` | The env var overrides. `unset DITTO_API_KEY` (and remove from `~/.zshrc` etc.) or rely on Hermes' env-var injection. |
| Connection failures | `heyditto status --output json` to verify endpoint and tool list; rotate key with `heyditto logout && heyditto login <new>`. |
| Anything else | support@heyditto.ai |

## Where to get help

- **Skill issues:** https://github.com/ditto-assistant/ditto-hermes/issues
- **CLI issues:** https://github.com/ditto-assistant/ditto-cli/issues
- **Account / backend:** **support@heyditto.ai**
- **CLI on npm:** https://www.npmjs.com/package/@heyditto/cli
