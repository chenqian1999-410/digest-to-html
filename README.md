# 摘要转 HTML 阅读页

`digest-to-html` 是一个面向 Codex 的 Skill，可将已有的 AI Builders 或其他主题摘要整理成清晰、适合手机阅读的 HTML 页面。

## 能做什么

- 将摘要按主题整理成易浏览的栏目和条目卡片。
- 生成自包含的 HTML 阅读页，使用内嵌样式，不依赖远程脚本、字体或样式表。
- 采用暖纸色与克制的编辑手记视觉风格，并适配手机屏幕。
- 保留每条内容的原始来源链接，方便读者查看上下文与出处。

## 使用方式

在 Codex 中调用 `$digest-to-html`，并提供要整理的摘要内容或文件。例如：

> 使用 `$digest-to-html`，把这份摘要制作成适合手机阅读的 HTML 页面。

如果项目中有 `outputs/` 目录，生成的页面会优先放在那里。带日期的 AI Builders 摘要默认使用 `ai-builders-digest-YYYY-MM-DD.html` 命名。

## 内容原则

这个 Skill 负责呈现和组织已有内容，不会自行抓取 feed 或补写未经来源支持的信息。使用 feed JSON 时，只依据 JSON 中的内容；没有有效直达来源链接的条目会跳过，不会用猜测的链接替代。

## 文件结构

```text
├── SKILL.md            # Skill 的任务说明与工作流程
└── agents/openai.yaml  # Codex 界面显示名称与调用提示
```
