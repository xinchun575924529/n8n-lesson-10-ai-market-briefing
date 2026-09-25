# 第 01 章 五大数据源拆解

## 学习目标
- 说清每个源提供什么数据、在早报里扮演什么角色
- 拿到 4 个免费 key，知道唯一付费源怎么裁剪

## 数据源全景

| 源 | 数据 | 角色 | 费用 | 鉴权方式 |
|---|---|---|---|---|
| **Twelve Data** | SPY/QQQ/VGK/EEM/GLD/TLT/EUR/USD 日线（最近 2 根） | 跨资产价格与涨跌幅（核心源） | 免费层 800 req/天、8 req/min | query `apikey` |
| **CoinGecko** | BTC/ETH 美元价 + 24h 涨跌 | 加密板块（核心源） | 免费（demo key 或公共端点免 key） | header |
| **FRED** | 美国 10 年期国债收益率 DGS10 最近 3 个观测 | 利率锚（核心源） | 免费注册 | query `api_key` |
| **QuantGist** | CPI/NFP/FOMC/PCE 等宏观事件日历 | 今日关注事件（非核心） | ⚠️**付费** | header |
| **Marketaux** | 美/德英文财经新闻（按标的过滤） | 头条新闻（非核心） | 免费层 100 req/天 | query `api_token` |

> 「核心源」= `core: true`：核心源挂了会进 `core_source_failures` 清单，AI 提示词会收到显著警告；非核心源挂了早报照常出。

## 免费 key 获取（每个 < 5 分钟）

1. **Twelve Data**：twelvedata.com → 邮箱注册 → Dashboard 直接显示 API key。
2. **FRED**：fred.stlouisfed.org → 注册 → My Account → API Keys → Request。
3. **Marketaux**：marketaux.com → 注册 → Dashboard 拿 token（免费层每天 100 次足够日报）。
4. **CoinGecko**：课程直接用公共端点；想稳一点可注册免费 Demo key 放 header `x-cg-demo-api-key`。

## 凭证接线对照（n8n 侧）

| 模板里的凭证占位 | n8n 凭证类型 | 填什么 |
|---|---|---|
| Twelve Data API | HTTP Query Auth | name=`apikey`，value=你的 key |
| Coingecko Demo API | HTTP Header Auth | name=`x-cg-demo-api-key`（免 key 跑法：把节点 authentication 改 none） |
| FRED API | HTTP Query Auth | name=`api_key` |
| QuantGist API | HTTP Header Auth | （裁剪后删除该分支，见下） |
| Marketaux API | HTTP Query Auth | name=`api_token` |

## 付费源裁剪：把 QuantGist 裁掉的标准动作

1. 删除「Fetch QuantGist Macro Calendar」+「Normalize QuantGist Calendar Data」两个节点；
2. Merge 节点 `numberInputs` 从 5 改成 4；
3. **Prepare 节点不用改**——它会自动给缺失的 quantgist 合成 `error` 占位（这是模板设计好的容错，L2 已验证）；
4. 宏观日历免费替代思路（进阶）：FRED 加几个宏观序列（CPI/失业率），或用公开发售日历 ICS 源。

> ⚠️ 反向教训：如果只删 Fetch 不删 Normalize，Normalize 会收到空输入输出 `empty`，同样安全——但画布留死节点不规范，别这么交付。

## 本章验收
- 4 个免费 key 到手（或确认 CoinGecko 免 key 方案）；
- 能不看答案说出哪三个是核心源、为什么核心/非核心要分级。