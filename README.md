# Hermes Multi-Bot Discord Rules

> 一个 Skill，解决多 Bot 同频道的「抢话」「串味」「调度混乱」三大顽疾。

把你的 Discord 频道变成一间运作良好的「AI 虚拟公司」——老板发言全员可见，但只有被点名的人回答。每个 Bot 角色独立，风格互不污染。

> **English Summary:** An open-source Skill for coordinating multiple Hermes Bots within a single Discord channel. It addresses common multi-bot pitfalls — bots interrupting each other, role/personality confusion, scheduling chaos, and unsafe multi-agent behaviors. Built from real-world experience running multiple Hermes + Discord bots in production chat groups.

## 这解决了什么问题？

你在 Discord 频道里放了两个（或更多）Hermes Bot。然后：

- ❌ Bot A 没被 @ 也抢答
- ❌ Bot B 不知不觉模仿了 Bot A 的风格
- ❌ 叫 Bot A 调度 Bot B，结果 Bot A 自己把活干了
- ❌ 说"闭嘴"，Bot 回了一句"已闭嘴"（又说话了！）

**根本原因**：所有 Bot 共享同一个 LLM 上下文窗口。任何 Bot 的发言都会进入其他 Bot 的「视野」，造成无意识的风格沾染和应答混乱。

这个 Skill 用六条铁律堵住所有漏洞。

## 快速开始

### 1. 安装 Skill

将本仓库克隆到你的 Hermes skills 目录：

```bash
cd ~/.hermes/skills/
git clone https://github.com/chenguru/hermes-multi-bot-discord-rules.git multi-bot-discord-rules
```

或者直接下载 `SKILL.md`，放入 `~/.hermes/skills/multi-bot-discord-rules/`。

### 2. 配置每个 Bot

对每个 Bot 的 `config.yaml`：

```yaml
require_mention: true      # 非 @ 不答

discord:
  auto_thread: false       # 禁止自动创建子区
```

对主 Bot 的 `.env`：

```bash
DISCORD_ALLOW_BOTS=mentions   # 允许接收其他 Bot 的 @mention
```

### 3. 写入 SOUL.md

对每个 Bot，将 `templates/SOUL-template.md` 的内容复制到该 Bot 的 SOUL.md **最顶端**，并按自身角色填写风格声明。

### 4. 重启所有 Gateway

```bash
systemctl --user restart hermes-gateway-{bot-name}
```

### 5. 验证

在频道里依次测试：

```
@BotA 你好              → 只有 BotA 回复
@BotA 叫 BotB 做 X      → BotA 发出 <@BotB_ID>，然后闭嘴
@BotB                   → BotB 执行并回复
止                      → Bot 确认后完全沉默
```

## 核心规则速览

| # | 规则 | 一句话 |
|---|------|--------|
| 1 | **点名才答** | 不被 @，绝对不开口 |
| 2 | **风格隔离** | 别的 Bot 的风格，碰都不碰 |
| 3 | **一问一答** | 问什么答什么，不续话，不反问 |
| 4 | **调度透明** | @ 发出即完成，不追加"已通知" |
| 5 | **回报-确认-收声** | 收到对方结果，说"收到"就停 |
| 6 | **沉默即沉默** | "止"后说≤10字确认，然后真正闭嘴 |

## 为什么风格隔离是最高优先级？

多 Bot 同频道时，LLM 上下文窗口里充满了其他 Bot 的消息。如果 Bot A 用古风文言回复，Bot B 的下一条回复就可能不自觉带上古风味。

这种污染是**累积的、不可逆的**——一旦开始，角色就在崩塌。

解决方案：每个 Bot 的 SOUL.md 顶端**硬编码**自己的风格声明 + "绝不模仿他人"的铁律。这是角色独立性的防火墙。

## 仓库结构

```
├── SKILL.md                 # 核心 Skill（给 Bot 读的）
├── README.md                # 中文说明（本文件）
├── README_EN.md             # English README
├── templates/
│   ├── SOUL-template.md     # 风格隔离声明模板
│   └── config-snippets.yaml # 关键配置片段
└── LICENSE                  # MIT
```

## 适用场景

- ✅ 2+ 个 Hermes Bot 在同一个 Discord 频道
- ✅ Bot 之间需要互相调度（@mention 触发）
- ✅ 每个 Bot 有独立的人设和风格
- ✅ 需要一个总管 Bot 做调度，其他 Bot 做执行

不适用：
- ❌ 只有一个 Bot（不需要多 Bot 规则）
- ❌ Telegram（Telegram Bot 之间无法互见和 @）

## 常见问题

**Q: 新 Bot 加入后为什么还是抢答？**

检查 `require_mention: true` 是否生效。修改后必须重启 Gateway。

**Q: @mention 发了但目标 Bot 没反应？**

Discord 的 @mention 格式必须是 `<@数字ID>`，不是 `@Bot名`。可以在 Discord 开发者模式右键 Bot 头像复制 ID。

**Q: 喊暗号两个 Bot 同时响应？**

检查两个 Bot 的 `mention_patterns` 是否有重叠。确保暗号互不包含。

**Q: 修改配置后不生效？**

每次改 config.yaml、SOUL.md 或 .env 后，**必须重启对应 Bot 的 Gateway**。

## Open Source Value

This is an early-stage open-source project that distills real multi-bot coordination experience into reusable, tested rules. Rather than a theoretical framework, it provides concrete configuration templates and communication protocols that anyone building AI Agents, Discord Bots, or Hermes Skills can adopt immediately. The goal is to reduce the chaos and risk that inevitably arise when multiple autonomous agents share the same channel — a problem that grows as multi-agent systems become more common.

## Security Considerations

- **Reduces prompt-style pollution:** Each bot has a unique mention pattern and a hardcoded style declaration in SOUL.md, preventing one bot's system prompt from leaking into another's conversation context.
- **Reduces misrouting:** The `require_mention` + exclusive codeword design ensures tasks only reach the intended bot, avoiding accidental disclosure or incorrect execution.
- **Reduces unintended bot responses:** By disabling free response in group contexts, bots won't spontaneously reply to messages they shouldn't process.
- **Helps multi-bot workflows stay safer and more controllable:** The delegation protocol routes inter-bot communication through explicit @mentions rather than implicit context sharing, keeping task routing predictable and auditable.

## 许可

MIT License — 自由使用、修改、分发。

## 贡献

欢迎提 Issue 和 PR。如果你用这些规则跑通了自己的多 Bot 部署，欢迎分享经验。
