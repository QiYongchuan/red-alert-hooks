# 🔊 红警音效 Hooks for Claude Code

> 让 Claude Code 在关键时刻播放《命令与征服：红色警戒》的经典语音。"Mission accomplished!"、"Yes sir!"、"Awaiting orders!" ——你的终端瞬间变成作战指挥室。

[English](README.md) | 中文

---

## 效果演示

| 事件 | 音效 | 语音内容 |
|------|------|---------|
| 🚀 启动 Claude Code | `ra_yes_sir` | *"Yes sir!"* |
| ✅ Claude 完成一轮回复 | `ra_mission_accomplished` | *"Mission accomplished!"* |
| 🔔 收到通知 | `ra_awaiting_orders` | *"Awaiting orders!"* |
| ❌ 工具执行失败 | `ra_mission_failed` | *"Mission failed"* |
| 🕵️ 需要你审批操作 | `ra_spy_commander` | *"Commander."* |
| ⚠️ 上下文即将压缩 | `ra_base_under_attack` | *基地遭受攻击警报* |

---

## 安装

```bash
npx skills-installer install @QiYongchuan/red-alert-hooks/red-alert-hooks --client claude-code
```

然后在 Claude Code 里输入：

```
/red-alert-hooks
```

Claude 会自动检测你的操作系统、配置全部 hooks、并逐个播放测试音效。

**配置完成后重启 Claude Code** 即可生效。

> 音效文件（448KB）**已打包在 skill 里**，安装即用，无需额外下载。

---

## 工作原理

本 skill 使用 [Claude Code Hooks 机制](https://docs.anthropic.com/en/docs/claude-code/hooks) —— 在 Claude Code 生命周期的关键节点自动执行 shell 命令来播放音效。

- Windows：使用 PowerShell `System.Media.SoundPlayer`
- macOS：使用系统内置 `afplay`
- Linux：使用 `aplay` / `paplay`

所有 hooks 均设置 `"async": true`，音效在后台播放，不阻塞你的工作流。

---

## 自定义音效

**换掉某个音效**：编辑 `~/.claude/settings.json`，将对应 hook 里的文件名改成你想要的 `.wav` 路径即可。

**150+ 更多音效**，来自原始音效库：

👉 [`github.com/mgmobrien/red-alert-sounds`](https://github.com/mgmobrien/red-alert-sounds)

下载任意额外音效：
```bash
curl -sL "https://raw.githubusercontent.com/mgmobrien/red-alert-sounds/main/ra_tanya_lets_rock.wav" \
  -o "$HOME/.claude/skills/red-alert-hooks/sounds/ra_tanya_lets_rock.wav"
```

热门备选：

| 文件 | 语音 |
|------|------|
| `ra_tanya_lets_rock.wav` | Tanya："Let's rock!" |
| `ra_shock_burn_baby_burn.wav` | 电击兵："Burn baby burn!" |
| `ra_yes_sir_soviet_infantry_1.wav` | 苏联步兵口音 "Yes sir!" |
| `ra_mechanic_yee_haw.wav` | 机械师："Yee-haw!" |
| `ra_construction_complete.wav` | 建造完成提示音 |
| `td_mission_accomplished.wav` | 泰伯利亚黎明版本 |

**可扩展的 Hook 事件：**

| 事件 | 触发时机 |
|------|---------|
| `TaskCompleted` | 待办任务被勾选完成时 |
| `SubagentStop` | 后台 agent 任务结束时 |
| `SessionEnd` | 会话关闭时 |
| `PostCompact` | 上下文压缩完成后 |
| `PostToolUse(Write\|Edit)` | 每次写入/编辑文件后 |
| `PostToolUse(Bash)` | 每次执行命令后（注意：较频繁） |

---

## 平台支持

| 平台 | 播放方式 |
|------|---------|
| Windows | PowerShell `System.Media.SoundPlayer` |
| macOS | `afplay`（系统内置） |
| Linux | `aplay` / `paplay` |

---

## 致谢

音效文件来源：[`mgmobrien/red-alert-sounds`](https://github.com/mgmobrien/red-alert-sounds) —— 专为 Claude Code Hooks 整理的红警 WAV 音效库。

游戏原始音频版权归 Westwood Studios / Electronic Arts 所有。
