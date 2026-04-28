# 🔊 Red Alert Hooks for Claude Code

> Play classic **Command & Conquer: Red Alert** voice lines at key Claude Code lifecycle events. "Mission accomplished!", "Yes sir!", "Awaiting orders!" — your terminal just became a war room.

[English](README.md) | [中文](README.zh.md)

---

## Demo

| Event | Sound | Voice Line |
|-------|-------|-----------|
| 🚀 Session starts | `ra_yes_sir` | *"Yes sir!"* |
| ✅ Claude finishes a turn | `ra_mission_accomplished` | *"Mission accomplished!"* |
| 🔔 Notification | `ra_awaiting_orders` | *"Awaiting orders!"* |
| ❌ Tool fails | `ra_mission_failed` | *"Mission failed"* |
| 🕵️ Permission needed | `ra_spy_commander` | *"Commander."* |
| ⚠️ Context compacting | `ra_base_under_attack` | *Base under attack alarm* |

---

## Install

```bash
npx skills-installer install @QiYongchuan/red-alert-hooks/red-alert-hooks --client claude-code
```

Then in Claude Code, type:

```
/red-alert-hooks
```

Claude will auto-detect your OS, configure all 8 hooks, and play a test run.

**Restart Claude Code** after setup for hooks to take effect.

> Sound files (448KB total) are **bundled** — no extra downloads needed to get started.

---

## How it works

This skill uses [Claude Code hooks](https://docs.anthropic.com/en/docs/claude-code/hooks) — shell commands that fire automatically at lifecycle events. On Windows it uses PowerShell's `SoundPlayer`; on macOS `afplay`; on Linux `aplay`/`paplay`.

All hooks use `"async": true` so sounds play in the background without blocking your workflow.

---

## Customize

**Swap a sound** — edit `~/.claude/settings.json` and point any hook to a different `.wav` file path.

**150+ more sounds** available in the source library:

👉 [`github.com/mgmobrien/red-alert-sounds`](https://github.com/mgmobrien/red-alert-sounds)

Download any extra file:
```bash
curl -sL "https://raw.githubusercontent.com/mgmobrien/red-alert-sounds/main/ra_tanya_lets_rock.wav" \
  -o "$HOME/.claude/skills/red-alert-hooks/sounds/ra_tanya_lets_rock.wav"
```

Popular extras:

| File | Line |
|------|------|
| `ra_tanya_lets_rock.wav` | *"Let's rock!"* |
| `ra_shock_burn_baby_burn.wav` | *"Burn baby burn!"* |
| `ra_yes_sir_soviet_infantry_1.wav` | Soviet accent *"Yes sir!"* |
| `ra_mechanic_yee_haw.wav` | *"Yee-haw!"* |
| `td_mission_accomplished.wav` | Tiberian Dawn version |

**Add more hook events:**

```json
"TaskCompleted":  [ ... ],  // todo task checked off
"SubagentStop":   [ ... ],  // background agent done
"SessionEnd":     [ ... ],  // session closes
"PostCompact":    [ ... ]   // after context compaction
```

**Too noisy?** Remove the `Bash` matcher from `PostToolUse` in `~/.claude/settings.json` to stop the per-command sounds.

---

## Platform support

| Platform | Play method |
|----------|-------------|
| Windows | PowerShell `System.Media.SoundPlayer` |
| macOS | `afplay` (built-in) |
| Linux | `aplay` / `paplay` |

---

## Credits

Sound files sourced from [`mgmobrien/red-alert-sounds`](https://github.com/mgmobrien/red-alert-sounds) — a public library of C&C Red Alert WAVs built for Claude Code hooks.

Original game audio © Westwood Studios / Electronic Arts.
