# 推荐 Skills

安装命令优先使用 `npx skills add`，`-g` 全局安装，`-y` 跳过确认。Claude Code 用户安装 `obra/superpowers` 时优先用内置 plugin 命令。

安装前先查看仓库有哪些 skill：
```bash
npx skills add owner/repo --list
```

## 官方 & 平台

### anthropics/skills

官方 16 个：docx、pdf、pptx、xlsx、frontend-design、mcp-builder、skill-creator 等。

```bash
npx skills add https://github.com/anthropics/skills \
  --skill docx pdf pptx xlsx \
    algorithmic-art brand-guidelines canvas-design \
    frontend-design internal-comms mcp-builder \
    skill-creator slack-gif-creator theme-factory \
    web-artifacts-builder webapp-testing \
  --agent claude-code -g -y
```

### vercel-labs/skills

```bash
npx skills add https://github.com/vercel-labs/skills \
  --skill find-skills \
  --agent claude-code -g -y
```

### vercel-labs/agent-skills

React/Next.js 最佳实践、组合模式、性能优化。

```bash
npx skills add https://github.com/vercel-labs/agent-skills \
  --skill '*' \
  --agent claude-code -g -y
```

### vercel-labs/next-skills

Next.js 专项：RSC 模式、缓存策略、版本升级。

```bash
npx skills add https://github.com/vercel-labs/next-skills \
  --skill next-best-practices next-cache-components next-upgrade \
  --agent claude-code -g -y
```

### openai/skills

```bash
npx skills add https://github.com/openai/skills \
  --skill cloudflare-deploy gh-fix-ci jupyter-notebook \
    screenshot security-best-practices \
    security-ownership-map security-threat-model \
  --agent claude-code -g -y
```

### microsoft/skills

```bash
npx skills add https://github.com/microsoft/skills \
  --skill fastapi-router-py frontend-design-review \
    frontend-ui-dark-ts wiki-agents-md \
    wiki-changelog wiki-page-writer wiki-qa wiki-researcher \
  --agent claude-code -g -y
```

### google-gemini/gemini-cli

```bash
npx skills add https://github.com/google-gemini/gemini-cli \
  --skill code-reviewer pr-creator ci behavioral-evals \
    docs-writer review-duplication string-reviewer \
  --agent claude-code -g -y
```

## 框架专项

### flutter/skills

Flutter 开发全套：测试、架构、布局、路由、国际化、动画、原生互操作。

```bash
npx skills add https://github.com/flutter/skills \
  --skill '*' \
  --agent claude-code -g -y
```

### langchain-ai/langchain-skills

LangChain/LangGraph/Deep Agents 全套。

```bash
npx skills add https://github.com/langchain-ai/langchain-skills \
  --skill '*' \
  --agent claude-code -g -y
```

## 工具 & 元技能

### vercel-labs/agent-browser

浏览器自动化，Playwright 封装。

```bash
npx skills add https://github.com/vercel-labs/agent-browser \
  --skill agent-browser vercel-sandbox \
  --agent claude-code -g -y
```

### obra/superpowers

元技能框架：brainstorming、executing-plans、subagent-driven-development、verification-before-completion。

**Claude Code**——优先使用内置 plugin 命令安装：

```
/plugin install superpowers@superpowers-marketplace
```

**其他 agent**——用 `npx skills`：

```bash
npx skills add https://github.com/obra/superpowers \
  --skill '*' \
  --agent claude-code -g -y
```

## 精选合集

### ComposioHQ/awesome-claude-skills

```bash
npx skills add ComposioHQ/awesome-claude-skills \
  --skill connect changelog-generator \
  --agent claude-code -g -y
```

### agentbay-ai/agentbay-skills

中国股市分析、股票监控、微博热搜。

```bash
npx skills add https://github.com/agentbay-ai/agentbay-skills \
  --skill china-stock-analysis stock-watcher weibo-hot-search \
  --agent claude-code -g -y
```
