# DuckDuckGo

## 基本信息

- **费用**: 完全免费，无需注册
- **定位**: 最后兜底，仅在前三层不可用时使用

## 主要方式：ddgs

**安装**：`pip install ddgs`（~1MB，依赖 primp、lxml）

**调用**：

```python
from ddgs import DDGS

results = list(DDGS().text("搜索词", max_results=5))
# 每条: {title, body, href}
```

**优点**：自动处理反爬，返回结构化数据，稳定可靠。

---

## 兜底：裸 curl

当 `ddgs` 不可用时降级使用。

### 调用方式

`WebFetch` 直接 GET URL：

```
https://html.duckduckgo.com/html/?q=<URL编码的搜索词>
```

`curl` 调用需带浏览器 User-Agent，否则返回空白页：

```bash
curl -s -L \
  -A "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36" \
  "https://html.duckduckgo.com/html/?q=<query>"
```

### 反爬与限流

- **User-Agent 检测**：无 UA 或默认 curl UA 时返回空白首页
- **CAPTCHA**：连续请求可能触发 "Select all squares containing a duck" 验证
- **频控**：连续 2-3 次请求后触发 HTTP 202，约 2 秒后恢复

### 结果解析

返回 HTML，每一条结果的结构：

```
<a class="result__a" href="//duckduckgo.com/l/?uddg=<URL编码的真实链接>&rut=...">标题</a>
<a class="result__snippet">摘要文本</a>
```

**链接提取**：`href` 中的 `uddg` 参数是 URL 编码后的目标链接，需解码。

解析步骤：
1. 从 `<a class="result__a">` 提取标题和 `uddg` 值
2. URL-decode `uddg` 获得真实链接
3. 从 `<a class="result__snippet">` 提取摘要

### 使用策略

- **不要静默降级**——告知用户「上层搜索通道不可用，正在用 DuckDuckGo 兜底，结果质量可能有限」
- 获取链接后优先用 `WebFetch` 读原文验证相关性
- HTTP 202 表示被限流，等 3 秒后重试一次，仍失败则放弃
