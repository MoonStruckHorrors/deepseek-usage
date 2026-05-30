---
name: deepseek-usage
description: This skill should be used when the user asks to "check deepseek balance", "deepseek usage", "how much deepseek credit", "deepseek API balance", "check my deepseek usage", "deepseek billing", "deepseek credits remaining", or any question about their DeepSeek account balance or usage.
user-invocable: true
allowed-tools:
  - Read
  - Bash
  - Grep
  - Glob
argument-hint: <no args — just run it>
---

# /deepseek-usage — Check DeepSeek API Balance

Finds the DeepSeek API key from wherever it is stored, calls the balance
endpoint, and displays the result. Works on Windows, Linux, and macOS.

Invoke directly with `/deepseek-usage` or trigger with natural language like
"check my deepseek balance."

## Step 1 — Find the API key

Search in priority order. Stop at the first valid key found (starts with
`sk-`).

### 1a. Check the current environment

```bash
echo "${ANTHROPIC_AUTH_TOKEN:-}"
```

If it returns a value starting with `sk-`, use it. Skip to Step 2.

### 1b. Determine the OS and search profile files

Run `uname -s` to detect the platform, then search in this order:

**Windows** (`MINGW*` or `MSYS*`):

1. PowerShell profile:
   ```bash
   grep -oPh 'ANTHROPIC_AUTH_TOKEN\s*[="\s]+\Ksk-[^"'\'']+' "$(powershell.exe -NoProfile -Command '[System.Environment]::GetFolderPath([System.Environment+SpecialFolder]::Personal)')/PowerShell/Microsoft.PowerShell_profile.ps1" 2>/dev/null || echo ""
   ```
   Also check under `Documents/WindowsPowerShell/` and `Documents/PowerShell/`.

2. `~/.bashrc`:
   ```bash
   grep -oPh 'ANTHROPIC_AUTH_TOKEN[="\s]+sk-[^"\s]+' ~/.bashrc 2>/dev/null || echo ""
   ```

3. Project `.env` files:
   ```bash
   grep -oPh 'ANTHROPIC_AUTH_TOKEN[="\s]+sk-[^"\s]+' .env 2>/dev/null || echo ""
   ```

**Linux** (`Linux`):

1. `~/.bashrc`, then `~/.zshrc`, then `~/.profile`:
   ```bash
   for f in ~/.bashrc ~/.zshrc ~/.profile; do
     key=$(grep -oPh 'ANTHROPIC_AUTH_TOKEN[="\s]+sk-[^"\s]+' "$f" 2>/dev/null | head -1)
     [ -n "$key" ] && echo "$key" && break
   done
   ```

2. Project `.env` files.

**macOS** (`Darwin`):

1. `~/.zshrc`, then `~/.bash_profile`:
   ```bash
   for f in ~/.zshrc ~/.bash_profile; do
     key=$(grep -oPh 'ANTHROPIC_AUTH_TOKEN[="\s]+sk-[^"\s]+' "$f" 2>/dev/null | head -1)
     [ -n "$key" ] && echo "$key" && break
   done
   ```

2. Project `.env` files.

### 1c. Extract the key

From whatever source found, extract only the `sk-...` token value. If the
grep captured a prefix like `ANTHROPIC_AUTH_TOKEN=` or
`$env:ANTHROPIC_AUTH_TOKEN="`, strip it:

```bash
DEEPSEEK_KEY=$(echo "$raw_match" | grep -oP 'sk-[a-zA-Z0-9]+')
```

### 1d. Not found?

If no key is found anywhere, tell the user:

> No DeepSeek API key found. Get one from https://platform.deepseek.com/api_keys
> and set it as the `ANTHROPIC_AUTH_TOKEN` environment variable.
>
> **Windows (PowerShell):**
> ```powershell
> $env:ANTHROPIC_AUTH_TOKEN="sk-your-key-here"
> ```
>
> **Linux / macOS:**
> ```bash
> export ANTHROPIC_AUTH_TOKEN="sk-your-key-here"
> ```

Then stop.

## Step 2 — Call the DeepSeek API

```bash
curl -s -w "\n%{http_code}" \
  -H "Authorization: Bearer ${DEEPSEEK_KEY}" \
  "https://api.deepseek.com/user/balance"
```

Separate the body (all lines except the last) from the HTTP status code (last
line).

## Step 3 — Parse and display the result

If HTTP status is `200`, parse the JSON body. Try `jq` first, then fall back
to raw output:

### With jq (preferred)

```bash
echo "$BODY" | jq -r '
  "Available:   \(.is_available)",
  "Balance:     \(.balance_infos[0].total_balance // "N/A") \(.balance_infos[0].currency // "")",
  "  Paid:      \(.balance_infos[0].topped_up_balance // "N/A")",
  "  Granted:   \(.balance_infos[0].granted_balance // "N/A")"
'
```

### Without jq (fallback)

Display the raw JSON with a note that installing `jq` gives cleaner output.
Parse manually with grep:

```bash
echo "$BODY" | grep -oP '"total_balance":"[^"]*"'
echo "$BODY" | grep -oP '"granted_balance":"[^"]*"'
echo "$BODY" | grep -oP '"topped_up_balance":"[^"]*"'
echo "$BODY" | grep -oP '"is_available":[^,}]*'
```

Format as a clean table for the user.

## Step 4 — Error handling

| HTTP code | Meaning | Action |
|-----------|---------|--------|
| `200` | Success | Display result |
| `401` | Invalid or expired key | Tell user to regenerate at https://platform.deepseek.com/api_keys |
| `403` | Key lacks permission | Ensure the key has access to the balance endpoint |
| `429` | Rate limited | Suggest waiting 30 seconds before retrying |
| `000` | Network error | Check internet connection and that `api.deepseek.com` is reachable |
| Other 5xx | Server error | DeepSeek API may be down; suggest retrying later |

## Tips

- The skill searches multiple locations — it finds the key even when
  `ANTHROPIC_AUTH_TOKEN` is not in the current shell session's env.
- If `jq` is not installed, the skill still works with grep-based fallback.
  Mention that `jq` can be installed for nicer output.
- If the key appears in multiple locations, use the first one found and note
  where it was sourced from.
