# wechat-reports

一个用来生成**可以在手机上阅读的 HTML 报告**的起始仓库，配合微信「GH HTML 查看器」小程序查看。

你在这个仓库里跟 AI 编程助手对话（「帮我规划 3 天东京行程」「把这些笔记整理成一页」），它会在 `reports/` 下写一个自包含的 `.html` 报告，然后 commit 并 push。因为报告对小程序用来渲染 HTML 的 [mp-html](https://github.com/jin-yufeng/mp-html) 组件是安全的，所以在浏览器和小程序里都能正确显示。

## 工作原理

- **[AGENTS.md](AGENTS.md)** 告诉 AI 怎么写报告。
- **[.claude/skills/mp-html-page/SKILL.md](.claude/skills/mp-html-page/SKILL.md)** 是让 HTML 留在小程序 `style` 插件能力范围内的技能。它是项目级技能，所以这个仓库里的 Claude / Copilot 助手会自动加载。
- **`reports/`** 存放生成的页面。[reports/tokyo-3-day-trip.html](reports/tokyo-3-day-trip.html) 是一个示例。

## 怎么用

1. **Clone 这个仓库**，建议 push 到你自己的 GitHub 账号并设为 **private**，这样报告不会公开。
2. 用 **[wechat-acp](https://github.com/formulahendry/wechat-acp)** 把微信和本机的 AI agent 连起来，`--cwd` 指向你 clone 下来的这个仓库，让 agent「在这个项目里」工作：

   ```
   npx -y wechat-acp@latest --agent copilot --cwd <你clone的仓库路径>
   ```

   首次运行会在终端显示二维码，用微信扫码登录。
3. 在微信里给 bot 发消息让它写报告，例如「帮我规划 3 天东京行程」。它会按 [AGENTS.md](AGENTS.md) 在 `reports/` 下生成 HTML 并自动 commit、push。
4. 在「GH HTML 查看器」小程序里指向这个仓库，打开 `reports/` 下的报告。

   小程序需要一个 **GitHub Fine-grained PAT** 才能访问仓库（尤其是 private 仓库）。生成步骤：
   - 登录 GitHub → 右上角头像 → **Settings** → 左侧 **Developer settings** → **Personal access tokens** → **Fine-grained tokens**
   - 点 **Generate new token**，**Resource owner** 选自己，**Repository access** 选「Only select repositories」并选中这个仓库，**Permissions** 里把 **Contents** 设为 `Read-only`，设好有效期，生成后**立即复制**（只显示一次）。
   - 在小程序设置里粘贴这个 token 即可。
