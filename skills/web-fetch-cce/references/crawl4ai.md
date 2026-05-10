# Crawl4AI

> 以下基于 macOS 实测（Crawl4AI 0.8.6，Python 3.13 venv，全 Chromium）

## 基本信息

- **仓库**: https://github.com/unclecode/crawl4ai
- **类型**: 开源 Python 库（Apache 2.0）
- **引擎**: Playwright（异步浏览器自动化）
- **输出**: Markdown、结构化 JSON、截图
- **要求**: Python 3.10+

## 推荐配置

```python
from crawl4ai import AsyncWebCrawler, BrowserConfig, CrawlerRunConfig

browser_cfg = BrowserConfig(
    browser_type="chromium",   # 全 Chromium，比 headless shell 快 3x
    headless=True,
)

run_cfg = CrawlerRunConfig(
    magic=True,                # 关键：绕过部分反爬（微信等）
    cache_mode=CacheMode.BYPASS,
)

async with AsyncWebCrawler(config=browser_cfg) as crawler:
    result = await crawler.arun(url="...", config=run_cfg)
    print(result.markdown)
```

## 浏览器选择

Playwright 提供两种 Chromium，默认安装同时下载两者：

| 浏览器 | 速度 |
|------|------|
| Headless Shell | ~7s |
| **全 Chromium** (`browser_type="chromium"`) | **~2s** |

全 Chromium 快 3 倍以上，应始终指定。

## `magic=True` —— 关键发现

`magic=True` 是 Crawl4AI 提供的启发式反爬参数。实测对不同站点效果差异极大：

| 站点 | 不加 magic | 加 magic |
|------|------|------|
| **微信文章** | 69 字"环境异常" | **16,691 字完整全文** |
| 微博 | 1 字 | 1 字（无效） |
| 淘宝 | 登录墙 | 登录墙（无效） |
| 阮一峰 | 60s 超时 | 60s 超时（无效） |

`browser_type="undetected"` 实测效果不如 `magic=True`，不推荐。

## `success` 标志不可信

Crawl4AI 的 `result.success` 在某些场景会返回 `False` 但实际拿到了完整内容：

| 站点 | `success` | Markdown 长度 | 实际情况 |
|------|------|------|------|
| 知乎文章 | False | 18,254 字 | 正文完整可读 |
| 苏剑林博客 | False | 32,430 字 | 正文完整可读 |

**评估结果应看 `len(markdown)`，不要只看 `success`。**

## 中国主流站点实测矩阵

### 两者均可 —— 日常用 WebFetch，更快

| 站点 | WebFetch | Crawl4AI |
|------|------|------|
| [豆瓣](https://movie.douban.com/) | ✅ | ✅ 26K |
| [B站](https://www.bilibili.com/) | ✅ | ✅ 11K |
| [百度百科](https://baike.baidu.com/item/Python) | ✅ | ✅ 54K |
| [维基百科](https://en.wikipedia.org/wiki/Web_scraping) | ✅ | ✅ 68K |
| [Reddit](https://www.reddit.com/r/Python/) | ✅ | ✅ 17K |
| [GitHub](https://github.com/unclecode/crawl4ai) | ✅ | ✅ 315K |
| [scikit-learn](https://scikit-learn.org/stable/user_guide.html) | ✅ | ✅ 53K |
| [阮一峰](https://www.ruanyifeng.com/blog/) | ✅ 2.1K | ❌ 60s 超时 |

### Crawl4AI 必需 —— WebFetch 读不到

| 站点 | WebFetch | Crawl4AI |
|------|------|------|
| [36kr](https://36kr.com/) | ❌ | ✅ 23K |
| [小红书](https://www.xiaohongshu.com/explore) | ❌ | ✅ 22K |

### Crawl4AI + `magic=True` 必需

| 站点 | 普通 Crawl4AI | + `magic=True` |
|------|------|------|
| [微信公众号](https://mp.weixin.qq.com/s/c4QEumcT2_UKz6PREKkYwg) | 69 字"环境异常" | **✅ 16K 全文** |

### 部分可用 —— `success=False` 但内容在

| 站点 | WebFetch | Crawl4AI | 实际 |
|------|------|------|------|
| [知乎](https://zhuanlan.zhihu.com/p/1985282813134660545) | 403 | ❌ success=False, 18K | 正文可读 |
| [苏剑林](https://www.spaces.ac.cn/) | 403 | ❌ success=False, 32K | 正文可读 |

### 两者都失败

| 站点 | 原因 |
|------|------|
| [微博](https://weibo.com/) | JS 指纹检测，0 字 |
| [淘宝](https://www.taobao.com/) | 需登录 |
| [阿里百炼](https://bailian.console.aliyun.com/cn-beijing/?tab=doc#/doc) | SPA 空壳 |

## 安装

```bash
pip install crawl4ai
playwright install chromium   # ~150MB
```

建议用 venv 隔离，conda 环境可能有 pydantic_core 冲突。

## 决策流程

```
拿到 URL
  │
  ├─ 能用 WebFetch? ──是──▶ WebFetch（<1s，零依赖）
  │
  ├─ 微信公众号? ──是──▶ Crawl4AI + magic=True
  │
  ├─ JS 渲染/SPA（36kr、小红书）? ──是──▶ Crawl4AI + 全 Chromium
  │
  ├─ 知乎/苏剑林? ──是──▶ Crawl4AI，忽略 success=False
  │
  ├─ 微博/淘宝? ──是──▶ 两者都不行
  │
  └─ 同一URL反复读? ──是──▶ Crawl4AI + CacheMode.ENABLED
```

## 在 Claude Code 中使用

```bash
source <venv_path>/bin/activate && python3 << 'EOF'
import asyncio
from crawl4ai import AsyncWebCrawler, BrowserConfig, CrawlerRunConfig, CacheMode

async def main():
    browser_cfg = BrowserConfig(browser_type="chromium", headless=True)
    run_cfg = CrawlerRunConfig(magic=True, cache_mode=CacheMode.BYPASS)
    async with AsyncWebCrawler(config=browser_cfg) as crawler:
        result = await crawler.arun(url="<URL>", config=run_cfg)
        print(result.markdown)

asyncio.run(main())
EOF
```
