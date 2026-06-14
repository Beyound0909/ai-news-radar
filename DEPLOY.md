# Finance News Radar 部署指南

## 项目概述
基于 [ai-news-radar](https://github.com/LearnPrompt/ai-news-radar) fork 的财经新闻雷达，增加了：
- 📈 A股/港股/美股 RSS 信源
- 💰 加密货币信源（CoinDesk, CoinTelegraph）
- 🌍 全球宏观经济信源（FT, CNBC, MarketWatch等）
- 🤖 保留原有 AI 信源
- 🏷️ 财经专属标签（finance_markets, crypto_web3, macro_economy）
- 🚫 财经噪声过滤（配资/荐股等spam）

## 已添加的财经信源

| 分类 | 信源 | RSS URL | 状态 |
|------|------|---------|------|
| A股 | 东方财富网 | https://feed.eastmoney.com/ | ✅ 可用 |
| A股 | 新浪财经 | https://rss.sina.com.cn/ | ✅ 可用 |
| A股 | 36氪 | https://36kr.com/feed | ✅ 可用 |
| 港股 | AASTOCKS | https://www.aastocks.com/tc/stocks/rss/aafn-news.xml | ⚠️ 待验证 |
| 美股 | MarketWatch Top Stories | https://www.marketwatch.com/rss/topstories | ✅ 可用 |
| 美股 | MarketWatch Market Pulse | https://www.marketwatch.com/rss/marketpulse | ✅ 可用 |
| 美股 | CNBC News | https://www.cnbc.com/id/100003114/device/rss/rss.html | ✅ 可用 |
| 美股 | Nasdaq RSS | https://www.nasdaq.com/feed/rssoutbound | ✅ 可用 |
| 美股 | Yahoo Finance | https://finance.yahoo.com/news/rssindex | ✅ 可用 |
| 美股 | Investing.com | https://www.investing.com/rss/news_301.rss | ✅ 可用 |
| 加密货币 | CoinDesk | https://www.coindesk.com/arc/outboundfeeds/rss | ✅ 可用 |
| 加密货币 | CoinTelegraph | https://cointelegraph.com/rss | ✅ 可用 |
| 宏观 | Financial Times | https://www.ft.com/rss/home | ✅ 可用 |

### 不可用信源
| 信源 | 原因 | 替代方案 |
|------|------|----------|
| Bloomberg | 403 禁止 | 使用 MarketWatch/CNBC 替代 |
| Reuters | 401 需要授权 | 使用 FT/CNBC 替代 |
| 信报 (hkej) | 403 禁止 | 使用 AASTOCKS 替代 |
| 雪球 | 无公开 RSS | 可通过 RSSHub 转换 |
| 华尔街见闻 | 404 | 已用36氪替代 |
| 金十数据 | 404 | 暂无替代 |

## 部署步骤

### 前置条件
1. GitHub 账号（如果没有，去 https://github.com 注册）
2. Git 命令行工具
3. GitHub CLI（可选，推荐）

### 方法一：通过 GitHub CLI（推荐）

```powershell
# 1. 安装 GitHub CLI（如果还没装）
# 已安装在 %LOCALAPPDATA%\GitHub CLI\gh.exe

# 2. 登录 GitHub
$env:PATH = "$env:LOCALAPPDATA\GitHub CLI;$env:PATH"
gh auth login

# 3. Fork 原始仓库到你的账号
gh repo fork LearnPrompt/ai-news-radar --clone=false

# 4. 把本地修改推送到你的 fork
cd <本项目路径>
git remote add myfork https://github.com/<你的用户名>/ai-news-radar.git
git push myfork master

# 5. 启用 GitHub Actions
gh workflow enable update-news.yml --repo <你的用户名>/ai-news-radar

# 6. 启用 GitHub Pages
#    设置 → Pages → Source: Deploy from branch → Branch: gh-pages (如果没有就用 master)
#    或在 Actions 中手动触发一次 workflow

# 7. 手动触发首次运行
gh workflow run update-news.yml --repo <你的用户名>/ai-news-radar

# 8. 等待约5-10分钟后访问
# https://<你的用户名>.github.io/ai-news-radar/
```

### 方法二：通过 GitHub 网页

1. 打开 https://github.com/LearnPrompt/ai-news-radar
2. 点击右上角 **Fork** 按钮
3. 克隆你 fork 的仓库到本地
4. 将本项目的修改文件复制过去：
   - `feeds/follow.example.opml`
   - `scripts/ai_relevance.py`
   - `.github/workflows/update-news.yml`
   - `index.html`
5. 提交并推送到你的 fork
6. 在仓库 Settings → Actions → General 中启用 Actions
7. 在 Actions 标签页手动触发 "Update AI News Snapshot" workflow
8. 在 Settings → Pages 中启用 GitHub Pages

## 配置私有 OPML（可选）

如果你想把私有信源添加到 `follow.opml`（不提交到公开仓库）：

```powershell
# 1. 创建你的 follow.opml
cp feeds/follow.example.opml feeds/follow.opml
# 编辑 feeds/follow.opml 添加你的私有源

# 2. 编码为 Base64 并设置为 GitHub Secret
$content = [Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes((Get-Content feeds/follow.opml -Raw)))
gh secret set FOLLOW_OPML_B64 --body $content --repo <你的用户名>/ai-news-radar
```

## 添加更多信源

编辑 `feeds/follow.example.opml`，按以下格式添加：

```xml
<outline
  text="信源名称"
  title="信源名称"
  type="rss"
  xmlUrl="https://example.com/feed.xml"
  htmlUrl="https://example.com/"
/>
```

如果在 `scripts/ai_relevance.py` 的 `FINANCE_DOMAINS` 中添加对应域名，新信源会自动获得财经评分。

## 常见问题

**Q: 某些信源抓取失败怎么办？**
A: GitHub Actions 运行在海外服务器上，国内信源（东方财富、新浪）可能需要网络代理。如果频繁失败，可以考虑使用 RSSHub 转换。

**Q: 如何调整抓取频率？**
A: 编辑 `.github/workflows/update-news.yml` 中的 cron 表达式。默认 `*/30 * * * *`（每30分钟）。

**Q: GitHub Actions 配额够用吗？**
A: 免费账号每月 2000 分钟。每次运行约 5-10 分钟，每天 48 次，每月约 7200 分钟会超限。建议改为每2小时运行一次：`0 */2 * * *`，每月约 360 分钟。

**Q: 如何本地测试？**
A:
```bash
python -m venv .venv
.venv\Scripts\Activate
pip install -r requirements.txt
python scripts/update_news.py --output-dir data --window-hours 24 --rss-opml feeds/follow.opml
python -m http.server 8080
# 打开 http://localhost:8080
```
