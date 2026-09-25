# 课后练习（三级）

## 练习一（开卷 · 概念）
1. 五个数据源里哪三个是核心源？核心/非核心分级影响了什么？（提示：`core_source_failures`）
2. Normalize 节点的四态分别是什么？周末 FRED 没有新数据会是哪一态，为什么这不算故障？
3. 为什么错误消息要"消毒"后才进 AI 提示词？

## 练习二（动手 · 改造）
1. 把跟踪标的里的 `EEM` 换成 `EWJ`（日本 ETF），跑通并截图 TG 消息；
2. 裁掉 QuantGist 分支（按 docs/01 的标准动作），确认 Prepare 节点自动合成 error 占位、早报照常产出；
3. 给早报加中文：改提示词 + 两个 Build 节点的模板文案。

## 练习三（实战 · 全链路）
1. 注册 Twelve Data / FRED / Marketaux 三个免费 key + BotFather 建一个 TG bot，全链路真跑一次；
2. 把 Postgres 换成 Supabase 免费实例，完成入库；
3. **挑战**：加第 6 个数据源（建议：FRED 的 VIX 序列 `VIXCLS` 做恐慌指数），按 docs/05-B 五步完成，并让 AI 在 regime 判断里参考它（改提示词 MARKET 段）。

## 自评标准
- 练习二做完：你已掌握归一化模式的可迁移性；
- 练习三做完：你可以独立给客户交付"AI 日报"类项目。