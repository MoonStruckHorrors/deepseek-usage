# deepseek-usage

Check your [DeepSeek](https://deepseek.com) API balance and usage directly from
Claude Code — no browser, no curl memorization. Finds your API key
automatically.

## Install

**Step 1** — Add the marketplace:
```
/plugin marketplace add MoonStruckHorrors/deepseek-usage
```

**Step 2** — Install the plugin:
```
/plugin install deepseek-usage@deepseek-usage
```

## Usage

### Slash command

```
/deepseek-usage
```

### Natural language

Just ask:
- "check my deepseek balance"
- "how much deepseek credit do I have?"
- "deepseek billing"

### Expected output

```
Available:   true
Balance:     9.75 USD
  Paid:      9.75
  Granted:   0.00
```

## Prerequisites

A DeepSeek API key set as `ANTHROPIC_AUTH_TOKEN`. The skill auto-discovers it
from:

| OS | Locations checked |
|----|-------------------|
| Windows | Current env, PowerShell `$PROFILE`, `~/.bashrc`, project `.env` |
| Linux | Current env, `~/.bashrc`, `~/.zshrc`, `~/.profile`, project `.env` |
| macOS | Current env, `~/.zshrc`, `~/.bash_profile`, project `.env` |

Get a key at [platform.deepseek.com/api_keys](https://platform.deepseek.com/api_keys).

## License

MIT
