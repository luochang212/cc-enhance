<div align="right">
  <a title="简体中文" href="README.md"><img src="https://img.shields.io/badge/-%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87-545759?style=for-the-badge" alt="简体中文"></a>
  <a title="English" href="README_en.md"><img src="https://img.shields.io/badge/-English-A31F34?style=for-the-badge" alt="English" /></a>
</div>

# cc-enhance

配置精良的 [Claude Code](https://github.com/anthropics/claude-code) 才能发挥战斗力。cc-enhance 是一项 Claude Code 自提升计划，通过标准 Skill 格式和安装引导，让 Claude Code 逐步增强自己的能力。

## 💻 工作原理

1. Claude Code 读取仓库 → 先读到 `BOOT.md`
2. `BOOT.md` 引导 Claude **展示全部 skill 让用户多选**
3. 对每个选中的 skill → 复制到 `~/.claude/skills/` → 检查该 skill 的「环境依赖」→ 逐项询问是否安装
4. 只有一个 skill（`tool-registry`）会写入用户的 `~/.claude/CLAUDE.md`，且内容精确固定

## 📦 使用方法

在 Claude Code 会话中输入：

```
用 https://github.com/luochang212/cc-enhance 项目提升 Claude Code 能力
```

## 🔧 Skills 清单

| Skill | 功能 | 上下文开销 | 依赖 | 推荐 |
|-------|------|-----------|------|------|
| [tool-registry](skills/tool-registry/SKILL.md) | 工具可用性追踪 | ~300 tokens | 无 | ⭐⭐⭐⭐ 必装 |
| [web-search](skills/web-search/SKILL.md) | 搜索优先级链 | ~800 tokens | Tavily / SearXNG | ⭐⭐⭐ 推荐 |
| [web-fetch](skills/web-fetch/SKILL.md) | 网页提取策略 | ~1000 tokens | Crawl4AI(~300MB) | ⭐⭐ 可选 |
| [agent-skills](skills/agent-skills/SKILL.md) | Skill 管理 | ~600 tokens | npx | ⭐⭐⭐ 推荐 |
| [coding-philosophy](skills/coding-philosophy/SKILL.md) | 编程哲学 | ~800 tokens | 无 | ⭐ 可选 |

## 📁 项目结构

```
cc-enhance/
├── BOOT.md                          # 安装引导入口
├── README.md                        
├── README_en.md                     
└── skills/                          
    ├── tool-registry/               # 工具状态追踪
    │   ├── SKILL.md                 
    │   └── template.json            
    ├── web-search/                  # 网络搜索策略
    │   ├── SKILL.md                 
    │   └── references/
    ├── web-fetch/                   # 网页提取策略
    │   ├── SKILL.md                 
    │   └── references/
    ├── agent-skills/                # Skill 管理
    │   ├── SKILL.md                 
    │   └── references/
    └── coding-philosophy/           # 编程哲学
        ├── SKILL.md                 
        └── references/
```

> 每个 skill 的 `SKILL.md` 是 Claude Code 标准格式（frontmatter + 内容），`references/` 放技术细节。

## 📜 开源协议

[MIT](LICENSE)
