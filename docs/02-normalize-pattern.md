# 第 02 章 归一化模式（本课灵魂）

## 学习目标
- 逐行读懂 Normalize 节点的「四态状态机 + 错误消毒」
- 能把这套模式搬到自己未来的任何多源项目

## 为什么需要归一化

5 个数据源的返回结构完全不同：Twelve Data 按标的分键、CoinGecko 按币种分键、FRED 给 `observations` 数组、QuantGist 给分组事件、Marketaux 给文章列表。如果直接塞进 AI 提示词，AI 要在 5 种结构里游泳，幻觉率飙升。模板的做法：**每个源后面跟一个 Normalize Code 节点，统一输出同一个信封**：

```json
{
  "source": "twelve_data",
  "core": true,
  "status": "ok | error | empty | invalid",
  "data": { ...仅 ok 时有值... },
  "error": "仅 error 时有值，且已消毒"
}
```

## 四态状态机（判定顺序就是代码顺序）

```js
const failed  = Boolean(raw?.error || raw?.errors || raw?.status === 'error' || (raw?.code && raw?.message));
const empty   = raw == null || (typeof raw === 'object' && !Array.isArray(raw) && Object.keys(raw).length === 0);
// 各源自己的结构校验 → invalid
const status  = failed ? 'error' : empty ? 'empty' : invalid ? 'invalid' : 'ok';
```

- **error**：API 明确报错（httpRequest 配 `continueRegularOutput`，错误对象流进 Code 节点）
- **empty**：空响应（周末/假日 FRED 没新数据就会这样——**正常业务态，不是故障**）
- **invalid**：结构对不上（API 改版早期预警）
- **ok**：提取出业务字段（如 Twelve Data 顺手算好 `change_percent`）

## 错误消毒 sanitizeError（为什么值钱）

原始报错又长又脏（AxiosError 堆栈、内部路径）。消毒函数做三件事：
1. 砍堆栈：`AxiosError:...`、`at ...`、n8n 内部路径全部剥掉；
2. 归一化高频错误：`credentials not found` / `401` / `403` / `429` / apikey 无效 → 固定短文案；
3. 截断 300 字符。

**消毒后的错误是要进 AI 提示词的**——干净、一致的错误描述让 AI 能在早报里如实说"某源因限流缺失"，而不是被一堆堆栈搞晕。

## 实测发现的 4 个行为怪癖（L2 断言固化，面试级细节）

1. 裸 `{message:'...'}`（无 error/status/code 字段）**不会**被判 error，会落到 invalid；
2. `sanitizeError` 不读 `r.code`，HTTP 前缀只看 status/statusCode/response.status；
3. `{statusCode:500}` 单独出现**不满足 failed 检测**（检测项里没有 statusCode），会漏成 ok——真实场景 httpRequest 层会先产出 error 字段兜住，但你要知道这个缝隙；
4. Prepare 节点的 `all_sources_completed` **恒为 true**（先合成缺失占位再检查）——判断数据完整性要看 `core_source_failures`。

## Prepare Data for AI Model：汇总 + 健康度

```js
// 缺的分支自动合成 error 占位，core 源失败单独列清单
core_source_failures: [{source:'fred', status:'error', error:'...'}, ...]
source_health: { twelve_data:{status,error,core}, ... }  // 进提示词
```

## 动手任务
给 Normalize Twelve Data 节点喂以下四种输入，预测 status 再运行核对：
`{}`、`{status:'error',code:401,message:'apikey is incorrect'}`、`{foo:1}`、`{SPY:{values:[{close:'600'},{close:'594'}]}}`
（答案：empty / error+「API key missing or invalid」/ invalid / ok 且 change_percent≈1.01%）

## 本章验收
不看代码说出四态的判定顺序，以及「为什么 error 文案要消毒后才进 AI 提示词」。