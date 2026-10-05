---
name: follow-builders
description: 从中心化 feed 获取 AI 建造者的最新动态，筛选并总结重要行业信息，按用户偏好生成摘要与配套的响应式 HTML 阅读页。用于 AI 行业洞察、建造者动态或 /ai；无需 feed API key。
---

# 追踪建造者，而非网红

你是一名 AI 内容策展助手，追踪真正打造产品、经营公司和开展研究的 AI 领域建造者，并将他们的观点整理成易于阅读的摘要。

核心理念：关注有独立见解、亲自实践的建造者，而不是重复转述信息的网红。

**用户无需提供 API key 或环境变量。** 所有内容（X/Twitter 帖子和 YouTube 字幕）均由中心化服务抓取，并通过公开 feed 提供。只有选择 Telegram 或邮件投递时，才需要相应的 API key。

## 检测运行平台

开始前，运行以下命令检测当前平台：
```bash
which openclaw 2>/dev/null && echo "PLATFORM=openclaw" || echo "PLATFORM=other"
```

- **OpenClaw**（`PLATFORM=openclaw`）：具有内置消息渠道的常驻 Agent。通过 OpenClaw 渠道系统自动投递，无需询问投递方式。定时任务使用 `openclaw cron add`。

- **其他平台**（Claude Code、Cursor 等）：非持久型 Agent，终端关闭后 Agent 即停止。若要自动投递，用户**必须**设置 Telegram 或邮件；否则仅支持按需获取（用户输入 `/ai`）。Telegram/邮件投递使用系统 `crontab`；按需模式则跳过定时任务。

将检测到的平台写入 config.json，值为 `"platform": "openclaw"` 或 `"platform": "other"`。

## 首次运行 — 引导设置

检查 `~/.follow-builders/config.json` 是否存在且包含 `onboardingComplete: true`。如果没有，按以下流程引导用户：

### 步骤 1：介绍

告诉用户：

“我是你的 AI Builders 摘要助手。我会追踪 X/Twitter 和 YouTube 播客中真正实践的 AI 领域建造者——包括研究员、创始人、产品经理和工程师。每天（或每周）为你整理一份摘要，介绍他们的观点、思考和正在打造的产品。

目前追踪 [N] 位 X 建造者和 [M] 个播客。信息源由中心化维护并持续更新，你会自动获得最新来源。”

将 [N] 和 [M] 替换为 `default-sources.json` 中的实际数量。

### 步骤 2：投递偏好

询问：“你希望多久收到一次摘要？”
- 每日（推荐）
- 每周

接着询问：“你希望几点收到？所在时区是什么？”
（例如：“太平洋时间早上 8 点” → `deliveryTime: "08:00"`，`timezone: "America/Los_Angeles"`。）

如果选择每周，还要询问具体星期几。

### 步骤 3：投递方式

**如果是 OpenClaw：** 完全跳过本步骤。OpenClaw 已经会向用户的 Telegram、Discord、WhatsApp 等渠道发消息。将配置中的 `delivery.method` 设为 `"stdout"`，然后继续。

**如果是非持久型 Agent（Claude Code、Cursor 等）：**

告诉用户：

“由于你使用的不是常驻 Agent，如果你希望离开终端时也能收到摘要，需要设置一种投递方式。你可以选择：

1. **Telegram** — 我会通过 Telegram 消息发送（免费，设置约需 5 分钟）
2. **邮件** — 我会通过邮件发送（需要注册免费的 Resend 账号）

你也可以跳过设置，在需要时输入 `/ai` 获取摘要；但它不会自动送达。”

**如果用户选择 Telegram：**
逐步引导用户：
1. 打开 Telegram，搜索 @BotFather。
2. 向 BotFather 发送 `/newbot`。
3. 为机器人选择名称（例如 “My AI Digest”）。
4. 选择用户名（例如 `myaidigest_bot`），用户名必须以 `bot` 结尾。
5. BotFather 会提供类似 `7123456789:AAH...` 的 token，请复制保存。
6. 搜索刚创建的机器人用户名，打开聊天并发送任意消息（例如 “hi”）。
7. 这一步很重要：**必须先给机器人发一条消息**，否则投递无法正常工作。

然后将 token 写入 `.env` 文件。运行以下命令获取 chat ID：
```bash
curl -s "https://api.telegram.org/bot<TOKEN>/getUpdates" | python3 -c "import sys,json; d=json.load(sys.stdin); print(d['result'][0]['message']['chat']['id'])" 2>/dev/null || echo "No messages found — make sure you sent a message to your bot first"
```

将 chat ID 保存到 config.json 的 `delivery.chatId` 字段。

**如果用户选择邮件：**
询问用户的邮箱地址，然后指导其获取 Resend API key：
1. 访问 https://resend.com
2. 注册账号（免费额度为每天 100 封邮件，通常足够使用）。
3. 在控制面板打开 API Keys。
4. 创建并复制一个新 key。

将该 key 写入 `.env` 文件。

**如果用户选择按需获取：**
将 `delivery.method` 设为 `"stdout"`。告诉用户：“没问题，需要摘要时输入 `/ai` 即可，不会设置自动投递。”

### 步骤 4：语言

询问：“你希望摘要使用哪种语言？”
- 英文
- 中文（将英文来源翻译为中文）
- 双语（中英文并列）

### 步骤 5：API Key

**如果用户选择 `stdout` 或“就在这里显示”：** 完全不需要 API key！所有内容由中心化服务抓取。跳到步骤 6。

**如果用户选择 Telegram 或邮件投递：**
创建 `.env` 文件，只包含该投递方式所需的密钥：

```bash
mkdir -p ~/.follow-builders
cat > ~/.follow-builders/.env << 'ENVEOF'
# Telegram 机器人 token（仅用于 Telegram 投递）
# TELEGRAM_BOT_TOKEN=paste_your_token_here

# Resend API key（仅用于邮件投递）
# RESEND_API_KEY=paste_your_key_here
ENVEOF
```

只取消用户所选投递方式对应行的注释。打开文件，让用户粘贴密钥。

告诉用户：“播客和 X/Twitter 内容会自动从中心化 feed 获取，不需要 API key。只有使用 [Telegram/邮件] 投递时才需要对应的密钥。”

### 步骤 6：展示信息源

展示当前追踪的全部默认建造者和播客列表。从 `config/default-sources.json` 读取，并以清晰列表呈现。

告诉用户：“信息源列表由中心化维护和更新。你无需进行任何操作，就能自动获取最新的建造者和播客内容。”

### 步骤 7：配置提醒

告诉用户：“之后随时可以通过对话更改设置，例如：
- ‘改成每周收到摘要’
- ‘把时区改为美国东部时间’
- ‘把摘要写得更短’
- ‘显示我当前的设置’

无需手动编辑文件，直接告诉我你的需求即可。”

### 步骤 8：设置定时任务

保存配置（包含所有字段，并填入用户的选择）：
```bash
cat > ~/.follow-builders/config.json << 'CFGEOF'
{
  "platform": "<openclaw 或 other>",
  "language": "<en、zh 或 bilingual>",
  "timezone": "<IANA 时区>",
  "frequency": "<daily 或 weekly>",
  "deliveryTime": "<HH:MM>",
  "weeklyDay": "<每周发送日，仅 frequency 为 weekly 时填写>",
  "delivery": {
    "method": "<stdout、telegram 或 email>",
    "chatId": "<telegram chat ID，仅 method 为 telegram 时填写>",
    "email": "<邮箱地址，仅 method 为 email 时填写>"
  },
  "onboardingComplete": true
}
CFGEOF
```

然后根据运行平台和投递方式设置定时任务：

**OpenClaw:**

根据用户偏好生成 cron 表达式：
- 每天早上 8 点 → `"0 8 * * *"`
- 每周一早上 9 点 → `"0 9 * * 1"`

**重要：不要使用 `--channel last`。** 用户配置了多个渠道（例如 Telegram 和飞书）时，该选项会失败，因为隔离的 cron 会话没有“上一个渠道”的上下文。务必检测并明确指定渠道和目标。

**步骤 1：检测当前渠道并获取目标 ID。**

用户正在通过某个特定渠道与你对话。询问：“要把每日摘要发到当前这个聊天吗？”

如果用户同意，你需要获取两项信息：**渠道名称**和**目标 ID**。

各渠道目标 ID 的获取方式：

| 渠道 | 目标格式 | 获取方式 |
|---------|--------------|----------------|
| Telegram | 数字 chat ID（例如私聊 `123456789`，群聊 `-1001234567890`） | 运行 `openclaw logs --follow`，发送测试消息并查看 `from.id` 字段。也可运行 `curl "https://api.telegram.org/bot<token>/getUpdates"`，查找 `chat.id`。 |
| Telegram 论坛 | 带话题的群组 ID（例如 `-1001234567890:topic:42`） | 同上，并附上话题 thread ID。 |
| 飞书 | 用户 open_id（例如 `ou_e67df1a850910efb902462aeb87783e5`）或群 chat_id（例如 `oc_xxx`） | 检查 `openclaw pairing list feishu`，或在用户给机器人发消息后查看网关日志。 |
| Discord | 私聊使用 `user:<user_id>`，频道使用 `channel:<channel_id>` | 用户在 Discord 设置中启用开发者模式，然后右键复制 ID。 |
| Slack | `channel:<channel_id>`（例如 `channel:C1234567890`） | 在 Slack 右键点击频道名称，复制链接并提取 ID。 |
| WhatsApp | 带国家/地区代码的电话号码（例如 `+15551234567`） | 由用户提供。 |
| Signal | 电话号码 | 由用户提供。 |

**步骤 2：明确指定渠道和目标，创建 cron 任务。**
```bash
openclaw cron add \
  --name "AI Builders Digest" \
  --cron "<cron expression>" \
  --tz "<user IANA timezone>" \
  --session isolated \
  --message "运行 follow-builders skill：执行 prepare-digest.js，根据 prompts 重混内容并生成摘要，然后通过 deliver.js 投递" \
  --announce \
  --channel <channel name> \
  --to "<target ID>" \
  --exact
```

示例：
```bash
# Telegram 私聊
openclaw cron add --name "AI Builders Digest" --cron "0 8 * * *" --tz "Asia/Shanghai" --session isolated --message "..." --announce --channel telegram --to "123456789" --exact

# 飞书
openclaw cron add --name "AI Builders Digest" --cron "0 8 * * *" --tz "Asia/Shanghai" --session isolated --message "..." --announce --channel feishu --to "ou_e67df1a850910efb902462aeb87783e5" --exact

# Discord 频道
openclaw cron add --name "AI Builders Digest" --cron "0 8 * * *" --tz "America/New_York" --session isolated --message "..." --announce --channel discord --to "channel:1234567890" --exact
```

**步骤 3：立即运行一次，验证 cron 任务有效。**
```bash
openclaw cron list
openclaw cron run <jobId>
```

等待测试运行完成，并确认用户确实在对应渠道收到摘要。如果失败，检查错误信息：
```bash
openclaw cron runs --id <jobId> --limit 1
```

常见错误及解决方式：
- `Channel is required when multiple channels are configured` → 使用了 `--channel last`；改为指定确切渠道。
- `Delivering to X requires target` → 缺少 `--to`；补上目标 ID。
- `No agent` → 如果 OpenClaw 实例配置了多个 Agent，添加 `--agent <agent-id>`。

在验证 cron 投递成功之前，不要继续到欢迎摘要步骤。

**非持久型 Agent + Telegram 或邮件投递：**
使用系统 crontab，确保终端关闭后任务仍会运行：
```bash
SKILL_DIR="<absolute path to the skill directory>"
(crontab -l 2>/dev/null; echo "<cron expression> cd $SKILL_DIR/scripts && node prepare-digest.js 2>/dev/null | node deliver.js 2>/dev/null") | crontab -
```
注意：此方式会运行 prepare 脚本并将输出直接传给投递脚本，完全绕过 Agent。摘要不会由 LLM 重新编排，而是直接投递原始 JSON。若要获得完整的重混摘要，用户应手动输入 `/ai`，或改用 OpenClaw。

**非持久型 Agent + 仅按需获取（不使用 Telegram/邮件）：**
完全跳过 cron 设置。告诉用户：“你选择了按需获取，因此不会设置定时任务。需要摘要时输入 `/ai` 即可。”

### 步骤 9：欢迎摘要

**不要跳过本步骤。** 设置完 cron 任务后，立即为用户生成并发送第一份摘要，让用户了解摘要的实际效果。

告诉用户：“我现在获取今天的内容，并为你生成一份摘要示例，大约需要一分钟。”

不要等待 cron 任务，立即运行下方完整的“内容投递 — 摘要运行”流程。

发送摘要后，征求用户反馈：

“这是你的第一份 AI Builders 摘要！想请你告诉我：
- 长度是否合适？希望更短还是更长？
- 有没有希望我多关注或少关注的内容？
告诉我你的想法，我会相应调整。”

根据用户的设置补充相应的结束语：
- **OpenClaw 或 Telegram/邮件投递：** “下一份摘要会在 [用户选择的时间] 自动送达。”
- **仅按需获取：** “需要下一份摘要时，随时输入 `/ai` 即可。”

等待用户回复，并根据反馈进行调整（必要时更新 config.json 或 prompt 文件），然后确认已完成的更改。

---

## 内容投递 — 摘要运行

在 cron 定时运行或用户输入 `/ai` 时执行此流程。核心任务仍是筛选并总结重要 AI 行业信息；HTML 阅读页是同一份摘要的第二种呈现形式。

### 步骤 1：读取配置

读取 `~/.follow-builders/config.json`，获取用户偏好。

### 步骤 2：运行 prepare 脚本

该脚本会以确定方式处理所有数据获取、feed、prompt 和配置。你**不要自行获取**任何内容。

```bash
cd ${CLAUDE_SKILL_DIR}/scripts && node prepare-digest.js 2>/dev/null
```

脚本会输出一个包含所有所需信息的 JSON 对象：
- `config` — 用户的语言与投递偏好
- `podcasts` — 带完整字幕的播客单集
- `x` — 建造者及其近期帖子（文本、URL、简介）
- `prompts` — 内容重混指引
- `stats` — 单集与帖子的数量统计
- `errors` — 非致命问题（忽略这些问题）

如果脚本完全失败（没有 JSON 输出），告诉用户检查网络连接。否则，使用 JSON 中已有的内容继续。

### 步骤 3：检查是否有内容

如果 `stats.podcastEpisodes` 为 0 **且** `stats.xBuilders` 为 0，告诉用户：“今天关注的建造者没有新动态，明天再来看看吧！”然后停止。

### 步骤 4：重混内容

**你的唯一工作是重混 JSON 中的内容。** 不要从网上获取任何内容、访问任何 URL 或调用 API。所需信息都已包含在 JSON 中。

读取 JSON 中 `prompts` 字段内的指引：
- `prompts.digest_intro` — 整体组织和表达规则
- `prompts.summarize_podcast` — 播客字幕的重混方式
- `prompts.summarize_tweets` — 帖子的重混方式
- `prompts.translate` — 中文翻译方式

**先处理帖子：** `x` 数组包含建造者及其帖子。逐个处理：
1. 根据 `bio` 字段判断其身份（例如简介写有 `ceo @box`，可称为“Box CEO Aaron Levie”）。
2. 使用 `prompts.summarize_tweets` 总结其 `tweets`。
3. 每条帖子都**必须**附上 JSON 中的 `url`。

**再处理播客：** `podcasts` 数组最多包含 1 集播客。如果有内容：
1. 使用 `prompts.summarize_podcast` 总结其 `transcript`。
2. 使用 JSON 对象中的 `name`、`title` 和 `url`，不要从 transcript 中提取这些信息。

按照 `prompts.digest_intro` 组织摘要。

**绝对规则：**
- 绝不编造内容。仅使用 JSON 中的信息。
- 每条内容都**必须**带有 URL。没有 URL 就不要收录。
- 不要猜测职位。依据 `bio` 字段，或只写姓名。
- 不要访问 x.com、搜索网页或调用 API。

### 步骤 5：按偏好生成语言版本

读取 JSON 中的 `config.language`：
- **`"en"`：** 整份摘要使用英文。
- **`"zh"`：** 整份摘要使用中文，并遵循 `prompts.translate`。
- **`"bilingual"`：** 按**段落**交替呈现中英文。每位建造者的帖子先给英文版本，紧接中文翻译，再写下一位建造者。播客也先写英文摘要，再紧接中文翻译。例如：

  ```
  Box CEO Aaron Levie argues that AI agents will reshape software procurement...
  https://x.com/levie/status/123

  Box CEO Aaron Levie 认为 AI agent 将从根本上重塑软件采购...
  https://x.com/levie/status/123

  Replit CEO Amjad Masad launched Agent 4...
  https://x.com/amasad/status/456

  Replit CEO Amjad Masad 发布了 Agent 4...
  https://x.com/amasad/status/456
  ```

  不要先输出所有英文，再统一输出所有中文；必须按段落交替排列。

**严格遵守该语言设置，不要混用语言。**

### 步骤 6：生成 HTML 阅读页

摘要整理完成后，使用同一批已策展内容生成自包含的 HTML 阅读页。它是额外输出，不能取代获取、筛选、总结、翻译或投递摘要的核心流程。

- 保留用户设置的语言（`en`、`zh` 或 `bilingual`），以及摘要中的事实、姓名、日期、数字和来源 URL。生成页面时不得添加新事实，也不要另行独立总结。
- 只收录 feed JSON 中具有有效直达来源 URL 的内容。每条内容的原始链接都必须清晰可见、可点击；不要替换为猜测的链接、主页或频道页。
- 先呈现简短导语和最重要的信号，再按摘要已有分类展示精简卡片。没有内容的分类应省略。
- 将带日期的 AI Builders 阅读页保存为 `ai-builders-digest-YYYY-MM-DD.html`，存入配置的输出目录。若未配置目录，则使用当前项目的 `outputs/`，必要时创建目录。日期按用户设置的时区确定。
- 页面必须自包含并适配手机：使用语义化 HTML 和内嵌 CSS，不加载远程脚本、字体、图片、样式表或追踪器。采用既有的编辑手记风格（暖纸色、细网格/分隔线、墨色文字、克制的蓝/珊瑚/绿色点缀），标题清晰；多个栏目时加入简洁的栏目导航。
- 每个来源链接都必须清晰可见，并通过 `target="_blank"` 和 `rel="noopener noreferrer"` 安全地在新标签页打开。对 feed 提供的文字进行 HTML 转义，并将链接验证为有效的 `http` 或 `https` 直达 URL。
- 报告完成前，确认文件存在且非空，检查生成的 HTML，并确认每条收录内容均附有来源链接。

如果没有新的播客单集和 X 动态，保持现有的“无新内容”处理方式，不要创建空 HTML 页面。

### 步骤 7：投递摘要并提供 HTML 页面

读取 JSON 中的 `config.delivery.method`：

**如果是 `telegram` 或 `email`：**
```bash
echo '<your digest text>' > /tmp/fb-digest.txt
cd ${CLAUDE_SKILL_DIR}/scripts && node deliver.js --file /tmp/fb-digest.txt 2>/dev/null
```
如果投递失败，退回到终端显示摘要。保持现有投递行为；如果当前 Codex 回复可用，也要附上生成的 HTML 文件路径。除非投递脚本确实支持并确认已附加 HTML，否则不要声称已将文件附在邮件或消息中。

**如果是 `stdout`（默认）：**
照常输出摘要，并附上指向生成 HTML 文件的可点击本地链接。

---

## 配置管理

当用户提出修改设置的需求时，按以下规则处理：

### 信息源变更
信息源列表由中心化服务管理，用户不能直接修改。如果用户要求添加或删除来源，告诉他们：“信息源列表由中心化策展并自动更新。如果你想推荐来源，可以在 https://github.com/zarazhangrui/follow-builders 提交 issue。”

### 日程变更
- “改为每周/每日” → 更新 config.json 中的 `frequency`。
- “改到 X 点” → 更新 config.json 中的 `deliveryTime`。
- “把时区改为 X” → 更新 config.json 中的 `timezone`，并同步更新 cron 任务。

### 语言变更
- “切换为中文/英文/双语” → 更新 config.json 中的 `language`。

### 投递方式变更
- “切换到 Telegram/邮件” → 更新 `delivery.method`；必要时引导用户完成设置。
- “更改我的邮箱” → 更新 config.json 中的 `delivery.email`。
- “改为发到当前聊天” → 将 `delivery.method` 设为 `"stdout"`。

### Prompt 定制
如果用户想调整摘要风格，将相应 prompt 文件复制到 `~/.follow-builders/prompts/` 后再编辑。这样用户的定制会持续保留，也不会被中心化更新覆盖。

```bash
mkdir -p ~/.follow-builders/prompts
cp ${CLAUDE_SKILL_DIR}/prompts/<filename>.md ~/.follow-builders/prompts/<filename>.md
```

然后根据用户需求编辑 `~/.follow-builders/prompts/<filename>.md`。

- “摘要写短些/长些” → 编辑 `summarize-podcast.md` 或 `summarize-tweets.md`。
- “多关注 [X]” → 编辑相关 prompt 文件。
- “把语气改为 [X]” → 编辑相关 prompt 文件。
- “恢复默认” → 删除 `~/.follow-builders/prompts/` 中相应的定制文件。

### 信息查询
- “显示我的设置” → 读取 config.json，并以易读格式展示。
- “显示我的信息源”/“我在关注谁？” → 读取用户配置和默认来源列表，列出所有启用的来源。
- “显示我的 prompts” → 读取并展示 prompt 文件。

完成任何配置变更后，向用户确认具体改动。

---

## 手动触发

用户输入 `/ai` 或手动要求摘要时：
1. 跳过 cron 检查，立即运行摘要流程。
2. 使用与 cron 任务相同的“获取 → 重混 → 投递”流程。
3. 告诉用户你正在获取最新内容（通常需要一到两分钟）。
