---
name: web-search
description: 网络搜索策略。当需要搜索网络信息、查询最新资料、查找技术文档、或选择搜索工具时，必须使用此技能。定义了搜索工具优先级链（Tavily/SearXNG → web_search_prime → DuckDuckGo），同层并发、失败即降级。
---

# 网络搜索

## 核心原则

1. **先读注册表**。`~/.claude/tool-registry.json` 记录了每个工具的实测状态。
2. **同层并发**。独立通道同时发起，谁先返回用谁。
3. **失败即降级，不重试**。一个工具失败一次就跳到下一层。

## 优先级链

### Tier 0 — 直接 URL

用户给了 URL 或你知道目标地址 → 跳到 [web-fetch](../web-fetch/SKILL.md) 直接读，不搜索。

### Tier 1 — Tavily / SearXNG（并列）

**Tavily** — ~0.6s 响应，JSON 结构化返回，不限频，英中文均可。1000 次/月。

→ [references/tavily.md](references/tavily.md)

**SearXNG** — 自部署 Docker，零额度限制，~3s 响应，中文 18 条/Tavily 5 条。

→ [references/searxng.md](references/searxng.md)

### Tier 2 — web_search_prime MCP

域名过滤、时效性、内容长度控制优于 Tavily。有月额度，备选。

→ [references/web-search-prime.md](references/web-search-prime.md)

### Tier 3 — DuckDuckGo（兜底）

纯免费但 ~2s、HTML 需解析、限频。降级前告知用户。

→ [references/duckduckgo.md](references/duckduckgo.md)

## 降级流程

```
需要搜索
  ├─ Tavily curl (Bash)      ──失败──┐
  ├─ SearXNG (本地 Docker)    ──失败──┤
  │                                   ▼
  ├─ web_search_prime MCP     ──失败──┐
  │                                   ▼
  └─ DuckDuckGo WebFetch       ── 兜底
```

搜索拿到 URL 后，进入 [web-fetch](../web-fetch/SKILL.md) 提取页面内容。

## 维护

工具状态变更时更新 `~/.claude/tool-registry.json`。

## 环境依赖

| 依赖 | 用途 | 安装方式 | 磁盘 | 必需 |
|------|------|----------|------|------|
| Tavily API Key | 首选搜索，~0.6s，1000次/月 | 注册 [tavily.com](https://tavily.com) → `export TAVILY_API_KEY=xxx` | 0 | 否 |
| Docker + SearXNG | 无限额度本地搜索，中文强 | 安装 Docker → `docker run -d -p 8080:8080 searxng/searxng` | ~1.5GB | 否 |
| 无（DuckDuckGo） | 兜底方案，零安装始终可用 | — | 0 | 否 |
