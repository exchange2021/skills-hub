# ChainUp Margin Endpoints (OpenAPI V2)

来源：
- https://exchangedocsv2.gitbook.io/open-api-doc-v2/jian-ti-zhong-wen-v2/gang-gan-jiao-yi

说明：
- 以下为杠杆交易常用接口，均需签名与 API Key。
- 签名头：`X-CH-APIKEY`、`X-CH-TS`、`X-CH-SIGN`
- 文档示例域名为占位：`https://openapi.xxx.xx`

## Margin Trade (Signed)

- `POST /sapi/v1/margin/order` 创建杠杆订单
- `GET /sapi/v1/margin/order` 杠杆订单查询
- `POST /sapi/v1/margin/cancel` 撤销杠杆订单
- `GET /sapi/v1/margin/openOrders` 杠杆当前委托
- `GET /sapi/v1/margin/myTrades` 杠杆交易记录

## Order Fields

`POST /sapi/v1/margin/order` 常用字段：
- `type`：`LIMIT` / `MARKET`
- `price`：`LIMIT` 必填
- `newClientOrderId`：客户端订单标识（<= 32）
- `side`：`BUY` / `SELL`
- `volume`：订单数量
- `symbol`：例如 `BTC/USDT`

`GET /sapi/v1/margin/order` 常用查询参数：
- `orderId`
- `newClientOrderId`
- `symbol`

`POST /sapi/v1/margin/cancel` 常用字段：
- `orderId`
- `newClientOrderId`
- `symbol`

## Practical Checklist

- 先确认用户是“现货”还是“杠杆”，避免下错市场。
- 下单前校验 `symbol`、`volume`、`price` 的精度与最小限制。
- 对真实交易动作先二次确认（`CONFIRM`）再执行。
