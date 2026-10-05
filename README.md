# AI Builders 摘要与 HTML 阅读页

`follow-builders` 是一个 Codex Skill：从中心化 feed 获取 AI 建造者的最新动态，筛选值得关注的产品、工程、Agent 与 AI 安全信号，按偏好生成摘要，并额外制作一份适合手机阅读的 HTML 页面。

## 功能

- 自动获取中心化 feed 中的 X 帖子和 YouTube 播客内容，无需为内容抓取配置 API key。
- 按用户选择生成中文、英文或中英双语摘要。
- 保留每条收录内容的原始直达来源链接；没有有效链接的内容不收录。
- 生成自包含、响应式 HTML 阅读页，采用暖纸色的编辑手记风格，并提供栏目导航。
- 支持按需运行或配置定时任务；可按设置投递至 stdout、Telegram 或邮件。

HTML 页面是同一份已筛选、总结的摘要的附加呈现，不会取代内容获取与行业信息策展。Skill 只依据 feed JSON，不自行搜索或补写 feed 中没有的信息。

## 在 Codex 中使用

安装 Skill 后可输入 `$follow-builders` 或 `/ai` 获取最新摘要。首次运行时，Skill 会询问频率、时区、语言和投递方式；之后也可以通过对话修改这些偏好。

带日期的 HTML 文件命名为 `ai-builders-digest-YYYY-MM-DD.html`，默认保存在当前项目的 `outputs/` 目录。

## 文件结构

```text
├── SKILL.md            # 完整的中文引导、摘要、HTML 与投递流程
├── README.md           # 项目介绍
└── agents/openai.yaml  # Codex 界面名称与调用提示
```
