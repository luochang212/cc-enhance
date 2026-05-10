# Playwright（原生）

> 以下基于 macOS 实测（Playwright 1.59，全 Chromium headless）

## 基本信息

- **仓库**: https://github.com/microsoft/playwright
- **类型**: 微软开源浏览器自动化库
- **引擎**: Chromium / Firefox / WebKit
- **输出**: 原始 HTML / 截图 / `inner_text`
- **定位**: Crawl4AI 搞不定时的降级控制层

## 为什么需要它

Crawl4AI 是 Playwright 的包装层，有两个硬编码限制：

1. **等待策略固定为 `domcontentloaded`**——某些站点（阮一峰）永远不触发此事件，导致 60s 超时
2. **反爬策略是启发式的**——对微信加了 `magic=True` 才有效，但原生 Playwright 不加任何东西就能读

当 Crawl4AI 超时或被检测时，原生 Playwright 提供了更细粒度的控制。

## 与 Crawl4AI 的分工

| | Crawl4AI | 原生 Playwright |
|------|------|------|
| 输出 | 结构化 Markdown | `inner_text` 纯文本 |
| 反爬 | magic/undetected 策略 | 手动注入 |
| 等待策略 | 固定 domcontentloaded | commit/load/networkidle 可选 |
| 易用性 | 开箱即用 | 需手写代码 |
| 速度 | ~2-4s | ~1-5s（取决于 wait 策略） |

## 推荐配置

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    context = browser.new_context(
        user_agent="Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36",
        viewport={"width": 1920, "height": 1080},
        locale="zh-CN",
    )
    page = context.new_page()
    page.goto(url, timeout=30000, wait_until="commit")
    # 给 SPA 5 秒渲染时间
    try:
        page.wait_for_load_state("networkidle", timeout=5000)
    except:
        pass
    text = page.inner_text("body")
    browser.close()
```

## 关键参数

### `wait_until`

| 值 | 含义 | 适用场景 |
|------|------|------|
| `commit` | HTTP 响应到达即返回 | 阮一峰（domcontentloaded 永远不触发） |
| `domcontentloaded` | DOM 解析完成 | 大多数页面（Crawl4AI 默认） |
| `load` | 所有资源加载完成 | 静态页面 |
| `networkidle` | 500ms 内无新请求 | SPA（36kr、小红书） |

**阮一峰超时的根因就是 Crawl4AI 写死了 `domcontentloaded`**，而这个博客永远不触发该事件。换成 `commit` 就能拿到 540 字完整内容。

### `wait_for_load_state("networkidle")`

SPA 页面在 `commit` 后内容还是空的，需要等 JS 渲染。`networkidle` 在网络安静后返回，给 SPA 渲染时间。

## 实测对比（macOS，全 Chromium headless）

| 站点 | Crawl4AI | 原生 Playwright | 关键差异 |
|------|------|------|------|
| [阮一峰](https://www.ruanyifeng.com/blog/) | 60s 超时 | **540 字** | `wait_until=commit` |
| [微信公众号](https://mp.weixin.qq.com/s/c4QEumcT2_UKz6PREKkYwg) | 69 字 / magic 后 16K | **7,738 字** | 无需 magic |
| [36kr](https://36kr.com/) | 23K | 5,640 字 | Crawl4AI 内容更丰富 |
| [知乎](https://zhuanlan.zhihu.com/p/1985282813134660545) | **18K**（success=False） | 403 | Crawl4AI 反爬有效 |
| [苏剑林](https://www.spaces.ac.cn/) | **32K**（success=False） | 6,097 字 | Crawl4AI 内容更丰富 |
| [scikit-learn](https://scikit-learn.org/stable/user_guide.html) | **53K** | 可读 | Crawl4AI 内容更丰富 |
| [豆瓣](https://movie.douban.com/) | **26K** | 可读 | Crawl4AI 内容更丰富 |
| [微博](https://weibo.com/) | 1 字 | 0 字 | 都失败，JS 指纹检测 |
| [淘宝](https://www.taobao.com/) | 4.6K（登录墙） | 登录墙 | 都需登录 |
| [阿里百炼](https://bailian.console.aliyun.com/cn-beijing/?tab=doc#/doc) | 100 字 | SPA 空壳 | 都需登录 |

## 在 Claude Code 中使用

```bash
source <venv_path>/bin/activate && python3 << 'EOF'
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    browser = p.chromium.launch(headless=True)
    context = browser.new_context(
        user_agent="Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36",
        viewport={"width": 1920, "height": 1080},
        locale="zh-CN",
    )
    page = context.new_page()
    page.goto("<URL>", timeout=30000, wait_until="commit")
    try:
        page.wait_for_load_state("networkidle", timeout=5000)
    except:
        pass
    print(page.inner_text("body"))
    browser.close()
EOF
```

## 决策：Crawl4AI or 原生 Playwright？

```
Crawl4AI 超时或被检测？
  │
  ├─ 阮一峰 blog? ──是──▶ 原生 Playwright + wait_until="commit"
  │
  ├─ 微信公众号（不想用 magic）? ──是──▶ 原生 Playwright
  │
  ├─ 知乎? ──是──▶ Crawl4AI（反爬更强）
  │
  └─ 其他? ──▶ Crawl4AI（Markdown 质量更好）
```
