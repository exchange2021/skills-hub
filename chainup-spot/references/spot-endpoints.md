# ChainUp Spot Endpoints (OpenAPI V2)

来源：
- https://exchangedocsv2.gitbook.io/open-api-doc-v2/jian-ti-zhong-wen-v2/bi-bi-jiao-yi

说明：
- 以下为现货（币币）常用接口，路径与鉴权规则以线上网关实际行为为准。
- 文档示例域名为占位：`https://openapi.xxx.xx`

## Public

- `GET /sapi/v2/ping` 测试连通性
- `GET /sapi/v2/time` 服务器时间
- `GET /sapi/v2/symbols` 币对和精度/最小值信息
- `GET /sapi/v2/depth` 订单簿
- `GET /sapi/v2/ticker` 24h 行情
- `GET /sapi/v2/trades` 最近成交
- `GET /sapi/v2/klines` K 线

## Trade (Signed)

- `POST /sapi/v2/order` 创建订单
- `POST /sapi/v2/order/test` 测试下单
- `POST /sapi/v2/batchOrders` 批量下单（最多 10）
- `GET /sapi/v2/order` 查询订单
- `POST /sapi/v2/cancel` 撤单
- `POST /sapi/v2/batchCancel` 批量撤单（最多 10）
- `GET /sapi/v2/openOrders` 当前委托
- `GET /sapi/v2/myTrades` 交易记录

## Account (Signed)

- `GET /sapi/v1/account` 现货账户资产
- 实务输出建议：先从 `balances` 中筛出非零资产（`free != 0` 或 `locked != 0`）作为默认结果；需要对账时再返回全量 `balances`
- `POST /sapi/v1/asset/transfer` 账户划转
- `POST /sapi/v1/asset/transferQuery` 划转记录

## Order Fields

`POST /sapi/v2/order` 常用字段：
- `symbol` 例如 `BTC/USDT`（部分实现支持 `BTCUSDT`）
- `volume` 订单数量
- `side` `BUY` / `SELL`
- `type` `LIMIT` / `MARKET` / `FOK` / `POST_ONLY` / `IOC` / `STOP`
- `price`（`LIMIT` 必填）
- `triggerPrice`（`STOP` 相关）
- `recvwindow` 可选
- `newClientOrderId` 可选

## Practical Checklist

发交易请求前先做：
- 从 `/sapi/v2/symbols` 获取该交易对 `pricePrecision`、`quantityPrecision`、最小下单约束。
- 按精度截断 `price` 与 `volume`。
- 主网真实交易先让用户回复 `CONFIRM`。
