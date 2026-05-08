# Setup

After `hermes skills install ditto-assistant/ditto-hermes/ditto`:

## 1. Install the Ditto CLI

```bash
npm install -g @heyditto/cli
ditto --version
```

Hermes installs Node.js as part of its own installer, so npm should already be on PATH. If `npm` isn't found, install Node 20+ first.

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
ditto login <paste-key>
```

`DITTO_API_KEY` env wins over the config file — useful for one-off overrides, otherwise the file persists across shells.

## 3. Smoke test

```bash
ditto status
```

Expected output:

```
@heyditto/cli 1.x.x
endpoint:  https://api.heyditto.ai/mcp
api key:   set  (source: env)
tools:     fetch_memories, get_memory_network, save_memory,
           search_memories, search_memories_in_subjects, search_subjects
```

Then:

```bash
ditto subjects "test" --output json
```

Should return JSON results from the user's account.

## You're done

Hermes will now use Ditto memory automatically when the conversation calls for it. See [`examples.md`](examples.md) for agent patterns, or run `ditto help` for the full CLI reference.

## Troubleshooting

| Symptom | Fix |
|---|---|
| `ditto: command not found` | `npm install -g @heyditto/cli`; reopen shell to pick up the new bin. |
| `error: no Ditto API key configured` | `hermes skills config ditto` (re-prompts) or `ditto login <key>`. |
| `ditto status` shows `source: env` but you wanted `config` | The env var overrides. `unset DITTO_API_KEY` (and remove from `~/.zshrc` etc.) or rely on Hermes' env-var injection. |
| Connection failures | `ditto status` to verify endpoint; rotate key with `ditto logout && ditto login <new>`. |
| Anything else | support@heyditto.ai |

## Where to get help

- **Skill issues:** https://github.com/ditto-assistant/ditto-hermes/issues
- **CLI issues:** https://github.com/ditto-assistant/ditto-cli/issues
- **Account / backend:** **support@heyditto.ai**
- **CLI on npm:** https://www.npmjs.com/package/@heyditto/cli
