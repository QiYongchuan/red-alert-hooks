---
name: red-alert-hooks
version: 1.1.0
description: |
  Configure Claude Code hooks with Command & Conquer Red Alert classic sound effects.
  Maps 8 Claude Code lifecycle events to iconic unit voices (Mission accomplished!,
  Yes sir!, Awaiting orders!, Sure thing boss!, etc.).
  Works on Windows (PowerShell SoundPlayer), macOS (afplay), Linux (aplay/paplay).
  Sound files are bundled — no download needed to get started.
allowed-tools:
  - Bash
  - Read
  - Edit
  - Write
  - AskUserQuestion
---

# Red Alert Hooks — Setup Skill

You are configuring Claude Code to play Command & Conquer Red Alert sound effects at key lifecycle events.

---

## Sound Files

8 core WAV files are **bundled with this skill** in the `sounds/` subdirectory — no download needed.

Skill sounds directory (after installation):
```
~/.claude/skills/red-alert-hooks/sounds/
```

| File | Hook | Content |
|------|------|---------|
| `ra_construction_complete.wav` | SessionStart | Construction complete chime |
| `ra_mission_accomplished.wav` | Stop | "Mission accomplished!" |
| `ra_awaiting_orders_allied_infantry_1.wav` | Notification | "Awaiting orders!" |
| `ra_yes_sir_allied_infantry_1.wav` | PostToolUse Write/Edit | "Yes sir!" |
| `ra_mechanic_sure_thing_boss.wav` | PostToolUse Bash | "Sure thing, boss!" |
| `ra_mission_failed.wav` | PostToolUseFailure | "Mission failed" |
| `ra_spy_commander.wav` | PermissionRequest | "Commander." |
| `ra_base_under_attack.wav` | PreCompact | Base under attack alarm |

For **150+ additional sounds** (Tanya, Soviet infantry, Shock Trooper, etc.), see:
**`https://github.com/mgmobrien/red-alert-sounds`**

---

## Step 1 — Detect OS and resolve sounds path

```bash
# Detect OS
OS=$(uname -s 2>/dev/null || echo "Windows")

# Resolve the skill's bundled sounds directory
# skills-installer places files at ~/.claude/skills/<skill-name>/
SKILL_SOUNDS="$HOME/.claude/skills/red-alert-hooks/sounds"

# Verify bundled files exist
if ls "$SKILL_SOUNDS"/*.wav &>/dev/null; then
  echo "✓ Bundled sounds found at: $SKILL_SOUNDS"
else
  echo "⚠ Bundled sounds not found — will download from GitHub"
  SKILL_SOUNDS=""
fi

echo "OS: $OS"
```

---

## Step 2 — If bundled sounds missing, download them

Only run this if Step 1 shows bundled sounds are missing.

```bash
BASE="https://raw.githubusercontent.com/mgmobrien/red-alert-sounds/main"
DEST="$HOME/.claude/skills/red-alert-hooks/sounds"
mkdir -p "$DEST"

for f in \
  ra_construction_complete.wav \
  ra_mission_accomplished.wav \
  ra_awaiting_orders_allied_infantry_1.wav \
  ra_yes_sir_allied_infantry_1.wav \
  ra_mechanic_sure_thing_boss.wav \
  ra_mission_failed.wav \
  ra_spy_commander.wav \
  ra_base_under_attack.wav; do
  curl -sL "$BASE/$f" -o "$DEST/$f" && echo "✓ $f" || echo "✗ $f"
done
```

---

## Step 3 — Build the play command per OS

Resolve the actual absolute path to the sounds directory.

**Windows** — use PowerShell SoundPlayer with Windows-style path:
```powershell
# Get Windows path (convert from bash-style if needed)
$soundsDir = "$env:USERPROFILE\.claude\skills\red-alert-hooks\sounds"
# Play command format:
(New-Object System.Media.SoundPlayer '$soundsDir\FILE.wav').PlaySync()
```
Use `"shell": "powershell"` in the hook config.

**macOS**:
```bash
afplay ~/.claude/skills/red-alert-hooks/sounds/FILE.wav
```

**Linux**:
```bash
aplay ~/.claude/skills/red-alert-hooks/sounds/FILE.wav 2>/dev/null \
  || paplay ~/.claude/skills/red-alert-hooks/sounds/FILE.wav 2>/dev/null || true
```

---

## Step 4 — Read and merge settings.json

Target: `~/.claude/settings.json` (global settings, applies to all projects).

**Always read the file first.** Merge the `hooks` key — preserve all other keys (`env`, `permissions`, `statusLine`, etc.).

On **Windows**, use backslash paths and `"shell": "powershell"`. The sounds path is:
`C:\Users\<USERNAME>\.claude\skills\red-alert-hooks\sounds\`

Get the actual username from `$env:USERPROFILE` or `echo $HOME` and convert slashes.

Hooks block to merge in:

```json
"hooks": {
  "SessionStart": [{
    "hooks": [{
      "type": "command",
      "command": "<play ra_construction_complete.wav>",
      "shell": "powershell",
      "async": true
    }]
  }],
  "Stop": [{
    "hooks": [{
      "type": "command",
      "command": "<play ra_mission_accomplished.wav>",
      "shell": "powershell",
      "async": true
    }]
  }],
  "Notification": [{
    "hooks": [{
      "type": "command",
      "command": "<play ra_awaiting_orders_allied_infantry_1.wav>",
      "shell": "powershell",
      "async": true
    }]
  }],
  "PostToolUse": [
    {
      "matcher": "Write|Edit",
      "hooks": [{
        "type": "command",
        "command": "<play ra_yes_sir_allied_infantry_1.wav>",
        "shell": "powershell",
        "async": true
      }]
    },
    {
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "<play ra_mechanic_sure_thing_boss.wav>",
        "shell": "powershell",
        "async": true
      }]
    }
  ],
  "PostToolUseFailure": [{
    "hooks": [{
      "type": "command",
      "command": "<play ra_mission_failed.wav>",
      "shell": "powershell",
      "async": true
    }]
  }],
  "PermissionRequest": [{
    "hooks": [{
      "type": "command",
      "command": "<play ra_spy_commander.wav>",
      "shell": "powershell",
      "async": true
    }]
  }],
  "PreCompact": [{
    "hooks": [{
      "type": "command",
      "command": "<play ra_base_under_attack.wav>",
      "shell": "powershell",
      "async": true
    }]
  }]
}
```

---

## Step 5 — Validate JSON

**Windows:**
```powershell
$s = Get-Content "$env:USERPROFILE\.claude\settings.json" -Raw | ConvertFrom-Json
$s.hooks | Get-Member -MemberType NoteProperty | Select-Object Name
```

**macOS / Linux:**
```bash
python3 -c "
import json, os
d = json.load(open(os.path.expanduser('~/.claude/settings.json')))
print(list(d['hooks'].keys()))
"
```

Expect 7 hook event keys (PostToolUse contains 2 matchers).

---

## Step 6 — Test play each sound

Play all 8 sounds once for the user to confirm:

**Windows:**
```powershell
$dir = "$env:USERPROFILE\.claude\skills\red-alert-hooks\sounds"
@(
  "ra_construction_complete",
  "ra_mission_accomplished",
  "ra_awaiting_orders_allied_infantry_1",
  "ra_yes_sir_allied_infantry_1",
  "ra_mechanic_sure_thing_boss",
  "ra_mission_failed",
  "ra_spy_commander",
  "ra_base_under_attack"
) | ForEach-Object {
  Write-Host "▶ $_"
  (New-Object System.Media.SoundPlayer "$dir\$_.wav").PlaySync()
  Start-Sleep -Milliseconds 500
}
```

**macOS / Linux:**
```bash
DIR="$HOME/.claude/skills/red-alert-hooks/sounds"
for f in ra_construction_complete ra_mission_accomplished \
  ra_awaiting_orders_allied_infantry_1 ra_yes_sir_allied_infantry_1 \
  ra_mechanic_sure_thing_boss ra_mission_failed \
  ra_spy_commander ra_base_under_attack; do
  echo "▶ $f"
  afplay "$DIR/$f.wav" 2>/dev/null || aplay "$DIR/$f.wav" 2>/dev/null
  sleep 0.5
done
```

---

## Step 7 — Handoff

Tell the user:
1. **Restart Claude Code** (or open `/hooks`) for hooks to take effect.
2. Hooks are global — apply to every project.
3. To swap a sound, point any hook's command to a different WAV file path.
4. To disable one hook, remove its entry from `~/.claude/settings.json`.

---

## Customization — Expanding with more sounds

Browse **150+ additional Red Alert sounds** at:
`https://github.com/mgmobrien/red-alert-sounds`

Download any extra sound:
```bash
curl -sL "https://raw.githubusercontent.com/mgmobrien/red-alert-sounds/main/FILENAME.wav" \
  -o "$HOME/.claude/skills/red-alert-hooks/sounds/FILENAME.wav"
```

Popular picks:
| File | Content |
|------|---------|
| `ra_tanya_lets_rock.wav` | Tanya: "Let's rock!" |
| `ra_shock_burn_baby_burn.wav` | "Burn baby burn!" |
| `ra_yes_sir_soviet_infantry_1.wav` | Soviet accent "Yes sir!" |
| `ra_mechanic_yee_haw.wav` | "Yee-haw!" |
| `td_mission_accomplished.wav` | Tiberian Dawn version |
| `ra_tanya_shake_it_baby.wav` | Tanya: "Shake it baby!" |

Additional hook events you can add:
| Event | When it fires |
|-------|--------------|
| `TaskCompleted` | A todo task is checked off |
| `SubagentStop` | A background agent finishes |
| `SessionEnd` | Session closes |
| `PostCompact` | After context compaction |

### Reduce noise
If `PostToolUse(Bash)` fires too often, remove the Bash matcher entry from `PostToolUse` in `settings.json`.

### Download the full library
```bash
BASE="https://raw.githubusercontent.com/mgmobrien/red-alert-sounds/main"
DEST="$HOME/.claude/skills/red-alert-hooks/sounds"
curl -s "https://api.github.com/repos/mgmobrien/red-alert-sounds/contents/" \
  | python3 -c "
import sys, json
for f in json.load(sys.stdin):
    if f['name'].endswith('.wav'):
        print(f['name'])
" | while read fname; do
  curl -sL "$BASE/$fname" -o "$DEST/$fname" && echo "✓ $fname"
done
```
