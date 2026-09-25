# 第 04 章 校验层与三路投递

## 学习目标
- 看懂 Validate 节点的"三重执法"
- 配通 Telegram / Discord / Postgres 三条投递路（可任选）

## Validate AI Output：AI 输出的海关

```
AI 输出 → ① 找文本（output/text/response/message/content 多字段兜底）
        → ② JSON.parse（失败 → 抛错，整条执行失败，什么都不投递）
        → ③ market_regime 枚举校验（只认五档，越界 → 抛错）
        → ④ 字段规范化兜底（assets/drivers/watch_next 统一成标准结构，非数组 → []）
```

设计哲学：**硬错误（无法解析/枚举越界）直接炸断链路——宁可今天没早报，不能发错早报；软问题（字段缺失）兜底放行**。

一个实用细节：`market_regime` 如果 AI 给了对象 `{label:'Risk-Off'}`，Validate 会自动归一成 `risk_off`——对模型的小任性有容忍度，但不放任意值。

## 三路投递

### ① Telegram
- Build Telegram Message（Code）：把 JSON 排成 TG 消息文本（自动处理 4096 字符上限截断），输出 `telegram_text`；
- Send to Telegram：需要 Bot token（@BotFather 免费建）+ chat_id（给你的 bot 发条消息后用 getUpdates 查）。
- ⚠️ 国内服务器连 Telegram 需要代理/海外节点；云上海外 ECS 直连即可。

### ② Discord
- Build Discord Embed（Code）：拼 embed JSON（标题/颜色/字段块）；
- Send to Discord：最省事的用法是频道 → 设置 → 集成 → **Webhook** 创建一个，把节点改为 HTTP Request POST 到 webhook URL 也行。免费零审批。

### ③ Postgres 入库
- Create Database Insert Query（Code）：拼参数化 INSERT（JSON 字段打 `::jsonb`）；
- Insert into Postgres：需要一张表，建表 SQL：

```sql
CREATE TABLE public.briefings (
  id BIGSERIAL PRIMARY KEY,
  generated_at timestamptz DEFAULT now(),
  market_regime text, headline text, summary text,
  assets jsonb, events_today jsonb, top_news jsonb,
  drivers jsonb, opportunities jsonb, watch_next jsonb,
  raw_sources jsonb, model_provider text, model_name text
);
```

- 免费 Postgres：Supabase / Neon 注册即得（500MB 免费层够存几十年日报）。
- 不想上 PG？把这一支整支删掉即可（便签写明：不用的投递分支可直接禁用）。

## 动手任务
1. 只接 Telegram 一路跑通（最常见交付形态）；
2. 故意把 AI 输出的 `market_regime` 改成 `euphoric`（在 Validate 前插一个 Code 节点篡改），确认执行在 Validate 处抛错、TG 没收到消息——体会"海关"的价值。

## 本章验收
至少一路投递真发出消息/写入记录，并能说出 Validate 哪类问题抛错、哪类兜底。