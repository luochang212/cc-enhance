# SearXNG

> 以下基于 macOS 实测（SearXNG latest，Docker 部署）

## 基本信息

- **网站**: https://docs.searxng.org
- **类型**: 开源元搜索引擎（聚合 Google、Bing、DuckDuckGo 等）
- **费用**: 完全免费，无额度限制
- **部署**: Docker，启动后即用

## 工作原理

不维护自己的索引，实时抓取各家搜索引擎结果页，聚合去重后返回。隐私好、结果质量高、不受商业 API 额度限制。

## 部署

```bash
docker run -d --name searxng \
  -p 8080:8080 \
  -v ./searxng:/etc/searxng \
  searxng/searxng
```

**必须配置 `settings.yml`**，默认关闭了 JSON API：

```yaml
# settings.yml（在挂载目录下）
search:
  formats: [html, json]  # 默认空，不加这行 JSON API 返回 403
```

不用这行配置的话 `GET /search?format=json` 返回 403 Forbidden。

## API 调用

部署后通过 `WebFetch` 或 `Bash` + `curl` 调用本地 JSON API：

```
GET http://localhost:8080/search?format=json&q=<URL 编码的搜索词>
```

返回格式：

```json
{
  "query": "搜索词",
  "results": [
    {
      "url": "https://...",
      "title": "页面标题",
      "content": "摘要",
      "engines": ["google", "bing"],
      "score": 1.0
    }
  ]
}
```

`engines` 字段显示该结果来自哪些搜索引擎。`score` 可能超过 1.0（不同于 Tavily 的 0-1 归一化）。

## 实测数据（macOS，本地 Docker）

### 与 Tavily 对比

| | SearXNG | Tavily |
|------|------|------|
| 速度 | ~3s | ~0.6s |
| 英文结果数 | 10 条 | 5 条 |
| 中文结果数 | **18 条** | 5 条 |
| 中文质量 | 知乎、博客园、腾讯云 | 知乎、36kr、腾讯云 |
| 限流 | **无**（10 次连续请求全 200） | 1000 次/月 |
| 费用 | 免费 | 免费 1000 次/月 |

### 稳定性

- 冷门英文查询偶尔返回 0 结果（上游引擎超时），重试一次通常恢复
- 中文查询稳定，未观察到 0 结果情况
- 自托管意味着可用性取决于你的 Docker 环境和网络

## 与搜索策略的关系

部署后 SearXNG 升级为 **Tier 1**，与 Tavily 并列：

```
搜索请求
  ├─ Tavily curl        ──失败──┐
  ├─ SearXNG (本地)      ──失败──┤
  │                              ▼
  ├─ web_search_prime MCP  ──失败──┐
  │                                ▼
  └─ DuckDuckGo WebFetch    ── 兜底
```

SearXNG 适合：
- **高频搜索**——调试、批量信息收集，不用心疼额度
- **中文内容**——聚合了知乎、博客园等中文源
- **隐私敏感**——搜索词不出本地

Tavily 仍然适合：
- **速度优先**——1s vs 3s
- **英文精准搜索**——相关性评分更准确（0-1 归一化）
- **不需要维护 Docker**

## 管理

```bash
docker start searxng   # 启动
docker stop searxng    # 停止
docker logs searxng    # 查看日志（引擎超时/错误）
```
