# ditto-hermes

The Ditto Memory skill for [Hermes Agent](https://hermes-agent.nousresearch.com) (Nous Research).

This repo is a **Hermes custom tap** — users add it once, then install the skill from it:

```bash
hermes skills tap add ditto-assistant/ditto-hermes
hermes skills install ditto-assistant/ditto-hermes/ditto
```

> **Sibling repos:** the `ditto` CLI binary lives in [**`ditto-cli`**](https://github.com/ditto-assistant/ditto-cli) (published as [**`@heyditto/cli`**](https://www.npmjs.com/package/@heyditto/cli) on npm). This repo is just the Hermes-format SKILL bundle that teaches the Hermes agent how and when to invoke it.

## What's in the bundle

```
ditto/
├── SKILL.md                   # Hermes-format frontmatter + agent decision guide
└── references/
    ├── setup.md               # 3-step user setup (auto-prompt for API key, smoke test)
    └── examples.md            # agent patterns (recall, opinion, save, traverse, first-run-no-key)
```

`SKILL.md` declares `required_environment_variables: [{ name: DITTO_API_KEY, prompt: ..., help: ... }]` for existing human keys, but the default agent path is `heyditto init --agent --json`. That creates a temporary claimable account without email, OTP, dashboard setup, or browser login; the agent shares only the short claim link with the user. The claim token lives in the link's `#t=...` fragment.

## Architecture

```
Hermes user                                                    Hermes agent
  │                                                                  │
  ├─ hermes skills install ditto-assistant/ditto-hermes/ditto        │
  │     extracts SKILL.md into ~/.hermes/skills/.../ditto/           │
  │                                                                  │
  ├─ user runs `npm install -g @heyditto/cli`                        │
  │     (Hermes installer ships Node 22; npm available)              │
  │                                                                  │
  ├─ agent runs `heyditto init --agent --agent-caller hermes --json` │
  │     creates a free claimable account and stores the API key      │
  │                                                                  │
  ├─ "what did I say about X?"                                       │
  │                                          ┌──────────────────┐    │
  │                                          │ heyditto binary  │ ◀──┤ heyditto save / search /
  │                                          │   on PATH        │    │ fetch / subjects /
  │                                          └────────┬─────────┘    │ memories / network
  │                                                   │              │
  │                                                   ▼              │
  │                                       Authorization: Bearer …    │
  │                                       https://api.heyditto.ai/mcp ◀──┘
```

Auth is API key. Agents can self-provision with `heyditto init --agent --json`; `DITTO_API_KEY` env (set by Hermes) still takes priority, and `heyditto login <key>` storing `~/.config/heyditto/cli/config.json` remains the fallback for existing keys.

## How this differs from `ditto-clawhub`

| | ditto-hermes | [ditto-clawhub](https://github.com/ditto-assistant/ditto-clawhub) |
|---|---|---|
| Target agent | Hermes (Nous Research) | OpenClaw / ClawHub (steipete) |
| SKILL frontmatter shape | `required_environment_variables`, `prerequisites.commands`, `metadata.hermes.*` | inline-JSON `metadata.openclaw.*` with `install` spec |
| Distribution | GitHub tap (this repo) → `hermes skills tap add` | drag `publish/` into clawhub.ai/publish |
| CLI auto-install | No — user runs `npm i -g @heyditto/cli` | Yes — one-click "Install Ditto CLI (npm)" button via openclaw's `install` spec |
| Key onboarding URL | https://app.heyditto.ai/connect/hermes | https://app.heyditto.ai/connect/openclaw |
| Auth flow | Agent runs `heyditto init --agent --json`; optional env-injected key override | Agent runs `ditto init --agent --json`; optional `ditto login <key>` fallback |

Same CLI binary (`@heyditto/cli`), same backend MCP, two skill bundles tailored to each agent's idioms.

## Status

- [x] `@heyditto/cli` published on npm (1.2.x current skill target, Trusted Publisher via [`ditto-cli`](https://github.com/ditto-assistant/ditto-cli))
- [x] Skill bundle finalized — `ditto/SKILL.md` matches the canonical [Hermes mcporter skill](https://github.com/NousResearch/hermes-agent/blob/main/optional-skills/mcp/mcporter/SKILL.md) shape
- [ ] `https://app.heyditto.ai/connect/hermes` live (frontend)
- [ ] Repo flipped public so `hermes skills tap add` works
- [ ] (Optional) PR upstream into `NousResearch/hermes-agent/optional-skills/productivity/ditto/` for first-class listing
- [ ] (Optional) Publish to [agentskills.io](https://agentskills.io) for cross-agent discovery

## Distribution paths

Three options, pick one (or all):

1. **Custom tap (this repo)** — works today once the repo is public:
   ```bash
   hermes skills tap add ditto-assistant/ditto-hermes
   hermes skills install ditto-assistant/ditto-hermes/ditto
   ```
2. **PR into hermes-agent** — submit `optional-skills/productivity/ditto/` to [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) for inclusion in the official optional skills.
3. **agentskills.io** — publish to the cross-agent registry for discovery from Hermes, ClawHub, and beyond.

## Related

- [`ditto-cli`](https://github.com/ditto-assistant/ditto-cli) — `@heyditto/cli` source. Where `heyditto save`, `heyditto login`, etc. are implemented (also exposed under the `ditto` alias bin).
- [`ditto-mcp`](https://github.com/ditto-assistant/ditto-mcp) — sibling stdio MCP bridge (`@heyditto/mcp`) with browser-OAuth, for Claude Desktop / Cursor.
- [`ditto-clawhub`](https://github.com/ditto-assistant/ditto-clawhub) — sibling skill bundle for OpenClaw / ClawHub.
- [Hermes Agent](https://hermes-agent.nousresearch.com) ([repo](https://github.com/NousResearch/hermes-agent)) — the agent.
- [Hermes skill format docs](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills)
- [Reference skill we modeled after](https://github.com/NousResearch/hermes-agent/blob/main/optional-skills/mcp/mcporter/SKILL.md) — Hermes' own `mcporter` skill.
