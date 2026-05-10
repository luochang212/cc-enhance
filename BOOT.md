# BOOT — cc-enhance 安装引导

按以下流程逐项引导用户配置 Claude Code。**每步都要问，不要替用户做决定。**

---

## 阶段 1：选择 Skills

一次性展示所有 skill，让用户**多选**。

### Skill 清单

| # | Skill | 功能 | Token | 推荐 |
|---|-------|------|-------|------|
| 1 | `tool-registry` | 工具可用性追踪，跨会话记住工具状态 | ~300 | ⭐⭐⭐ 必装 |
| 2 | `web-search` | 搜索优先级链，同层并发、失败降级 | ~800 | ⭐⭐⭐ 推荐 |
| 3 | `web-fetch` | 网页提取三层策略，含中国站点实测 | ~1000 | ⭐⭐ 可选 |
| 4 | `agent-skills` | Skill 市场导航、安装命令、管理纪律 | ~600 | ⭐⭐ 推荐 |
| 5 | `coding-philosophy` | 开发（熵增）+ 审查（熵减）决策框架 | ~800 | ⭐ 可选 |

展示后问用户：「你想安装哪些？」（可以用序号多选，如「1,2,4」）

---

## 阶段 2：安装已选择的 Skill

按序号依次处理用户选择的 skill。对每个选中的 skill：

### 2.1 复制 Skill

```bash
cp -r skills/<name> ~/.claude/skills/<name>
```

### 2.2 处理环境依赖

读该 skill 末尾的「环境依赖」表，逐条检查：

- **已满足** → 跳过
- **未满足** → 告知用途、安装方式、磁盘占用 → 问：「要不要装/配置？」
  - 用户同意 → 执行安装
  - 用户拒绝 → 跳过（非必需依赖缺失不影响基本功能）

### 2.3 tool-registry 额外步骤

如果用户选了 `tool-registry`，额外处理：

1. 检查 `~/.claude/tool-registry.json` 是否存在
   - 不存在 → 复制 `skills/tool-registry/template.json` → `~/.claude/tool-registry.json`
   - 已存在 → 不覆盖

2. 向用户 `~/.claude/CLAUDE.md` 写入工具可用性块。**严格使用以下文本，一个字不改**：

```markdown
## 工具可用性

查询 `~/.claude/tool-registry.json` 决定工具策略：
- `updated` ≤ 3 天 → 信任，直接用
- `updated` 3-7 天 → 信任但首次调用时观察
- `updated` > 7 天或不存在 → 用轻量调用重新验证

工具调用失败时即刻更新注册表。
```

写入逻辑：
- 如果 CLAUDE.md 已有 `## 工具可用性` 章节 → 替换整节
- 如果没有 → 在末尾追加
- **绝不在此之外写入任何其他内容**

---

## 阶段 3：完成

安装完成后告知用户：

- 哪些 skill 已安装
- 哪些依赖已安装/未安装
- CLAUDE.md 是否有更新
- 下一步可以做什么（例如注册 Tavily、启动 SearXNG Docker）
