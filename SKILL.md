---
name: red-alert-hooks
version: 1.0.0
description: |
  Configure Claude Code hooks with Command & Conquer Red Alert classic sound effects.
  Downloads WAV files from GitHub and wires them to 8 Claude Code lifecycle events:
  SessionStart, Stop, Notification, PostToolUse(Write/Edit), PostToolUse(Bash),
  PostToolUseFailure, PermissionRequest, PreCompact.
  Works on Windows (PowerShell SoundPlayer), macOS (afplay), Linux (aplay/paplay).
allowed-tools:
  - Bash
  - Read
  - Edit
  - Write
  - AskUserQuestion
---

# Red Alert Hooks — Setup Skill

You are configuring Claude Code to play Command & Conquer Red Alert sound effects at key lifecycle events. Follow the steps below precisely.

---

## Sound Source

All WAV files come from this public GitHub repository (built specifically for Claude Code hooks):

**`https://github.com/mgmobrien/red-alert-sounds`**

Raw file base URL:
```
https://raw.githubusercontent.com/mgmobrien/red-alert-sounds/main/
```

Notable files available (browse the repo for 150+ more):

| File | Content |
|------|---------|
| `ra_construction_complete.wav` | Construction complete chime |
| `ra_mission_accomplished.wav` | "Mission accomplished!" |
| `ra_awaiting_orders_allied_infantry_1.wav` | "Awaiting orders!" |
| `ra_yes_sir_allied_infantry_1.wav` | "Yes sir!" |
| `ra_mechanic_sure_thing_boss.wav` | "Sure thing, boss!" |
| `ra_mission_failed.wav` | "Mission failed" |
| `ra_spy_commander.wav` | "Commander." |
| `ra_base_under_attack.wav` | Base under attack alarm |
| `ra_tanya_lets_rock.wav` | Tanya: "Let's rock!" |
| `ra_shock_burn_baby_burn.wav` | Shock trooper: "Burn baby burn!" |
| `ra_yes_sir_soviet_infantry_1.wav` | Soviet: "Yes sir!" |
| `td_mission_accomplished.wav` | Tiberian Dawn version |

---

## Step 1 — Detect OS and set paths

```bash
OS=$(uname -s 2>/dev/null || echo "Windows")
case "$OS" in
  Darwin)  SOUNDS_DIR="$HOME/sounds/redalert"; PLAY_CMD="afplay" ;;
  Linux)   SOUNDS_DIR="$HOME/sounds/redalert"; PLAY_CMD="aplay" ;;
  *)       SOUNDS_DIR="$HOME/sounds/redalert"; PLAY_CMD="powershell_soundplayer" ;;
esac
echo "OS: $OS | Dir: $SOUNDS_DIR | Player: $PLAY_CMD"
mkdir -p "$SOUNDS_DIR"
```

On Windows, `uname` may not exist — treat any error as Windows.

---

## Step 2 — Download the 8 core sound files

```bash
BASE="https://raw.githubusercontent.com/mgmobrien/red-alert-sounds/main"

files=(
  "ra_construction_complete.wav"
  "ra_mission_accomplished.wav"
  "ra_awaiting_orders_allied_infantry_1.wav"
  "ra_yes_sir_allied_infantry_1.wav"
  "ra_mechanic_sure_thing_boss.wav"
  "ra_mission_failed.wav"
  "ra_spy_commander.wav"
  "ra_base_under_attack.wav"
)

for f in "${files[@]}"; do
  curl -sL "$BASE/$f" -o "$SOUNDS_DIR/$f" && echo "✓ $f" || echo "✗ $f"
done
```

---

## Step 3 — Build the play command per OS

**Windows** (PowerShell SoundPlayer — synchronous, works without extra deps):
```
(New-Object System.Media.SoundPlayer 'C:\Users\USERNAME\sounds\redalert\FILE.wav').PlaySync()
```
Use `"shell": "powershell"` in the hook. Use Windows backslash paths. Get actual home dir from `$env:USERPROFILE` or expand `~` manually.

**macOS**:
```bash
afplay ~/sounds/redalert/FILE.wav
```

**Linux**:
```bash
aplay ~/sounds/redalert/FILE.wav 2>/dev/null || paplay ~/sounds/redalert/FILE.wav 2>/dev/null || true
```

---

## Step 4 — Read and merge settings.json

Target file: `~/.claude/settings.json` (global, applies to all projects).

**Always read the file first before editing.** Merge the `hooks` key — do not overwrite other keys (`env`, `permissions`, `statusLine`, etc.).

Add this hooks block:

```json
"hooks": {
  "SessionStart": [
    {
      "hooks": [{
        "type": "command",
        "command": "<PLAY_COMMAND for ra_construction_complete.wav>",
        "shell": "<powershell on Windows, omit on mac/linux>",
        "async": true
      }]
    }
  ],
  "Stop": [
    {
      "hooks": [{
        "type": "command",
        "command": "<PLAY_COMMAND for ra_mission_accomplished.wav>",
        "shell": "<powershell on Windows, omit on mac/linux>",
        "async": true
      }]
    }
  ],
  "Notification": [
    {
      "hooks": [{
        "type": "command",
        "command": "<PLAY_COMMAND for ra_awaiting_orders_allied_infantry_1.wav>",
        "shell": "<powershell on Windows, omit on mac/linux>",
        "async": true
      }]
    }
  ],
  "PostToolUse": [
    {
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "<PLAY_COMMAND for ra_yes_sir_allied_infantry_1.wav>",
        "shell": "<powershell on Windows, omit on mac/linux>",
        "async": true
      }]
    },
    {
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "<PLAY_COMMAND for ra_mechanic_sure_thing_boss.wav>",
        "shell": "<powershell on Windows, omit on mac/linux>",
        "async": true
      }]
    }
  ],
  "PostToolUseFailure": [
    {
      "hooks": [{
        "type": "command",
        "command": "<PLAY_COMMAND for ra_mission_failed.wav>",
        "shell": "<powershell on Windows, omit on mac/linux>",
        "async": true
      }]
    }
  ],
  "PermissionRequest": [
    {
      "hooks": [{
        "type": "command",
        "command": "<PLAY_COMMAND for ra_spy_commander.wav>",
        "shell": "<powershell on Windows, omit on mac/linux>",
        "async": true
      }]
    }
  ],
  "PreCompact": [
    {
      "hooks": [{
        "type": "command",
        "command": "<PLAY_COMMAND for ra_base_under_attack.wav>",
        "shell": "<powershell on Windows, omit on mac/linux>",
        "async": true
      }]
    }
  ]
}
```

---

## Step 5 — Validate JSON

On Windows:
```powershell
$s = Get-Content "$env:USERPROFILE\.claude\settings.json" -Raw | ConvertFrom-Json
$s.hooks | Get-Member -MemberType NoteProperty | Select-Object Name
```

On Mac/Linux:
```bash
python3 -c "import json; d=json.load(open(os.path.expanduser('~/.claude/settings.json'))); print(list(d['hooks'].keys()))"
```

Expect 8 keys: `SessionStart, Stop, Notification, PostToolUse, PostToolUseFailure, PermissionRequest, PreCompact` (PostToolUse contains 2 matchers).

---

## Step 6 — Test play each sound

Play each file once so the user can confirm they all work:

```bash
DEST="~/sounds/redalert"
for f in \
  ra_construction_complete \
  ra_mission_accomplished \
  ra_awaiting_orders_allied_infantry_1 \
  ra_yes_sir_allied_infantry_1 \
  ra_mechanic_sure_thing_boss \
  ra_mission_failed \
  ra_spy_commander \
  ra_base_under_attack; do
  echo "▶ $f"
  # Windows: powershell -c "(New-Object System.Media.SoundPlayer '$DEST/$f.wav').PlaySync()"
  # macOS/Linux: afplay "$DEST/$f.wav"
done
```

---

## Step 7 — Handoff

Tell the user:
1. **Restart Claude Code** (or open `/hooks`) for the new configuration to take effect.
2. The hooks are global — they apply to every project.
3. To swap any sound, edit `~/.claude/settings.json` and point to a different WAV from the repo.
4. To disable a specific hook without deleting it, add `"async": false` and let the command silently fail, or remove that hook entry.

---

## Customization Guide (share with users)

### Swap a sound
Replace the filename in `settings.json`. Browse all 150+ files at:
`https://github.com/mgmobrien/red-alert-sounds`

Popular alternatives:
- `ra_tanya_lets_rock.wav` — Tanya's "Let's rock!" (fun Stop sound)
- `ra_shock_burn_baby_burn.wav` — "Burn baby burn!" (wild Bash sound)
- `ra_yes_sir_soviet_infantry_1.wav` — Soviet accent variant
- `td_mission_accomplished.wav` — Tiberian Dawn version of mission accomplished
- `ra_mechanic_yee_haw.wav` — "Yee-haw!" mechanic

### Add more hooks
Claude Code supports additional events you can hook into:
- `TaskCompleted` — when a todo task is checked off
- `SubagentStop` — when a background agent finishes
- `SessionEnd` — when the session closes
- `PostCompact` — after context compaction completes

### Reduce noise
If `PostToolUse(Bash)` fires too often (every shell command), remove the Bash matcher entry from `PostToolUse` in `settings.json`.

### Download the entire library
```bash
BASE="https://raw.githubusercontent.com/mgmobrien/red-alert-sounds/main"
# Get full file list via GitHub API
curl -s "https://api.github.com/repos/mgmobrien/red-alert-sounds/contents/" \
  | python3 -c "
import sys, json
for f in json.load(sys.stdin):
    if f['name'].endswith('.wav'):
        print(f['name'])
" | while read fname; do
  curl -sL "$BASE/$fname" -o "$SOUNDS_DIR/$fname" && echo "✓ $fname"
done
```
