# XFCE4 Screenshot to Clipboard for Claude Code

Screenshot → save to file → copy path to clipboard for pasting into Claude Code terminal.

## Problem

Claude Code terminal doesn't support direct image pasting. This script provides a simple workaround to reference screenshots.

## Solution

Take screenshot → auto-saves to `~/.tmp/clipboard/` → path copied to clipboard → paste path in Claude Code.

Auto-cleans screenshots older than 1 day.

## Requirements

- Linux (XFCE4)
- `scrot` (screenshot tool)
- `xsel` (clipboard)
- `libnotify` (notifications)

## One-Line Install

```bash
cat << 'EOF' > ~/.local/bin/screenshot-to-clipboard
#!/bin/bash
SAVE_DIR="$HOME/.tmp/clipboard"
mkdir -p "$SAVE_DIR"
find "$SAVE_DIR" -name "screenshot_*.png" -type f -mtime +0 -delete
FILENAME="screenshot_$(date +%Y%m%d_%H%M%S).png"
FILEPATH="$SAVE_DIR/$FILENAME"
scrot -s -f "$FILEPATH"
if [ -f "$FILEPATH" ]; then
    echo -n "$FILEPATH" | xsel -b
    notify-send "Screenshot saved" "Path copied to clipboard:\n$FILENAME" -t 2000
else
    notify-send "Screenshot cancelled" -t 1000
fi
EOF
chmod +x ~/.local/bin/screenshot-to-clipboard
sudo apt install scrot xsel libnotify-bin -y
xfconf-query -c xfce4-keyboard-shortcuts -p "/commands/custom/<Alt>p" -n -t string -s "$HOME/.local/bin/screenshot-to-clipboard"
```

## Manual Install

### 1. Install dependencies

```bash
sudo apt install scrot xsel libnotify-bin -y
```

### 2. Create the script

```bash
mkdir -p ~/.local/bin

cat << 'EOF' > ~/.local/bin/screenshot-to-clipboard
#!/bin/bash
# Screenshot → save to temp dir → copy path to clipboard
# For XFCE4 - Uses scrot for reliable silent operation

SAVE_DIR="$HOME/.tmp/clipboard"
mkdir -p "$SAVE_DIR"

# Delete screenshots older than 1 day
find "$SAVE_DIR" -name "screenshot_*.png" -type f -mtime +0 -delete

FILENAME="screenshot_$(date +%Y%m%d_%H%M%S).png"
FILEPATH="$SAVE_DIR/$FILENAME"

# Take screenshot with scrot (region selection)
# -s = select region, -f = freeze screen while selecting
scrot -s -f "$FILEPATH"

# Check if screenshot was taken
if [ -f "$FILEPATH" ]; then
    # Copy path to clipboard
    echo -n "$FILEPATH" | xsel -b
    notify-send "Screenshot saved" "Path copied to clipboard:\n$FILENAME" -t 2000
else
    notify-send "Screenshot cancelled" -t 1000
fi
EOF

chmod +x ~/.local/bin/screenshot-to-clipboard
```

### 3. Set up keyboard shortcut

**Via terminal:**

```bash
xfconf-query -c xfce4-keyboard-shortcuts -p "/commands/custom/<Alt>p" -n -t string -s "$HOME/.local/bin/screenshot-to-clipboard"
```

**Via GUI:**

Settings Manager → Keyboard → Application Shortcuts → Add:
- Command: `/home/YOUR_USERNAME/.local/bin/screenshot-to-clipboard`
- Shortcut: `Alt + P`

## Usage

1. Press `Alt + P`
2. Select area with mouse
3. Path is copied to clipboard
4. Paste into Claude Code

## Customization

**Change screenshot directory:**

Edit `SAVE_DIR` in the script.

**Change keybinding:**

```bash
# Remove old binding
xfconf-query -c xfce4-keyboard-shortcuts -p "/commands/custom/<Alt>p" -r

# Add new binding (example: Ctrl+Shift+S)
xfconf-query -c xfce4-keyboard-shortcuts -p "/commands/custom/<Ctrl><Shift>s" -n -t string -s "$HOME/.local/bin/screenshot-to-clipboard"
```

**Full screen instead of region:**

Change `scrot -s -f` to `scrot` in the script.

**Active window instead of region:**

Change `scrot -s -f` to `scrot -u` in the script.

## Uninstall

```bash
rm ~/.local/bin/screenshot-to-clipboard
xfconf-query -c xfce4-keyboard-shortcuts -p "/commands/custom/<Alt>p" -r
```

## Credits

Inspired by [claude-code-linux-screenshot](https://github.com/andresparra1980/claude-code-linux-screenshot) (GNOME version).

## License

MIT
