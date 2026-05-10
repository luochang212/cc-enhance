# Skills 市场

## skills.sh

- **CLI**: `npx skills add owner/repo`
- **搜索**: 让 Claude 搜索 GitHub：`site:github.com <功能> skill SKILL.md Claude Code`
- **下载**: `npx skills add stared/vibe-commit`（实测通过 ✅）
- **存放**: `.agents/skills/<name>/`，symlink 到 55 个 agent
- **文件**: SKILL.md + 附属脚本（如 `ai-blame.py`）
- **机制**: skills.sh 是索引站，列出 GitHub 上的 skill 仓库。分发由 `npx skills` CLI 完成（clone 仓库 → 发现 SKILL.md → 安装 + symlink）

实测示例：
```bash
$ npx skills add stared/vibe-commit
◇  Found 1 skill
●  Skill: commit
│  Commit with user prompts from this conversation.
◇  55 agents
●  Installing to: Antigravity, Claude Code, OpenClaw, Codex, Cursor, ...
✓  ~/project/.agents/skills/commit
```

## LobeHub Skills Marketplace

- **CLI**: `@lobehub/market-cli`
- **规模**: 296K+
- **首次使用**: 需注册身份
  ```bash
  npx -y @lobehub/market-cli register \
    --name "<name>" --description "<desc>" --source claude-code
  ```
  凭证: `~/.lobehub-market/credentials.json`
- **搜索**: 
  ```bash
  npx -y @lobehub/market-cli skills search --q "<关键词>"
  ```
  返回 stars + installs 数，是质量筛选核心信号
- **下载**: 
  ```bash
  npx -y @lobehub/market-cli skills install github-awesome-copilot-git-commit --agent claude-code
  # → Installed to .claude/skills/github-awesome-copilot-git-commit
  ```
- **存放**: `.claude/skills/<name>/`（直接安装到 Claude Code 目录）
- **评分**: 
  ```bash
  npx -y @lobehub/market-cli skills rate <id> --score 4
  npx -y @lobehub/market-cli skills comment <id> -c "评价"
  ```

实测示例：
```
$ npx @lobehub/market-cli skills search --q "git commit"
github-awesome-copilot-git-commit   git-commit   23.6k stars  88 installs
davila7-claude-code-templates-git-commit-helper  Git Commit Helper  22.1k stars  3 installs
```

## ModelScope Skills 中心

- **CLI**: `npx skills add <url>`
- **搜索**: 网站浏览（modelscope.cn/skills），分类：开发工具、前端开发、媒体处理、代码质检、云效工具等
- **下载**: 
  ```bash
  npx skills add https://modelscope.cn/skills/@Alipay/alipay-payment-integration
  ```
  实测通过 ✅
- **存放**: `.agents/skills/<name>/`，symlink 到 Claude Code
- **文件**: SKILL.md + README.md + references/（实测 4 个文件）

实测示例：
```
$ npx skills add https://modelscope.cn/skills/@Alipay/alipay-payment-integration
◇  Found 1 skill
●  Skill: alipay-payment-integration
│  支付宝开放平台支付产品接入最佳实践
│  Files: SKILL.md, README.md, references/checklist.md, references/product-decision.md
✓  ~/project/.agents/skills/alipay-payment-integration
```

## SkillHub

- **CLI**: `curl -fsSL https://skillhub.cn/install/install.sh | bash`
- **规模**: 68K+
- **搜索**: 网站浏览（skillhub.cn）
- **下载**: 脚本从腾讯 COS 下载 `latest.tar.gz` → 解压 → 执行内部 `cli/install.sh`
  ```bash
  curl -fsSL https://skillhub.cn/install/install.sh | bash
  ```
- **存放**: 取决于内部安装脚本，需审查后确认
- **注意**: 非标准格式。安装脚本只有 483 字节（引导脚本），实际逻辑在 COS 的 tar.gz 中。执行前应审查
- **状态**: ⚠️ 仅审查了引导脚本，未执行

## `npx skills` 通用命令

```bash
npx skills add <来源>       # 安装（GitHub repo / URL）
npx skills find             # 交互搜索
npx skills list             # 列出已安装
npx skills update           # 更新
npx skills remove <name>    # 删除
```
