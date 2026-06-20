# wechat-reports · `llm-wiki` 分支

这个分支把仓库当成一个**个人知识库（LLM Wiki）**：你往 `raw/` 丢原始资料，AI 读完后在 `wiki/` 里维护一套互相关联的 HTML 页面，配合微信「GH HTML 查看器」小程序在手机上浏览。

模式来自 [Karpathy 的 LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)，关键改动是 `wiki/` 里是 **HTML**（用 mp-html-page 技能生成），而不是 Karpathy 原版的 Markdown，所以能直接在小程序里看。

> 想写**一次性报告**（旅游路线、攻略、总结等）？切回 `main` 分支。

## 三层结构

- **`raw/`** — 你收集的原始资料（文章、笔记、PDF、图片）。只读，是事实来源。
- **`wiki/`** — AI 生成并维护的 HTML 页面：摘要、概念 / 主题页、总览。[wiki/index.html](wiki/index.html) 是入口目录，[wiki/log.html](wiki/log.html) 是时间线日志。
- **[AGENTS.md](AGENTS.md)** — 告诉 AI 知识库怎么组织、怎么录入资料、怎么回答和体检。
- **[.claude/skills/mp-html-page/SKILL.md](.claude/skills/mp-html-page/SKILL.md)** — 让 HTML 留在小程序 `style` 插件能力范围内的项目级技能，会被自动加载。

## 怎么用

1. **Clone 这个仓库并切到 `llm-wiki` 分支**，建议 push 到你自己的 GitHub 账号并设为 **private**，这样知识库不会公开。
2. 用 **[wechat-acp](https://github.com/formulahendry/wechat-acp)** 把微信和本机的 AI agent 连起来，`--cwd` 指向你 clone 的仓库：

   ```
   npx -y wechat-acp@latest --agent copilot --cwd <你clone的仓库路径>
   ```

   首次运行会在终端显示二维码，用微信扫码登录。
3. 把资料放进 `raw/`，在微信里让 bot「录入这条资料」。它会按 [AGENTS.md](AGENTS.md) 写摘要页、更新索引和相关主题页、追加日志，然后自动 commit、push。
4. 想查东西就直接提问，它会读知识库、给出带引用的回答，必要时把好答案沉淀成新页面。
5. 在「GH HTML 查看器」小程序里指向这个仓库，打开 `wiki/index.html` 作为入口浏览，再按页面里列出的路径打开具体页面。

   小程序需要一个 **GitHub Fine-grained PAT** 才能访问仓库（尤其是 private 仓库）。小程序是纯前端应用，token 只存储在当前设备本地，不会上传到任何服务器。生成步骤：
   - 登录 GitHub → 右上角头像 → **Settings** → 左侧 **Developer settings** → **Personal access tokens** → **Fine-grained tokens**
   - 点 **Generate new token**，**Resource owner** 选自己，**Repository access** 选「Only select repositories」并选中这个仓库，**Permissions** 里把 **Contents** 设为 `Read-only`，设好有效期，生成后**立即复制**（只显示一次）。
   - 在小程序设置里粘贴这个 token 即可。
