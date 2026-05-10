<div align="right">
  <a title="简体中文" href="README.md"><img src="https://img.shields.io/badge/-%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87-545759?style=for-the-badge" alt="简体中文"></a>
  <a title="English" href="README_en.md"><img src="https://img.shields.io/badge/-English-A31F34?style=for-the-badge" alt="English" /></a>
</div>

# cc-enhance

本项目致力于让 [Claude Code](https://github.com/anthropics/claude-code) 自己配置自己。

> **cc-enhance** 是一项 Claude Code 能力提升计划。通过预设的安装引导程序，增强 Claude Code 的网络搜索、网页获取、技能下载等能力。

## 💻 工作原理

1. Claude Code 读取本仓库 → 先读 `BOOT.md`
2. `BOOT.md` 引导 Claude 展示全部 skill 让用户多选
3. 对每个选中的 skill → 复制到 `~/.claude/skills/` → 检查该 skill 的「环境依赖」→ 逐项询问是否安装

## 🚀 使用方法

在 Claude Code 会话中输入：

```
用 https://github.com/luochang212/cc-enhance 提升 Claude Code 能力
```

## 🔧 Skills 清单

| Skill | 功能 | 上下文开销 | 依赖 | 推荐 |
|-------|------|-----------|------|------|
| [tool-registry-cce](skills/tool-registry-cce/SKILL.md) | 工具可用性追踪 | ~300 tokens | 无 | ⭐⭐⭐ 推荐 |
| [web-search-cce](skills/web-search-cce/SKILL.md) | 搜索优先级链 | ~800 tokens | Tavily / SearXNG | ⭐⭐⭐ 推荐 |
| [web-fetch-cce](skills/web-fetch-cce/SKILL.md) | 网页提取策略 | ~1000 tokens | Crawl4AI(~300MB) | ⭐⭐ 可选 |
| [agent-skills-cce](skills/agent-skills-cce/SKILL.md) | Skill 管理 | ~600 tokens | npx | ⭐⭐⭐ 推荐 |
| [coding-philosophy-cce](skills/coding-philosophy-cce/SKILL.md) | 编程哲学 | ~800 tokens | 无 | ⭐ 可选 |

## 📁 项目结构

```
cc-enhance/
├── BOOT.md                          # 安装引导入口
├── README.md                        
├── README_en.md                     
└── skills/
    ├── tool-registry-cce/           # 工具状态追踪
    │   ├── SKILL.md
    │   └── references/
    │       └── tool-catalog.json
    ├── web-search-cce/              # 网络搜索策略
    │   ├── SKILL.md
    │   └── references/
    ├── web-fetch-cce/               # 网页提取策略
    │   ├── SKILL.md
    │   └── references/
    ├── agent-skills-cce/            # Skill 管理
    │   ├── SKILL.md
    │   └── references/
    └── coding-philosophy-cce/       # 编程哲学
        ├── SKILL.md
        └── references/
```

## 🤝 如何贡献

我们欢迎任何形式的贡献！

- 🐛 报告 Bug - 发现问题请提交 Issue
- 💡 功能建议 - 有好想法就告诉我们
- 📝 内容完善 - 帮助改进内容
- 🔧 技能优化 - 提交 Pull Request

## 📜 开源协议

[MIT](LICENSE)
