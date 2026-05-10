<div align="right">
  <a title="简体中文" href="README.md"><img src="https://img.shields.io/badge/-%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87-545759?style=for-the-badge" alt="简体中文"></a>
  <a title="English" href="README_en.md"><img src="https://img.shields.io/badge/-English-A31F34?style=for-the-badge" alt="English" /></a>
</div>

# cc-enhance

This project lets [Claude Code](https://github.com/anthropics/claude-code) configure itself.

> cc-enhance is a Claude Code capability enhancement plan. Through a preset installation wizard, it enhances Claude Code's web search, web fetching, and skill downloading capabilities.

## 💻 How It Works

1. Claude Code reads this repo → reads `BOOT.md` first
2. `BOOT.md` guides Claude to **present all skills for the user to multi-select**
3. For each selected skill → copied to `~/.claude/skills/` → checks that skill's dependencies → asks whether to install each

## 🚀 Usage

From any Claude Code session, type:

```
Help me enhance Claude Code with https://github.com/luochang212/cc-enhance
```

## 🔧 Skills

| Skill | Function | Context Cost | Dependencies | Recommendation |
|-------|------|-----------|------|------|
| [tool-registry-cce](skills/tool-registry-cce/SKILL.md) | Tool availability tracking | ~300 tokens | None | ⭐⭐⭐ Recommended |
| [web-search-cce](skills/web-search-cce/SKILL.md) | Search priority chain | ~800 tokens | Tavily / SearXNG | ⭐⭐⭐ Recommended |
| [web-fetch-cce](skills/web-fetch-cce/SKILL.md) | Web content extraction | ~1000 tokens | Crawl4AI (~300MB) | ⭐⭐ Optional |
| [agent-skills-cce](skills/agent-skills-cce/SKILL.md) | Skill management | ~600 tokens | npx | ⭐⭐⭐ Recommended |
| [coding-philosophy-cce](skills/coding-philosophy-cce/SKILL.md) | Coding philosophy | ~800 tokens | None | ⭐ Optional |

## 📁 Project Structure

```
cc-enhance/
├── BOOT.md                          # Installation wizard entry point
├── README.md                        
├── README_en.md                     
└── skills/
    ├── tool-registry-cce/           # Tool state tracking
    │   ├── SKILL.md
    │   └── references/
    │       └── tool-catalog.json
    ├── web-search-cce/              # Search strategies
    │   ├── SKILL.md
    │   └── references/
    ├── web-fetch-cce/               # Content extraction
    │   ├── SKILL.md
    │   └── references/
    ├── agent-skills-cce/            # Skill management
    │   ├── SKILL.md
    │   └── references/
    └── coding-philosophy-cce/       # Coding principles
        ├── SKILL.md
        └── references/
```

## 🤝 Contributing

We welcome all forms of contribution!

- 🐛 Bug Reports — Found an issue? Submit an Issue
- 💡 Feature Suggestions — Have a good idea? Let us know
- 📝 Content Improvements — Help refine the documentation
- 🔧 Skill Enhancements — Submit a Pull Request

## 📜 License

[MIT](LICENSE)
