# AGENTS.md

这个仓库存放你为用户生成的 **HTML 报告**（旅游路线、攻略、总结等）。每份报告会在两个地方被阅读：普通浏览器，以及微信**「GitHub HTML 查看器」小程序**——小程序用 `mp-html` 渲染 HTML，且只内置了 `style` 插件。

## 你的任务

当用户要一份报告时（例如「帮我规划 3 天东京行程」）：

1. 写成**一个自包含的 `.html` 文件**，放在 `reports/` 下，用简短的英文 slug 命名——例如 `reports/tokyo-3-day-trip.html`。
2. 遵循 **[`mp-html-page` 技能](.claude/skills/mp-html-page/SKILL.md)**，确保它能在小程序里正常渲染。要点：
   - 所有 CSS 写在 `<head>` 的 `<style>` 里，且放在被它修饰的内容**之前**。
   - **只用字面值**——不要 `var()`、`@media`、`@keyframes`、`@font-face`、`@import`、`*`、`[attr]`、`:hover` 等伪类。
   - 在包裹所有内容的**外层 `<div>`** 上设置 `background` + `color`（否则小程序会回退到默认的白底黑字）。
   - 不要 `<script>`、`<iframe>`、`<canvas>`、表单、动画或过渡。
   - 图片：用 `<img>` 配 `https://` 链接或同仓库相对路径——**不要**用 `background-image: url()`。用系统字体栈。
3. **commit 并 push** 到 GitHub，这样小程序才能打开它。

`reports/tokyo-3-day-trip.html` 是一个可用的示例——照着它的结构写。

## 在小程序里查看

打开 GitHub HTML 查看器小程序 → 指向这个仓库 → 选择 `reports/` 下报告的路径。
