# 第 05 章 部署、排错与二次开发

## 学习目标
- 把早报工作流部署成 7×24 无人值守
- 掌握三个最常见的二次开发动作

## 部署清单

1. **先手动跑一次全绿**（5 源 ok、Validate 过、至少一路投递成功），再激活 Schedule；
2. **时区**：Schedule Trigger 的 cron `45 7 * * 1-5` 跟随 n8n 实例时区（默认 UTC！）。在 n8n 环境变量设 `TZ` 和 `GENERIC_TIMEZONE`；Setup 节点里的 `report_timezone` 控制的是新闻回溯窗口和事件时间显示，两者别搞混；
3. **上云建议**：数据源+TG/Discord 都需要外网，国内云服务器连 TG 要另想办法，海外节点（如新加坡 ECS）最省事；
4. **监控**：n8n 开执行日志保留；Validate 抛错=当天无投递，建议加一个 Error Trigger 工作流给管理员发飞书/TG 告警。

## 三个高频二次开发

### A. 改跟踪标的
改两个地方：① Fetch Twelve Data 的 `symbol` 参数（逗号分隔，注意免费层 8 req/min，标的多了一次查不完就分批）；② Fetch Marketaux 的 `symbols` 参数。Normalize 代码**完全不用动**——这就是归一化模式的红利。

### B. 加第六个数据源（标准动作）
1. 拖 httpRequest（`onError: continueRegularOutput` + retry）；
2. 复制任意 Normalize 节点，改三处：`source` 名、`core` 布尔、数据提取段；
3. Merge 节点 `numberInputs` +1，接入；
4. Prepare 的 `expected` 数组加上新源名；
5. （可选）提示词里补一段该源的字段说明。

### C. 中文化
提示词里「All human-readable fields must be in English」改成中文要求即可；Validate 无需改动（枚举值不变）。注意 Telegram/Discord 的 Build 节点里有少量英文模板文案要同步翻。

## 常见故障速查

| 症状 | 大概率原因 | 处置 |
|---|---|---|
| 全部源 error | 凭证类型建错（Query vs Header Auth 搞反） | 对照 docs/01 的凭证接线表 |
| AI 节点报错 model not found | 没换假模型 gpt-5.6-luna | docs/03 换 DeepSeek |
| Validate 报 not valid JSON | 模型 temperature 太高 / 没开 JSON mode | temperature≤0.3，DeepSeek 开 response_format |
| 周末数据 empty 刷屏告警 | 把 empty 当 error 监控了 | empty 是正常业务态，监控只盯 error+核心源 |
| TG 发不出、报错 ETIMEDOUT | 服务器在国内 | TG 走海外节点或代理 |

## 本章验收
工作流激活后连续 3 个工作日自动产出早报（或等效的手动触发记录），并能完成一次"加数据源"演练。