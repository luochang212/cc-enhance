# BOOT

分六个阶段安装 cc-enhance skills。每个阶段用 `AskUserQuestion` 交互，不要文字让用户输入。

## 阶段 0：获取仓库

下载 zip 到 /tmp（比逐个 curl 快得多）：

```bash
curl -sL -o /tmp/cc-enhance.zip https://github.com/luochang212/cc-enhance/archive/refs/heads/main.zip
unzip -qo /tmp/cc-enhance.zip -d /tmp
```

解压后路径为 `/tmp/cc-enhance-main/`，后续所有 `skills/` 操作以此为基础。

## 阶段 1：选择 skills

用 `AskUserQuestion`（multiSelect）让用户选择。options:

| label | description |
|-------|-------------|
| tool-registry-cce | 工具可用性追踪，跨会话记住工具状态。~300tokens。推荐。 |
| web-search-cce | 搜索优先级链（Tavily/SearXNG→web_search_prime→DuckDuckGo），同层并发失败降级。~800tokens。 |
| web-fetch-cce | 网页提取三层策略（WebFetch→Crawl4AI→Playwright），含中国站点实测。~1000tokens。 |
| agent-skills-cce | Skill 市场导航、安装、管理。~600tokens。 |
| coding-philosophy-cce | 开发（熵增）+ 审查（熵减）决策框架。~800tokens。 |

question 和 header 自行拟定。用户可能全选或部分选。

## 阶段 2：复制文件 + 收集依赖

对每个选中的 skill：

1. 从 `/tmp/cc-enhance-main/skills/<name>/` 递归复制到 `~/.claude/skills/<name>/`（源仓库已含 `-cce` 后缀，无需额外改名）
2. 读 SKILL.md 末尾「环境依赖」表，逐项检查，区分「已满足」和「缺失」

全部 skill 处理完后，汇总展示：
- 哪些 skill 已复制
- 所有缺失依赖（跨 skill 合并），标注用途、安装方式、磁盘开销

## 阶段 3：安装依赖

如果阶段 2 没有缺失依赖 → 跳过本阶段。

否则，用 `AskUserQuestion`（multiSelect）一次性让用户选择要安装哪些依赖。options 列所有缺失依赖，description 写清用途和开销。用户勾选的 → 执行安装；未勾选的 → 跳过。

安装过程中如遇错误，报告但不中断后续安装。

## 阶段 4：生成 tool-registry.json

仅当用户选了 `tool-registry`。**不要直接复制 catalog，要扫描本机后动态生成。**

### 4.1 读取 catalog

读 `/tmp/cc-enhance-main/skills/tool-registry-cce/references/tool-catalog.json`。这是常用工具的参考样本，分 `mcp` / `cli` / `api` 三组，每个条目有 `invoke` 和 `notes`。catalog 未覆盖的工具同样需要写入注册表——自行推断 invoke 即可。

### 4.2 扫描本机

逐一核查 catalog 中每个工具在本机是否存在：

- **MCP 工具**：读 `~/.claude/settings.json`（和 `~/.claude/settings.local.json`），检查 `mcpServers` 或 `mcpServersFromPlugins` 中有没有对应条目。用 catalog 中该工具的 `invoke` 前缀（如 `mcp__web-search-prime__`）去匹配。
- **CLI 工具**：检查对应的环境变量（`$TAVILY_API_KEY`）或可执行文件（`which docker`、`python3 -c "import crawl4ai"` 等）。同时检查阶段 3 是否刚刚安装过。
- **API 端点**：不做网络请求验证，直接标记 `not_verified`。

### 4.3 生成注册表

读取已有的 `~/.claude/tool-registry.json`（如果存在）保留旧记录。

对每个**本机检测到的**工具：
- 已有记录 → 保留其 `status` / `updated` / `history`
- 没有记录 → 新增条目，从 catalog 取 `invoke` 和 `notes`，`status` 设为 `not_verified`，`updated` 设为 `null`

对**本机未检测到的**工具 → 不写入注册表（用户没装，不需要追踪）。

写入 `~/.claude/tool-registry.json`。

### 4.4 CLAUDE.md 注入

用 `AskUserQuestion`（非 multiSelect）问是否允许修改 `~/.claude/CLAUDE.md`。options: 允许 / 拒绝。用户选「允许」才写入：

```markdown
## 工具可用性

查询 `~/.claude/tool-registry.json` 决定工具策略：
- `updated` ≤ 3 天 → 信任，直接用
- `updated` 3-7 天 → 信任但首次调用时观察
- `updated` > 7 天或不存在 → 用轻量调用重新验证

工具调用失败时即刻更新注册表。
```

写入规则：已有该章节则替换整节，否则末尾追加。不在此之外写任何内容。

## 阶段 5：完成

简短总结：已安装 skills、已安装/未安装的依赖、CLAUDE.md 是否更新、下一步建议（注册 Tavily、启动 SearXNG 等）。
