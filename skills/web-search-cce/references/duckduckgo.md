# DuckDuckGo

## 基本信息

- **接口**: `https://html.duckduckgo.com/html/?q=<query>`
- **费用**: 完全免费，无需注册
- **响应速度**: ~2s
- **语言**: 英文为主，中文也可用但结果偏少

## 定位

**最后兜底。** 仅在更优通道全部不可用时启用，且启用前告知用户。

## 调用方式

`WebFetch` 直接 GET URL，无需特殊 header：

```
https://html.duckduckgo.com/html/?q=<URL编码的搜索词>
```

`curl` 调用需带浏览器 User-Agent，否则返回空白页：

```bash
curl -s -L \
  -A "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36" \
  "https://html.duckduckgo.com/html/?q=<query>"
```

## 反爬与限流

- **User-Agent 检测**：无 UA 或默认 curl UA 时返回空白首页，需模拟浏览器
- **频控**：连续 2-3 次请求后触发 HTTP 202，约 2 秒后恢复。实际可用频率约为 **1 次/3 秒**
- 这意味着 DuckDuckGo 不适合批量搜索，仅适合单次兜底

## 结果解析

返回 HTML，每一条结果的结构：

```
<a class="result__a" href="//duckduckgo.com/l/?uddg=<URL编码的真实链接>&rut=...">标题</a>
<a class="result__snippet">摘要文本</a>
```

**链接提取**：`href` 中的 `uddg` 参数是 URL 编码后的目标链接，需解码才能拿到真实 URL。

Claude 解析步骤：
1. 从 `<a class="result__a">` 提取标题和 `uddg` 值
2. URL-decode `uddg` 获得真实链接
3. 从 `<a class="result__snippet">` 提取摘要
4. 每页返回约 10 条结果

## 使用策略

- **不要静默降级**——切换前告知用户「上层搜索通道不可用，正在用 DuckDuckGo 兜底，结果质量可能有限」
- 获取链接后优先用 `WebFetch` 读原文验证相关性
- 不去重（DuckDuckGo 本身就是聚合，不会重复）
- HTTP 202 表示被限流，等 3 秒后重试一次，仍失败则放弃
