# browser-harness

## What it does

Drive a real Chrome browser to navigate, screenshot, click, type, and scrape. Useful when:

- Capturing reference screenshots from web apps (vendor UIs, trial environments, internal tools).
- Pulling state from web tools that have no usable API or where the API is gated.
- Filling/submitting forms on services without a programmatic surface.

Screenshots are saved to disk as PNG; the agent then `Read`s the file and the image lands directly in context.

Upstream: https://github.com/browser-use/browser-harness

## Setup state

There is no API token. The harness drives an **isolated Chrome profile** at `~/.browser-harness-chrome/` that's separate from the user's daily Chrome.

Login state for any service is persisted as cookies in that profile dir. The first time the agent points at a service that needs login, the user logs in **once** in the agent's Chrome window. Subsequent runs are already authenticated.

```
Wrapper command:  bh-agent              (~/.local/bin/bh-agent, on PATH)
Profile dir:      ~/.browser-harness-chrome/
Daemon socket:    /tmp/bu-agent.sock
Daemon log:       /tmp/bu-agent.log
```

### How the wrapper works

The `bh-agent` wrapper:

1. Checks `~/.browser-harness-chrome/DevToolsActivePort`. If missing or its port isn't accepting connections → launches Chrome with `--user-data-dir=~/.browser-harness-chrome --remote-debugging-port=0 --no-first-run --no-default-browser-check`.
2. Reads `port` and `path` from `DevToolsActivePort`, builds `ws://127.0.0.1:{port}{path}`.
3. Exports `BU_CDP_WS=<that URL>` and `BU_NAME=agent`, then `exec`s the underlying `browser-harness` Python harness.

`BU_CDP_WS` makes the harness skip its built-in profile-scanning (which only knows about the user's daily Chrome dirs). `BU_NAME=agent` keeps the daemon socket separate from the upstream `default`.

### Install

```bash
brew install uv
git clone https://github.com/browser-use/browser-harness ~/repos/integrations/browser-harness
cd ~/repos/integrations/browser-harness
uv sync && uv tool install -e .                 # installs `browser-harness` to ~/.local/bin
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc.local

# Write the wrapper at ~/.local/bin/bh-agent (full source below in "Wrapper source")
# Then: chmod +x ~/.local/bin/bh-agent
```

## Usage

ALWAYS use the `bh-agent` wrapper, NOT raw `browser-harness`. Raw `browser-harness` would try to attach to the user's daily Chrome.

### Example: capture a screenshot

```bash
bh-agent -c "
new_tab('https://example.com')
wait_for_load()
capture_screenshot('~/Desktop/example.png', full=True)
"
```

Then read the PNG with the `Read` tool — the image is included in the agent's context.

### Helpers

All helpers are pre-imported when running `bh-agent -c "..."`. Source lives in `~/repos/integrations/browser-harness/helpers.py` — editable, extend it when a helper is missing.

**Navigation**

| Function | Notes |
|---|---|
| `new_tab(url='about:blank')` | Open a NEW tab. **First navigation in a session must use this**, not `goto_url`. |
| `goto_url(url)` | Navigate the active tab. Clobbers existing page. |
| `wait_for_load(timeout=15)` | Block until `document.readyState == 'complete'`. Returns `True` on success. |

**Inspection**

| Function | Returns |
|---|---|
| `page_info()` | `{url, title, w, h, sx, sy, pw, ph}` (viewport + scroll + page size). If a native dialog is open, returns `{dialog: ...}` instead. |
| `js("expr")` | Run arbitrary JS, returns serialized result via `Runtime.evaluate`. |
| `list_tabs(include_chrome=False)` | List of `{targetId, title, url}`. |
| `current_tab()` | `{targetId, url, title}` of active tab. |
| `cdp(method, **params)` | Raw CDP call, e.g. `cdp("DOM.getDocument", depth=-1)`. |

**Interaction**

| Function | Notes |
|---|---|
| `click_at_xy(x, y, button="left", clicks=1)` | Synthesized mouse click at pixel coords (CSS pixels, not device pixels). |
| `type_text("hello")` | `Input.insertText` — fires on focused element. |
| `press_key("Enter", modifiers=0)` | Special keys: `Enter`, `Tab`, `Backspace`, `Escape`, `Delete`, `ArrowLeft/Right/Up/Down`, `Home`, `End`, `PageUp`, `PageDown`. Modifiers bitfield: 1=Alt, 2=Ctrl, 4=Cmd, 8=Shift. |
| `scroll(x, y, dy=-300, dx=0)` | Wheel event. `dy` negative = scroll down. |

**Visual**

| Function | Notes |
|---|---|
| `capture_screenshot(path="/tmp/shot.png", full=False)` | PNG to disk. `full=True` captures beyond viewport (whole scrollable page). |

**Tabs**

| Function | Notes |
|---|---|
| `switch_tab(target)` | Accepts a `targetId` string or a dict from `current_tab()` / `list_tabs()`. |
| `ensure_real_tab()` | Switch to a non-`chrome://` tab if current is internal. |
| `iframe_target(url_substr)` | First iframe target whose URL contains the substring. Use with `js(..., target_id=...)`. |

### Common workflows

**Take reference screenshots from a logged-in service:**

1. Have agent run `bh-agent -c "new_tab('https://login-page')"` — Chrome window opens, you see the login page.
2. **You manually log in** in that Chrome window (one-time).
3. Agent then runs `bh-agent -c "goto_url('https://...'); wait_for_load(); capture_screenshot('/path.png', full=True)"` to capture pages.

**Reset the agent's browser state:**

```bash
pkill -f "user-data-dir=$HOME/.browser-harness-chrome"
rm -rf ~/.browser-harness-chrome/
```

Next `bh-agent` call launches a fresh Chrome.

## Gotchas

- **First call after a reboot opens a Chrome window.** That's the agent's isolated Chrome — leave it running. It's separate from the user's daily Chrome.
- **Login per service is a one-time manual step.** After login in the agent's Chrome window, cookies persist in `~/.browser-harness-chrome/`.
- **First navigation must be `new_tab(url)`**, not `goto_url(url)`. `goto_url` runs in the active tab, which on a fresh launch is `about:blank` but on subsequent runs may be a tab the agent is mid-task on.
- **Screenshots are viewport-only by default.** Pass `full=True` for the entire scrollable page.
- **`screencapture` (macOS) requires screen-recording permission**, which terminals usually lack. Use `capture_screenshot` instead — captures from inside Chrome via CDP, no OS permission needed.
- **Chrome's UI-toggled "Allow remote debugging"** at `chrome://inspect/#remote-debugging` is NOT how this integration works. The wrapper uses an explicit `--remote-debugging-port` launch flag on the isolated profile, which is more reliable and creates the `DevToolsActivePort` file the harness needs. Do not enable the chrome://inspect toggle on the user's daily Chrome — it's intrusive and not required.

## Troubleshooting

| Symptom | Cause / Fix |
|---|---|
| `bh-agent: Chrome failed to expose CDP after 10s` | Stale `DevToolsActivePort` from a crashed Chrome. `rm ~/.browser-harness-chrome/DevToolsActivePort` and retry. |
| `daemon agent didn't come up` | Daemon socket wedged. `rm -f /tmp/bu-agent.sock /tmp/bu-agent.pid` and retry. Check `/tmp/bu-agent.log`. |
| `no close frame received` mid-task | Daemon's CDP websocket went stale. `pkill -f bu-agent` then retry (next call respawns). |
| Want a fully fresh browser per task | `BH_AGENT_PROFILE=/tmp/bh-throwaway-$$ bh-agent -c "..."` |
| `command not found: bh-agent` | `~/.local/bin` not on PATH. Add `export PATH="$HOME/.local/bin:$PATH"` to `~/.zshrc.local` and `source` it. |
| `command not found: browser-harness` (from inside wrapper) | `uv tool install -e .` was not run, or `~/.local/bin` not on PATH. |

## Wrapper source (`~/.local/bin/bh-agent`)

```bash
#!/usr/bin/env bash
# Runs browser-harness against an isolated Chrome profile at
# ~/.browser-harness-chrome/, never touching the user's daily Chrome.

set -euo pipefail

PROFILE_DIR="${BH_AGENT_PROFILE:-$HOME/.browser-harness-chrome}"
PORT_FILE="$PROFILE_DIR/DevToolsActivePort"
CHROME_APP="${BH_AGENT_CHROME:-Google Chrome}"

launch_chrome() {
  mkdir -p "$PROFILE_DIR"
  open -na "$CHROME_APP" --args \
    --user-data-dir="$PROFILE_DIR" \
    --remote-debugging-port=0 \
    --no-first-run \
    --no-default-browser-check
}

port_is_live() {
  [[ -f "$PORT_FILE" ]] || return 1
  local port
  port=$(head -n1 "$PORT_FILE")
  [[ -n "$port" ]] || return 1
  nc -z 127.0.0.1 "$port" 2>/dev/null
}

if ! port_is_live; then
  echo "bh-agent: launching isolated Chrome (profile=$PROFILE_DIR)" >&2
  launch_chrome
  for i in {1..20}; do
    sleep 0.5
    port_is_live && break
  done
  port_is_live || {
    echo "bh-agent: Chrome failed to expose CDP after 10s — check $PORT_FILE" >&2
    exit 1
  }
fi

PORT=$(head -n1 "$PORT_FILE")
WS_PATH=$(tail -n1 "$PORT_FILE")
export BU_CDP_WS="ws://127.0.0.1:${PORT}${WS_PATH}"
export BU_NAME="${BU_NAME:-agent}"

exec browser-harness "$@"
```

## Uninstall

```bash
# Remove wrapper + isolated profile
rm ~/.local/bin/bh-agent
rm -rf ~/.browser-harness-chrome/
pkill -f "user-data-dir=$HOME/.browser-harness-chrome" 2>/dev/null

# Remove browser-harness itself
uv tool uninstall browser-harness
rm -rf ~/repos/integrations/browser-harness/

# Optionally remove uv (if not used for anything else)
brew uninstall uv

# Edit ~/.zshrc.local to drop the `export PATH="$HOME/.local/bin:$PATH"` line
```
