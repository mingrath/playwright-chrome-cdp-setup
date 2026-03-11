# Playwright MCP + Chrome CDP Setup

Control your real Chrome browser through Claude Code using Playwright MCP over Chrome DevTools Protocol (CDP).

This lets Claude browse the web **as you** — logged into your Google account, Facebook, etc. — without needing to log in every session.

## How It Works

```
You (Claude Code) → Playwright MCP Server → CDP (port 9222) → Your Chrome Browser
```

- **Playwright MCP** is the tool Claude uses to control browsers
- **CDP (Chrome DevTools Protocol)** connects Playwright to your real Chrome
- **Persistent profile folders** keep your logins saved between sessions

## Prerequisites

- [Claude Code CLI](https://github.com/anthropics/claude-code) installed
- Google Chrome installed
- Playwright MCP server configured in Claude Code

## Setup

### Step 1: Install Playwright MCP

In Claude Code settings (`~/.claude/settings.json`), make sure you have the Playwright MCP server configured:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@anthropic-ai/mcp-playwright", "--cdp-endpoint", "http://localhost:9222"]
    }
  }
}
```

> **Note:** Check the actual package name of your Playwright MCP server — it may differ. The key part is `--cdp-endpoint http://localhost:9222`.

### Step 2: Create Profile Folders

Each Google account gets its own Chrome profile folder:

```bash
mkdir -p ~/.chrome-playwright-profiles/mingrath
mkdir -p ~/.chrome-playwright-profiles/chanika
# Add more as needed:
# mkdir -p ~/.chrome-playwright-profiles/another-account
```

### Step 3: Log In to Each Profile (One-Time)

Launch Chrome with each profile and log into the Google account:

```bash
# Mingrath account
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
  --remote-debugging-port=9222 \
  --user-data-dir="$HOME/.chrome-playwright-profiles/mingrath" \
  --no-first-run --no-default-browser-check
```

1. Log into the Google account in the Chrome window
2. Close Chrome
3. Repeat for other profiles:

```bash
# Chanika account
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
  --remote-debugging-port=9222 \
  --user-data-dir="$HOME/.chrome-playwright-profiles/chanika" \
  --no-first-run --no-default-browser-check
```

> **Important:** Only run one profile at a time on port 9222. Close Chrome before switching profiles.

### Linux

Replace the Chrome path:

```bash
google-chrome \
  --remote-debugging-port=9222 \
  --user-data-dir="$HOME/.chrome-playwright-profiles/mingrath" \
  --no-first-run --no-default-browser-check
```

### Windows

```powershell
"C:\Program Files\Google\Chrome\Application\chrome.exe" `
  --remote-debugging-port=9222 `
  --user-data-dir="%USERPROFILE%\.chrome-playwright-profiles\mingrath" `
  --no-first-run --no-default-browser-check
```

## Usage with Claude Code

Once set up, just tell Claude:

- **"Open Chrome with mingrath"** — launches Mingrath's profile
- **"Open Chrome with chanika"** — launches Chanika's profile
- **"Go to gmail.com"** — navigates in the open browser
- **"Click on the first email"** — interacts with the page

Claude will:
1. Kill any existing debug Chrome (`pkill -f chrome-playwright-profiles`)
2. Launch Chrome with the requested profile on port 9222
3. Connect via Playwright MCP over CDP
4. Control the browser as instructed

## Claude Code Memory (Optional)

Add this to your Claude memory (`~/.claude/MEMORY.md` or project memory) so Claude always knows:

```markdown
## Chrome Profiles for Playwright MCP (CDP)
- **mingrath** = Mingrath's Google account
- **chanika** = Chanika's Google account
- Profiles stored in `~/.chrome-playwright-profiles/<name>/`
- Launch: `/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222 --user-data-dir="$HOME/.chrome-playwright-profiles/<name>" --no-first-run --no-default-browser-check`
- Always kill existing debug Chrome before launching a new profile
- Only one profile at a time on port 9222
```

## Troubleshooting

### "connect ECONNREFUSED ::1:9222"
Chrome isn't running with CDP. Launch it with `--remote-debugging-port=9222`.

### "Opening in existing browser session"
Your main Chrome is already running and intercepting the launch. Either:
- Quit all Chrome windows first, OR
- Use `--user-data-dir` to force a separate instance (this guide already does this)

### "DevTools remote debugging requires a non-default data directory"
You forgot `--user-data-dir`. Always include it — this guide's commands already do.

### Login not persisted
Make sure you're using the same `--user-data-dir` path every time. The login is stored in that folder.

## File Structure

```
~/.chrome-playwright-profiles/
├── mingrath/     # Mingrath's Google account profile
└── chanika/      # Chanika's Google account profile
```

Each folder contains Chrome's user data (cookies, local storage, saved logins). **Do not delete these folders** or you'll need to log in again.
