# BOOT

分九个阶段安装 cc-enhance skills。每个阶段用 `AskUserQuestion` 交互，不要让用户手动输入文字。

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
| web-search-cce | 搜索优先级链（Tavily→web_search_prime→DuckDuckGo），同层并发失败降级。~800tokens。 |
| web-fetch-cce | 网页提取三层策略（WebFetch→Crawl4AI→Playwright），含中国站点实测。~1000tokens。 |
| agent-skills-cce | Skill 市场导航、安装、管理。~600tokens。 |

question 和 header 自行拟定。用户可能全选或部分选。

## 阶段 2：推荐插件

参考 `skills/agent-skills-cce/references/recommended.md` § obra/superpowers 和 § affaan-m/everything-claude-code。

用 `AskUserQuestion`（非 multiSelect）建议安装元技能插件。options:

| label | description |
|-------|-------------|
| 安装 superpowers | obra/superpowers 元技能框架：brainstorming、executing-plans、subagent-driven-development、verification-before-completion。Claude Code 内置 plugin 安装，零依赖。 |
| 安装 everything-claude-code | affaan-m/everything-claude-code：48 个 Agent、182 个 Skill、68 个命令，覆盖代码审查、构建修复、测试、质量安全、工作流编排、持续学习、多语言。 |
| 跳过 | 不安装 |

用户选「安装 superpowers」→ 执行：

```
/plugin install superpowers@superpowers-marketplace
```

用户选「安装 everything-claude-code」→ 执行：

```
/plugin marketplace add https://github.com/affaan-m/everything-claude-code
/plugin install everything-claude-code@everything-claude-code
```

安装后可选复制 rules：

```bash
git clone https://github.com/affaan-m/everything-claude-code.git /tmp/ecc
mkdir -p ~/.claude/rules/ecc
cp -R /tmp/ecc/rules/common ~/.claude/rules/ecc/
cp -R /tmp/ecc/rules/typescript ~/.claude/rules/ecc/  # 按需选择语言
```

安装完成后告知用户可使用 `/plugin list` 查看已安装插件。

## 阶段 3：复制文件 + 收集依赖

对每个选中的 skill：

1. 从 `/tmp/cc-enhance-main/skills/<name>/` 递归复制到 `~/.claude/skills/<name>/`（源仓库已含 `-cce` 后缀，无需额外改名）
2. 读 SKILL.md 末尾「环境依赖」表，逐项检查，区分「已满足」和「缺失」：
   - **CLI 工具**（pip/npm/docker 等）：`which <cmd>` 或 `<cmd> --version`
   - **Python 包**：`python3 -c "import <pkg>"`
   - **API Key / 环境变量**：先查当前 shell 环境（`env | grep <KEY>` 或 `echo $<KEY>`），若未设置则搜索常见 shell 配置文件（`~/.zshrc`、`~/.bashrc`、`~/.bash_profile`、PowerShell `$PROFILE` 等）中有无对应赋值。Key 可能在配置文件中但当前 shell 未加载。
   - **API 端点**：不做网络请求，标记为待验证

全部 skill 处理完后，汇总展示：
- 哪些 skill 已复制
- 所有缺失依赖（跨 skill 合并），标注用途、安装方式、磁盘开销

将缺失依赖按安装方式分为两类：
- **自动安装**：能通过 pip/npm/docker/apt 等命令行完成的 → 进入阶段 4
- **需注册账号**：安装方式涉及"注册""API Key""申请"等需要离开 CC 操作的 → 跳过阶段 4，统一在阶段 6（流程末尾）处理

> 分类规则依赖 SKILL.md「环境依赖」表的「安装方式」列。后续新增 skill 只需如实描述安装方式，无需修改 BOOT.md。

## 阶段 4：安装依赖（仅自动安装类）

如果阶段 3 没有「自动安装」类缺失依赖 → 跳过本阶段。

否则，用 `AskUserQuestion`（multiSelect）一次性让用户选择要安装哪些依赖。options 仅列自动安装类，description 写清用途和开销。用户勾选的 → 执行安装；未勾选的 → 跳过。

安装过程中如遇错误，报告但不中断后续安装。

> 「需注册账号」类依赖不在此阶段出现，统一在阶段 6 处理。

## 阶段 5：生成 tool-registry.json

仅当用户选了 `tool-registry`。**不要直接复制 catalog，要扫描本机后动态生成。**

### 4.1 读取 catalog

读 `/tmp/cc-enhance-main/skills/tool-registry-cce/references/tool-catalog.json`。这是常用工具的参考样本，分 `mcp` / `cli` / `api` 三组，每个条目有 `invoke` 和 `notes`。catalog 未覆盖的工具同样需要写入注册表——自行推断 invoke 即可。

### 4.2 扫描本机

逐一核查 catalog 中每个工具在本机是否存在：

- **MCP 工具**：读 `~/.claude/settings.json`（和 `~/.claude/settings.local.json`），检查 `mcpServers` 或 `mcpServersFromPlugins` 中有没有对应条目。用 catalog 中该工具的 `invoke` 前缀（如 `mcp__web-search-prime__`）去匹配。
- **CLI 工具**：检查可执行文件（`which docker`、`which gh` 等）或 Python 包（`python3 -c "import crawl4ai"`）。同时检查阶段 4 是否刚刚安装过。
- **API 端点**：检查对应的环境变量（`$TAVILY_API_KEY` 等），不做网络请求验证，标记 `not_verified`。

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

## 阶段 6：账号注册

如果阶段 3 没有「需注册账号」类缺失依赖 → 跳过本阶段。

此类依赖需要用户离开 CC 去网站操作，对体验打断最大，故放在流程末尾集中处理。每个依赖单独 `AskUserQuestion`（非 multiSelect），不要和其他依赖混在一起。

对每个需注册的依赖：
1. `AskUserQuestion`（非 multiSelect）：是否注册？options: 注册 / 跳过
2. 用户选「注册」→ 打开注册页面（macOS: `open <url>` / Linux: `xdg-open <url>` / Windows: `start <url>`）→ 等待用户粘贴 API Key / 凭证
3. 将凭证写入当前 shell 对应的配置文件（先 `echo $SHELL` 检测，再写入 `~/.zshrc` / `~/.bashrc` / `~/.bash_profile` 等），如 `export TAVILY_API_KEY=xxx`
4. 验证凭证可用（发一次轻量请求）

> 当前涉及「需注册账号」的依赖只有 Tavily API Key。后续新增 skill 若在「安装方式」中写了"注册""API Key"等关键词，会自动归入本阶段。

## 阶段 7：体验引导

安装只是开始，让用户立刻感受到技能的价值。

按以下优先级，取用户已安装的**最高优先级** skill 进行体验引导（只展示一个，用户选「跳过」则结束，不连续追问）：

### 优先级 1：搜索（web-search-cce）

1. `AskUserQuestion`（非 multiSelect）：安装完成！要不要用刚装好的搜索能力搜一下「Claude Code 最新技巧」试试效果？options: 试试 / 跳过
2. 用户选「试试」→ 按顺序尝试搜索：内置 `WebSearch` → Tavily → DuckDuckGo（最后兜底）。搜索过程自然触发 `~/.claude/tool-registry.json` 的首次验证。
3. 展示搜索结果时，**每条必须附带链接**，让用户能直接点开看原文。标注实际使用的工具和耗时。

### 优先级 2：网页拉取（web-fetch-cce）

1. `AskUserQuestion`（非 multiSelect）：安装完成！要不要试试拉取维基百科「拉丁超立方抽样」词条，看看网页提取的效果？options: 试试 / 跳过
2. 用户选「试试」→ 用 `WebFetch` 拉取 `https://zh.wikipedia.org/wiki/拉丁超立方抽样`（零依赖、<1s、实测稳定）。展示提取结果的前几段，标注工具和耗时。
3. 如 WebFetch 失败，降级到 Crawl4AI（若已安装）重试。

> 演示 URL 取自 references/crawl4ai.md 实测矩阵中标 ✅ 的站点，高概率成功。后续可换其他已验证 URL。

### 优先级 3：工具状态（tool-registry-cce）

1. 无需 AskUserQuestion，直接对 `~/.claude/tool-registry.json` 中已检测到的工具执行轻量验证：
   - CLI 工具：已在阶段 3/4 验证过，直接展示状态
   - API 端点（Tavily、DuckDuckGo 等）：发一次最小请求确认可用性
   - MCP 工具：检查 MCP server 是否可达
2. 更新注册表中各工具的 `status`（`working` / `unreachable`）和 `updated` 时间戳。
3. 向用户展示验证结果摘要：「✅ Tavily 可用」「✅ DuckDuckGo 可用」等。

### 优先级 4：Skills 市场（agent-skills-cce）

1. `AskUserQuestion`（非 multiSelect）：安装完成！要不要看看 skills.sh 上有哪些热门 skill？比如 Anthropic 官方出品的工具集？options: 看看 / 跳过
2. 用户选「看看」→ 执行 `npx skills add https://github.com/anthropics/skills --list` 展示可安装的官方 skill 列表（docx、pdf、pptx、frontend-design 等 16 个）。
3. 告知用户：想装哪个直接说「安装 skill pdf」即可。

### 扩展

后续新增 skill 若有可演示的能力，在本节追加优先级条目即可。

## 阶段 8：完成

简短总结：已安装 skills、已安装/未安装的依赖、CLAUDE.md 是否更新、已注册/未注册的账号。
