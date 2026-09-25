# 《每日 AI 市场早报：多数据源 + AI Agent + 三路投递》 ── lesson-10

> **一句话卖点：一份早报，教会你"多源数据归一化 → AI 结构化产出 → 多渠道投递"的完整工程范式。**
> 股票、加密、国债、宏观日历、新闻五个数据源同时喂给一个守纪律的 AI 分析师，每个工作日早上 7:45 自动把英文市场早报发到 Telegram、Discord 并存进 Postgres——数据源挂了也不炸，AI 输出不合格就拦下。

- 课程号：**lesson-10** ｜ 来源模板：[n8n.io/workflows/19457](https://n8n.io/workflows/19457/)
- 项目：19457（n8n 官方模板）｜ 工厂库：`n8n-lesson-factory/projects/19457-send-daily-ai-market-briefings-to-telegram-disco/`
- 版本：L3 ｜ 日期：2026-09-25（Asia/Shanghai）

---

## 学完你能得到什么

1. **一套可直接上线的每日市场早报机器人**：工作日定时产出英文结构化早报，三路投递（Telegram / Discord / Postgres）。
2. **「归一化 + 健康度」工程范式**：5 个数据源各挂一个同构的 Normalize 节点（ok/error/empty/invalid 四态 + 错误消毒），任何源挂了都不中断主链路——这套模式可以搬到你自己的任何多源项目。
3. **AI Agent 纪律性提示词模板**：禁止编造、因果谨慎用词、只输出 JSON——实测在 DeepSeek 上完美复现（L2 验证：价格涨跌幅与输入 100% 一致）。
4. **AI 输出强校验层**：JSON 解析失败/枚举越界/字段缺失一律拦下或兜底，绝不让脏数据进数据库和群消息。

## 学员画像

- 完成过 **lesson-05（Telegram AI 助理）** 或同等基础：会导入 n8n 工作流、会配凭证、看得懂 JSON。
- 想做**金融/资讯类自动化**（日报、盯盘、舆情）的个人或工作室；或想给客户交付"AI 早报"服务的接单者。

## 前置条件

- n8n 2.x 实例（本机或云）。
- **零付费也能学**：DeepSeek（约 ¥0.01/次）、CoinGecko 免 key，Twelve Data / FRED / Marketaux 免费注册；QuantGist 付费源可裁剪（课程教你怎么裁）。
- Telegram bot / Discord webhook / Postgres 均有免费获取路径（第 05 章给清单）。

## 章节目录

| 章 | 标题 | 一句话 |
|---|---|---|
| 00 | 课程总览与模板导入 | 10 分钟把 19457 跑起来看骨架 |
| 01 | 五大数据源拆解 | 每个源查什么、免费 key 怎么拿、付费源怎么裁 |
| 02 | 归一化模式（本课灵魂） | 四态状态机 + 错误消毒，逐行讲透 |
| 03 | AI Agent 纪律性提示词 | 2500 字英文提示词逐段拆解 + 换 DeepSeek |
| 04 | 校验与三路投递 | Validate 拦截逻辑、TG/Discord/PG 格式化 |
| 05 | 部署、排错与二次开发 | 上云、改时区改标的、加数据源的标准动作 |

## 本包文件

- `README.md`（本文件）｜ `docs/00~05` 六章教程 ｜ `exercises/exercise.md` 三级练习 ｜ `script/short-video.md` 获客口播稿
- `workflow.json`——原始模板工作流（导入即用，模型替换步骤见 docs/03）
- `site/index.html`——课程展示页（GitHub Pages 自动部署）

## 验证记录（L2 摘要）

- **A 单元级**：7 个 Code 节点忠实移植，**53/53 断言全过**；发现 4 个模板行为怪癖（详见 docs/02 与工厂库 l2-report.md）。
- **B 全流程**：mock 五源 + DeepSeek 真调 → 校验通过（mild_risk_on）→ TG/Discord/SQL 三产物齐；AI 输出与输入数据完全一致、零编造。