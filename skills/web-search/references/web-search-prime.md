# web_search_prime MCP

## 基本信息

- **提供商**: 智谱 AI（Zhipu GLM Coding Plan）
- **文档**: https://docs.bigmodel.cn/cn/coding-plan/mcp/search-mcp-server
- **类型**: Remote MCP Server（HTTP 协议，无需本地部署）
- **工具名**: `mcp__web-search-prime__web_search_prime`
- **能力**: 全网搜索、实时信息（新闻、股价、天气）

## 套餐与额度

这是 GLM Coding Plan 的专属 MCP，额度取决于套餐等级：

| 套餐 | 搜索次数/月 | 备注 |
|------|------------|------|
| Lite | 100 | 与网页读取 MCP 合计 |
| Pro | 1,000 | 与网页读取 MCP 合计 |
| Max | 4,000 | 与网页读取 MCP 合计 |

额度每月重置，用完后返回 429，当月无法调用。

## 安装

### Claude Code

一键安装（替换 `your_api_key`）：

```bash
claude mcp add -s user -t http web-search-prime \
  https://open.bigmodel.cn/api/mcp/web_search_prime/mcp \
  --header "Authorization: Bearer your_api_key"
```

或手动编辑 `~/.claude.json`：

```json
{
  "mcpServers": {
    "web-search-prime": {
      "type": "http",
      "url": "https://open.bigmodel.cn/api/mcp/web_search_prime/mcp",
      "headers": {
        "Authorization": "Bearer your_api_key"
      }
    }
  }
}
```

### 其他客户端

支持 Cline、OpenCode、Crush、Goose、Roo Code、Kilo Code。详见 [官方文档](https://docs.bigmodel.cn/cn/coding-plan/mcp/search-mcp-server)。

## 参数

```
mcp__web-search-prime__web_search_prime({
  search_query: "<搜索词>",
  search_domain_filter?: "<域名>",
  search_recency_filter?: "oneDay" | "oneWeek" | "oneMonth" | "oneYear" | "noLimit",
  content_size?: "medium" | "high",
  location?: "cn" | "us"
})
```

| 参数 | 说明 | 推荐 |
|------|------|------|
| `search_query` | 不超过 70 字符 | — |
| `search_domain_filter` | 限定域名 | GitHub 搜索用 `"github.com"` |
| `search_recency_filter` | 时间范围 | 新闻用 `"oneWeek"`，技术文档用 `"noLimit"` |
| `content_size` | 摘要长度（medium 400-600 字，high 2500 字） | 日常 `"medium"`，深度 `"high"` |
| `location` | 地区偏好 | 中文 `"cn"`，英文 `"us"` |

## 响应格式

```json
{
  "organic": [
    {
      "title": "页面标题",
      "link": "页面 URL",
      "snippet": "摘要",
      "date": "日期"
    }
  ],
  "related_searches": [...]
}
```

## 优势

相比 Tavily 的独特价值：
- **域名过滤** — `search_domain_filter` 精确限定来源
- **时间范围** — `search_recency_filter` 支持 1 天/1 周/1 月等粒度
- **内容长度控制** — `content_size: "high"` 返回 2500 字摘要
- **地区偏好** — `location` 优化中文/英文结果

在需要「限定 GitHub」「最近一周」「详细摘要」的场景下，优先用 web_search_prime 而非 Tavily。

## 错误处理

| 错误 | 原因 | 处理 |
|------|------|------|
| `MCP error -429` | 月额度耗尽 | 更新 `tool-registry.json` 状态为 `rate_limited`，降级到 Tavily |
| 访问令牌无效 | API key 错误或过期 | 检查智谱开放平台 API key 是否激活、余额是否充足 |
| 连接超时 | 网络问题 | 检查防火墙，确认 `open.bigmodel.cn` 可达 |
| 搜索结果为空 | 关键词过于具体 | 尝试更通用的搜索词 |
