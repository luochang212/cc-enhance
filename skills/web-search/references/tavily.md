# Tavily Search API

## 基本信息

- **网站**: https://tavily.com
- **接口**: `POST https://api.tavily.com/search`
- **免费额度**: 1000 次/月
- **响应速度**: basic ~0.6s，advanced ~1.6s
- **限流**: 宽松，实测连续 5 次请求全部 200，未触发限流
- **认证**: API key 通过请求 body 的 `api_key` 字段传入

## 注册与配置

1. 注册 [tavily.com](https://tavily.com)
2. Dashboard → API Keys → 复制 key
3. 设置环境变量 `TAVILY_API_KEY`：

```bash
export TAVILY_API_KEY="你的key"
```

## 调用方式

接口是 POST JSON，`WebFetch` 不支持 POST，必须用 `Bash` + `curl`：

```bash
curl -s -X POST https://api.tavily.com/search \
  -H "Content-Type: application/json" \
  -d '{"api_key": "$TAVILY_API_KEY", "query": "<搜索词>", "max_results": 5}'
```

注意：`$TAVILY_API_KEY` 在子 shell 中可能未加载，如果调用报 Unauthorized，先 `source ~/.zshrc`。

## 参数

| 参数 | 类型 | 说明 |
|------|------|------|
| `api_key` | string | 必填 |
| `query` | string | 必填。英文效果优于中文 |
| `max_results` | int | 默认 5，最大 20。精确返回，不会少也不会多 |
| `search_depth` | string | `"basic"`（默认，快）或 `"advanced"`（慢 2.5x） |
| `include_domains` | []string | 限定域名，如 `["github.com"]`。符合条件的可能少于 `max_results` |
| `exclude_domains` | []string | 排除域名 |

## `basic` vs `advanced`

| | basic | advanced |
|------|------|------|
| 速度 | ~0.6s | ~1.6s |
| `raw_content` | 总是 `null` | 可能为 `null`，实测多数情况下仍是 `null` |
| 结果质量 | 相同 | 相同 |
| **推荐** | ✅ 日常使用 | ❌ 仅在有明确深度搜索需求时使用 |

`advanced` 速度慢 2.5 倍，且 `raw_content` 字段经常不返回。日常搜索用 `basic` 即可。

## 响应格式

```json
{
  "query": "原始搜索词",
  "results": [
    {
      "url": "https://...",
      "title": "页面标题",
      "content": "内容摘要（Markdown，约 200-500 字）",
      "score": 0.90,
      "raw_content": null
    }
  ],
  "response_time": 0.65,
  "images": []
}
```

| 字段 | 说明 |
|------|------|
| `results[].url` | 页面 URL，用 `WebFetch` 获取全文 |
| `results[].title` | 页面标题 |
| `results[].content` | Tavily 提取的 Markdown 摘要 |
| `results[].score` | 0-1 相关性评分，英文结果通常 0.9+，中文 0.8+ |
| `results[].raw_content` | 页面全文，仅 advanced 模式且少数情况下返回 |
| `response_time` | API 耗时（秒） |

## 语言表现

| 语言 | 相关性 | 结果质量 |
|------|--------|----------|
| 英文 | ⭐⭐⭐ 0.9-1.0 | 优秀，覆盖技术文档、教程、GitHub |
| 中文 | ⭐⭐ 0.8-0.9 | 可用，覆盖知乎、技术博客、飞书文档，但广度不如英文 |

**策略**：能用英文关键词就用英文，中文场景也可用但预期结果偏少。

## 域名过滤

`include_domains` 精确限定来源。注意符合条件的 URL 可能少于 `max_results`——设 3 个可能只返回 2 个。不要因此认为搜索失败。

## 限流与容错

- 实测 5 次连续请求全部 200，未触发限流
- 免费额度 1000 次/月，用完后返回 429
- 额度查询接口不可用（`/usage` 返回 Method Not Allowed），建议自己在 Tavily Dashboard 查看

## 测试命令

```bash
curl -s -X POST https://api.tavily.com/search \
  -H "Content-Type: application/json" \
  -d '{"api_key": "$TAVILY_API_KEY", "query": "test", "max_results": 1}' \
  | python3 -m json.tool | head -10
```
