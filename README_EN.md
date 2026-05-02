# Hermes Multi-Bot Discord Rules

> One Skill. Three problems solved: bots talking over each other, style contamination, and dispatch chaos.

Turn your Discord channel into a well-run "AI company" — the boss speaks to everyone, but only the person called by name answers. Each bot has an independent persona. Styles never cross-contaminate.

## What Problem Does This Solve?

You put two (or more) Hermes bots in the same Discord channel. Then:

- ❌ Bot A answers even when not @mentioned
- ❌ Bot B unconsciously mimics Bot A's style
- ❌ You ask Bot A to dispatch Bot B, but Bot A does the job itself
- ❌ You say "shut up" and the bot replies "shutting up" (it just spoke again!)

**Root cause:** All bots share the same LLM context window. Every bot's messages enter every other bot's "field of view", causing unconscious style bleed and response chaos.

This Skill plugs every leak with six iron rules.

## Quick Start

### 1. Install the Skill

```bash
cd ~/.hermes/skills/
git clone https://github.com/chenguru/hermes-multi-bot-discord-rules.git multi-bot-discord-rules
```

Or download `SKILL.md` directly into `~/.hermes/skills/multi-bot-discord-rules/`.

### 2. Configure Each Bot

In each bot's `config.yaml`:

```yaml
require_mention: true      # Never answer without @mention

discord:
  auto_thread: false       # Disable auto thread creation
```

In the main bot's `.env`:

```bash
DISCORD_ALLOW_BOTS=mentions   # Allow @mentions from other bots
```

### 3. Write SOUL.md

For each bot, copy `templates/SOUL-template.md` to the **very top** of that bot's SOUL.md, and fill in your own role's style declaration.

### 4. Restart All Gateways

```bash
systemctl --user restart hermes-gateway-{bot-name}
```

### 5. Verify

Test in the channel:

```
@BotA hello              → Only BotA replies
@BotA tell BotB to do X  → BotA sends <@BotB_ID>, then shuts up
@BotB                    → BotB executes and replies
stop                     → Bot confirms then goes truly silent
```

## Core Rules at a Glance

| # | Rule | One-Liner |
|---|------|-----------|
| 1 | **Mention-Only** | Never speak unless @mentioned |
| 2 | **Style Isolation** | Never touch another bot's style |
| 3 | **One Q, One A** | Answer the question, then stop. No follow-ups. |
| 4 | **Dispatch Transparency** | Send the @mention, then say nothing more |
| 5 | **Report → Ack → Done** | "Got it" and stop. No analysis. |
| 6 | **Silence Means Silence** | After "stop", confirm with ≤10 chars, then zero output |

## Why Style Isolation Is Priority #1

In a multi-bot channel, the LLM context window fills with messages from other bots. If Bot A responds in classical poetic Chinese, Bot B's next response may unconsciously drift poetic.

This contamination is **cumulative and irreversible** — once it starts, persona collapse follows.

The fix: every bot's SOUL.md has a **hardcoded** style declaration at the very top, plus the iron rule "never mimic anyone else." This is the firewall for role integrity.

## Repo Structure

```
├── SKILL.md                 # Core Skill (for bots to read)
├── README.md                # Chinese README (中文说明)
├── README_EN.md             # This file (English README)
├── templates/
│   ├── SOUL-template.md     # Style isolation declaration template
│   └── config-snippets.yaml # Key config snippets
└── LICENSE                  # MIT
```

## Use Cases

- ✅ 2+ Hermes bots in the same Discord channel
- ✅ Bots need to dispatch each other (via @mention)
- ✅ Each bot has a distinct persona and style
- ✅ One coordinator bot dispatches, others execute

Not for:
- ❌ Single bot (no multi-bot rules needed)
- ❌ Telegram (Telegram bots can't see or @mention each other)

## FAQ

**Q: New bot still answers without @mention?**

Check `require_mention: true`. Must restart Gateway after config changes.

**Q: @mention sent but target bot doesn't respond?**

Discord @mention format must be `<@NumericID>`, not `@BotName`. Enable Developer Mode in Discord, right-click the bot's avatar to copy ID.

**Q: Two bots respond to the same trigger?**

Check if `mention_patterns` overlap between bots. Patterns must be mutually exclusive.

**Q: Config changes don't take effect?**

After modifying config.yaml, SOUL.md, or .env, you **must restart** that bot's Gateway.

## License

MIT — free to use, modify, and distribute.

## Contributing

Issues and PRs welcome. If you've used these rules to run your own multi-bot deployment, share your experience!
