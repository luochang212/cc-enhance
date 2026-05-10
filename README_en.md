<div align="right">
  <a title="简体中文" href="README.md"><img src="https://img.shields.io/badge/-%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87-545759?style=for-the-badge" alt="简体中文"></a>
  <a title="English" href="README_en.md"><img src="https://img.shields.io/badge/-English-A31F34?style=for-the-badge" alt="English" /></a>
</div>

# cc-enhance

A well-configured [Claude Code](https://github.com/anthropics/claude-code) performs at its best. cc-enhance is a self-enhancement plan — it teaches Claude Code how to improve itself through standard Skills and an interactive installation wizard.

## 💻 How It Works

1. Claude Code reads this repo → finds `BOOT.md` first
2. `BOOT.md` guides Claude to **present all skills for the user to multi-select**
3. For each selected skill → copied to `~/.claude/skills/` → checks that skill's dependencies → asks whether to install each
4. Only one skill (`tool-registry`) writes to the user's `~/.claude/CLAUDE.md`, with precise, fixed content

## 📦 Usage

From any Claude Code session, type:

```
Help me enhance Claude Code with https://github.com/luochang212/cc-enhance
```

## 🔧 Skills

| Skill | Function | Context Cost | Dependencies | Recommendation |
|-------|------|-----------|------|------|
| [tool-registry](skills/tool-registry/SKILL.md) | Tool availability tracking | ~300 tokens | None | ⭐⭐⭐ Essential |
| [web-search](skills/web-search/SKILL.md) | Search priority chain | ~800 tokens | Tavily / SearXNG | ⭐⭐⭐ Recommended |
| [web-fetch](skills/web-fetch/SKILL.md) | Web content extraction | ~1000 tokens | Crawl4AI (~300MB) | ⭐⭐ Optional |
| [agent-skills](skills/agent-skills/SKILL.md) | Skill management | ~600 tokens | npx | ⭐⭐⭐ Recommended |
| [coding-philosophy](skills/coding-philosophy/SKILL.md) | Coding philosophy | ~800 tokens | None | ⭐ Optional |

## 📁 Project Structure

```
cc-enhance/
├── BOOT.md                          # Installation wizard entry point
├── README.md                        
├── README_en.md                     
└── skills/                          
    ├── tool-registry/               # Tool state tracking
    │   ├── SKILL.md                 
    │   └── template.json            
    ├── web-search/                  # Search strategies
    │   ├── SKILL.md                 
    │   └── references/
    ├── web-fetch/                   # Content extraction
    │   ├── SKILL.md                 
    │   └── references/
    ├── agent-skills/                # Skill management
    │   ├── SKILL.md                 
    │   └── references/
    └── coding-philosophy/           # Coding principles
        ├── SKILL.md                 
        └── references/
```

> Each skill's `SKILL.md` follows Claude Code standard format (frontmatter + content). `references/` holds technical details.

## 📜 License

[MIT](LICENSE)
