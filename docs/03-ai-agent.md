# 第 03 章 AI Agent：纪律性提示词 + 换 DeepSeek

## 学习目标
- 逐段拆解这份 2500 字英文提示词为什么"防幻觉"
- 把占位的假模型换成 DeepSeek 并一次跑通

## ⚠️ 必做：替换假模型

模板挂的 Chat Model 节点模型名是 `gpt-5.6-luna`——**这是作者的未来占位名，根本不存在**，不换必挂。替换步骤：

1. 删掉「OpenAI Chat Model GPT-5.6 Luna」节点；
2. 拖入 **DeepSeek Chat Model** 节点（n8n 2.x 内置 `@n8n/n8n-nodes-langchain.lmChatDeepSeek`）；
3. 建 DeepSeek 凭证（api.deepseek.com 注册充 ¥1 够用几百次）；
4. 模型选 `deepseek-chat`，temperature 建议 0.3；
5. 接回 AI Agent 节点的 Chat Model 输入；
6. 把「Setup Workflow Configuration」里的 `model_provider` 改 `deepseek`、`model_name` 改 `deepseek-chat`（这两个字段会写进 Postgres 记录，便于追溯）。

> L2 已实测：DeepSeek-chat + 原提示词 + `response_format=json_object`，输出 JSON 合法、数据与输入 100% 一致、零编造。

## 提示词逐段拆解（这是模板最值钱的资产）

| 段落 | 原文意图 | 为什么有效 |
|---|---|---|
| CORE RULES | 禁止编造价格/事件/因果；数据缺失要明说 | 给 AI 画死红线，幻觉率大降 |
| 「may be related / is in focus / is a potential driver」 | 因果不确定时的规定措辞 | 把"谨慎"落到具体词表，不是空喊 |
| MARKET/RATES RULES | market_regime 五选一；TLT 是债券价格不是收益率；US10Y 只能引用 FRED | 防最常见的概念混淆（债券价格↑=收益率↓） |
| MACRO | events_today 只放当日；actual/forecast/previous 缺了就 null | 禁止脑补数值 |
| NEWS | 最多 3 条、必须真与市场相关、不够格就返回 `[]` | 允许"没有新闻"，防凑数 |
| 输出契约 | 固定 JSON schema（13 个顶层字段全列出） | 给下游 Validate 节点一个可校验的靶子 |

### 数据注入点（三处表达式）
提示词里通过 `{{ JSON.stringify($('Prepare Data for AI Model').item.json.xxx) }}` 注入：
- `normalized_sources`——五个源的全部归一化数据
- `source_health`——源健康度（AI 据此写 missing_sources）
- `core_source_failures`——核心源失败清单（AI 必须显著提示）

### 教学要点
**「提示词工程」在这个模板里不是玄学，是合约**：提示词承诺输出 schema → Validate 节点负责执法 → 不合法直接抛错重跑。下一章看执法侧。

## 动手任务
1. 把某核心源（如 FRED）的 key 故意填错跑一次，观察 AI 输出的 `missing_sources` 是否如实报告；
2. 把 temperature 调到 1.5 跑三次，对比 JSON 稳定性（体会为什么生产建议 0.3）。

## 本章验收
- 换好 DeepSeek 后手动执行，Validate 节点不报错；
- 能解释提示词里 "Never claim that a headline or macro event caused a market move unless the supplied data directly supports it" 防的是什么。