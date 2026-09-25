# 第 00 章 课程总览与模板导入

## 学习目标
- 说清这个工作流"每个工作日早上替你干什么"
- 把模板导入自己的 n8n 并看懂 17 个节点的分组

## 这个工作流在干什么

```
工作日 07:45 定时触发
  → 并行拉 5 个数据源（股指K线/加密货币/美债收益率/宏观日历/财经新闻）
  → 每个源一个 Normalize 节点 → 统一四态 {ok, error, empty, invalid}
  → Merge 5 路 → Prepare 汇总 + 源健康度
  → AI Agent（纪律性提示词）产出结构化英文早报 JSON
  → Validate 强校验（不是合法 JSON / 枚举越界 → 直接抛错拦截）
  → 三路投递：Telegram 消息 / Discord embed / Postgres 入库
```

## 动手：导入模板

1. 打开 https://n8n.io/workflows/19457/ → 点「Use workflow」复制 JSON（或直接用本课程包的 `workflow.json`）。
2. n8n 画布 → 右上「…」→ Import from File/URL。
3. 导入后先**不要激活**。点 Manual 执行一次，观察报错——全部报错是正常的（凭证都没配），这节课的目的就是看懂结构。

## 节点地图（对照便签分组）

| 分组 | 节点 | 作用 |
|---|---|---|
| 触发+配置 | When Weekdays at 7:45 AM、Setup Workflow Configuration | cron `45 7 * * 1-5`；全局配置（时区/模型名） |
| 拉数 | Fetch ×5（httpRequest） | 全部设了 `onError: continueRegularOutput` + `retryOnFail`——**源挂了不中断** |
| 归一化 | Normalize ×5（code） | 统一四态输出，本课灵魂（第 02 章） |
| 汇总 | Merge Source Data（5 输入）、Prepare Data for AI Model | 合并 + 健康度 + 核心源失败清单 |
| AI | AI Market Briefing Agent + OpenAI Chat Model | 第 03 章教你换成 DeepSeek |
| 校验 | Validate AI Output | JSON/枚举/字段三重关卡 |
| 投递 | Build Telegram Message→Send to Telegram；Build Discord Embed→Send to Discord；Create Database Insert Query→Insert into Postgres | 三个 Build 都是纯 Code 节点，可先单测 |

## 常见坑
- ⚠️ 模板挂的模型名 `gpt-5.6-luna` **是占位假名**，直接跑 AI 节点必挂——必须按第 03 章换掉。
- ⚠️ Postgres 节点要求先建表 `public.briefings`，建表 SQL 见第 04 章。
- ⚠️ 模板默认时区 `Europe/Berlin`，在 Setup 节点改成你的目标受众时区。

## 本章验收
能画出（或口述）上面那张数据流图，并指出"哪个环节保证源挂了不炸、哪个环节保证 AI 乱说话不炸"。