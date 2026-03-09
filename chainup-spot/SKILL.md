---
name: chainup-spot
description: ChainUp/OpenAPI V2 现货（币币）与杠杆交易技能。用于根据用户指令调用该交易所 `sapi` 接口完成行情查询、现货下单撤单、杠杆下单撤单、当前委托、历史成交与资产查询；当用户提到 OpenAPI V2、币币交易、杠杆交易、X-CH-SIGN 鉴权、`/sapi/v2/order`、`/sapi/v1/margin/*`、`/sapi/v1/account` 等场景时使用。
---

# ChainUp Spot

## Quick Start

按以下顺序执行：

1. 确认账户配置（`apiKey`、`secretKey`、`baseUrl`）是否可用。
2. 按 [`references/authentication.md`](./references/authentication.md) 规则构造 `X-CH-SIGN`、`X-CH-APIKEY`、`X-CH-TS`。
3. 从 [`references/spot-endpoints.md`](./references/spot-endpoints.md) 或 [`references/margin-endpoints.md`](./references/margin-endpoints.md) 选择目标接口并发起请求。
4. 返回 JSON 原始结果；如失败，返回错误码与可执行修复建议。

## Response Rules

- 始终优先返回交易所原始 JSON。
- `GET /sapi/v1/account`：默认额外返回“非零余额清单”（`free != 0` 或 `locked != 0`），便于快速查看资产。
- 如用户要求“只看重点”，再补充简短汇总（成交状态、数量、金额、手续费）。
- 任何会导致真实资产变动的动作（下单、撤单、划转）在主网执行前必须二次确认，要求用户回复 `CONFIRM`。

## Account Handling

读取 `TOOLS.md` 中的账户配置。推荐结构：

```bash
## ChainUp Accounts

### main
- Base URL: https://openapi.example.com
- API Key: your_api_key
- Secret: your_secret_key
- Testnet: false
- Description: Primary spot account

### testnet-dev
- Base URL: https://openapi-test.example.com
- API Key: your_testnet_api_key
- Secret: your_testnet_secret
- Testnet: true
- Description: Testing account
```

规则：
- 未指定账户时默认 `main`。
- 展示凭证时脱敏：
- API Key: 显示前 5 位 + 后 4 位，例如 `abcde...wxyz`
- Secret: 仅显示后 5 位，例如 `***...9k3fd`
- 若未配置 `baseUrl`，先向用户确认网关地址再执行请求。

## Common Tasks

按下列映射处理高频请求：

- “查现货资产/余额” -> `GET /sapi/v1/account`
- 返回规则：优先给非零余额清单；用户要求完整数据时再附原始 JSON 全量 `balances`
- “市价买入/卖出” -> `POST /sapi/v2/order` with `type=MARKET`
- “限价下单” -> `POST /sapi/v2/order` with `type=LIMIT` and `price`
- “撤单” -> `POST /sapi/v2/cancel`
- “查当前委托” -> `GET /sapi/v2/openOrders`
- “查历史成交” -> `GET /sapi/v2/myTrades`
- “查币对规则” -> `GET /sapi/v2/symbols`
- “查盘口/行情” -> `GET /sapi/v2/depth` / `GET /sapi/v2/ticker`
- “杠杆市价/限价下单” -> `POST /sapi/v1/margin/order`
- “杠杆撤单” -> `POST /sapi/v1/margin/cancel`
- “查杠杆订单” -> `GET /sapi/v1/margin/order`
- “查杠杆当前委托” -> `GET /sapi/v1/margin/openOrders`
- “查杠杆成交记录” -> `GET /sapi/v1/margin/myTrades`

下单前检查：
- `symbol` 格式按接口要求（常见 `BTC/USDT`，部分实现也接受 `BTCUSDT`）。
- `volume`、`price` 满足最小精度与最小下单限制（来自 `symbols` 返回）。
- 主网交易必须先 `CONFIRM`。

## Authentication Summary

签名要点：
- 必填头：`X-CH-APIKEY`、`X-CH-SIGN`、`X-CH-TS`
- 建议附加头：`admin-language: en_US`
- `requestPath` 必须是相对路径（`url - baseUrl`），不能带域名
- `GET`：`timestamp + "GET" + requestPath`（不拼 body，不拼 `{}`）
- `POST/PUT/PATCH`：`timestamp + METHOD + requestPath + bodyStr`
- 对有请求体的方法：
- 如果有请求体且不为 `{}`，`bodyStr` 使用原始 JSON 字符串
- 如果空体或 `{}`，`bodyStr` 固定使用 `"{}"` 参与签名
- 若接口要求 query 参与签名，query 必须已包含在 `requestPath`（如 `/sapi/v2/order?orderId=...&symbol=...`）
- 当前 `openapi.coobit.cc` 实测：`X-CH-SIGN` 使用 HMAC SHA256 的 HEX 小写字符串

完整示例见 [`references/authentication.md`](./references/authentication.md)。

## References

- 鉴权与签名：[`references/authentication.md`](./references/authentication.md)
- 现货接口清单：[`references/spot-endpoints.md`](./references/spot-endpoints.md)
- 杠杆接口清单：[`references/margin-endpoints.md`](./references/margin-endpoints.md)
- 官方文档入口：
- https://exchangedocsv2.gitbook.io/open-api-doc-v2/jian-ti-zhong-wen-v2
- https://exchangedocsv2.gitbook.io/open-api-doc-v2/jian-ti-zhong-wen-v2/bi-bi-jiao-yi
- https://exchangedocsv2.gitbook.io/open-api-doc-v2/jian-ti-zhong-wen-v2/gang-gan-jiao-yi

## User-Agent

发送请求时附带：
- `User-Agent: chainup-spot/1.0.0 (Skill)`
