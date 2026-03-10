---
name: chainup-spot
description: ChainUp/OpenAPI V2 现货（币币）与杠杆交易技能。优先通过 Python 脚本统一调用 `sapi` 接口，减少自然语言推理开销。
---

# ChainUp Spot (Script-First)

## Why This Skill

当用户提到 OpenAPI V2、币币交易、杠杆交易、`X-CH-SIGN`、`/sapi/v2/order`、`/sapi/v1/margin/*`、`/sapi/v1/account` 等场景时，直接使用本技能。

目标：
- 优先调用脚本，不再每次临时拼接签名逻辑。
- 默认返回交易所原始 JSON。
- 不确定参数先走占位（TODO），后续由你补齐。
- 命中技能后尽快直接发起脚本调用，减少解释性输出、额外推理和重复确认。

## Runtime Entry

统一入口脚本：
- `scripts/chainup_api.py`

核心能力：
- 内置签名：`X-CH-APIKEY`、`X-CH-SIGN`、`X-CH-TS`
- 内置公共头：`Content-Type`、`admin-language`、`User-Agent`
- 统一 action -> endpoint 映射
- 真实资产变动默认要求 `--confirm CONFIRM`
- 支持 `--show-todo` 查看当前 action 的待完善必填项占位

## Execution Rules

- 命中本技能时，优先直接调用 `scripts/chainup_api.py`，不要先用自然语言重复规划接口参数。
- 查询类 action（不影响余额/订单状态）在参数明确时直接调用脚本；只在参数缺失且无法安全推断时才追问。
- 所有会影响余额、持仓、挂单或成交状态的 action，在实际执行前必须先用自然语言给出执行摘要，并等待用户手动回复精确的 `Confirm` 后再调用脚本。
- 非资金变动查询尽量不加额外确认，直接执行。
- 不要把“用户表达了想下单/撤单/划转”视为可直接执行；没有收到单独一条 `Confirm` 前，一律不得发起真实请求。
- 收到用户单独回复的 `Confirm` 后，再执行真实资金变动动作，并沿用脚本参数 `--confirm CONFIRM`。
- 若 `spot_create_order` 成功，默认继续调用 `spot_get_order` 查询订单详情。
- 所有 HTTP 调用都必须从 `scripts/chainup_api.py` 发出。
- 如果脚本调用失败，只返回脚本失败结果并继续基于 Python 脚本排查；不要改用 `curl`、手写签名请求或其他临时 HTTP 方案。
- 不要为了“验证接口”额外引入第二套实现，避免增加推理和分叉逻辑。

## Required Config

优先使用 `/root/TOOLS.md`：
- `BASE_URL: ...`
- `API_KEY: ...`
- `SECRET_KEY: ...`

其次才使用环境变量：
- `CHAINUP_BASE_URL`
- `CHAINUP_API_KEY`
- `CHAINUP_SECRET_KEY`

也可命令行传入：
- `--base-url`
- `--api-key`
- `--secret-key`

配置读取优先级：
- `命令行参数` > `/root/TOOLS.md` > `环境变量`
- 如 `/root/TOOLS.md` 中已有可用配置，优先直接使用，不要求用户先设置环境变量。

敏感信息处理：
- `CHAINUP_API_KEY` 与 `CHAINUP_SECRET_KEY` 都属于敏感信息，禁止在控制台、自然语言回复、执行摘要、报错转述、示例命令中完整显示。
- 如必须引用，只允许脱敏展示：保留前 4 位和后 4 位，中间使用 `***`，例如 `915c***815e`。
- 优先避免在可见命令中直接内联密钥；如必须执行命令，回复里不要重复整条带密钥的命令。
- 用户主动提供完整密钥后，也不得在后续任何消息中原样回显。

## Call Template

```bash
python /root/.codex/skills/chainup-spot/scripts/chainup_api.py <action> \
  --query-json '<json-object>' \
  --body-json '<json-object>' \
  --show-todo
```

说明：
- `GET` 类型 action 使用 `--query-json`
- `POST` 类型 action 使用 `--body-json`
- 对有资金/订单变动的 action，只有在用户单独回复 `Confirm` 之后，才可追加：`--confirm CONFIRM`
- 除非用户要求解释，否则优先执行再返回结果，避免先给大段说明。

## Action Map

Public:
- `spot_ping` -> `GET /sapi/v2/ping`
- `spot_time` -> `GET /sapi/v2/time`
- `spot_symbols` -> `GET /sapi/v2/symbols`
- `spot_depth` -> `GET /sapi/v2/depth`
- `spot_ticker` -> `GET /sapi/v2/ticker`
- `spot_trades` -> `GET /sapi/v2/trades`
- `spot_klines` -> `GET /sapi/v2/klines`

Spot Signed:
- `spot_create_order` -> `POST /sapi/v2/order`
- `spot_test_order` -> `POST /sapi/v2/order/test`
- `spot_batch_orders` -> `POST /sapi/v2/batchOrders`
- `spot_get_order` -> `GET /sapi/v2/order`
- `spot_cancel_order` -> `POST /sapi/v2/cancel`
- `spot_batch_cancel` -> `POST /sapi/v2/batchCancel`
- `spot_open_orders` -> `GET /sapi/v2/openOrders`
- `spot_my_trades` -> `GET /sapi/v2/myTrades`

Account Signed:
- `spot_account` -> `GET /sapi/v1/account`
- `asset_transfer` -> `POST /sapi/v1/asset/transfer`
- `asset_transfer_query` -> `POST /sapi/v1/asset/transferQuery`

Margin Signed:
- `margin_create_order` -> `POST /sapi/v1/margin/order`
- `margin_get_order` -> `GET /sapi/v1/margin/order`
- `margin_cancel_order` -> `POST /sapi/v1/margin/cancel`
- `margin_open_orders` -> `GET /sapi/v1/margin/openOrders`
- `margin_my_trades` -> `GET /sapi/v1/margin/myTrades`

## Response Rules

- 默认返回原始 JSON（脚本 stdout）。
- `spot_create_order` 成功后，默认立刻补调一次 `spot_get_order`，返回最新订单详情。
- 补查订单时，优先使用下单响应里的 `symbol + orderId`；若网关返回 `orderIdString` 也一并保留。
- `spot_account` 会在 Python 脚本内直接过滤，只返回 `free > 0` 或 `locked > 0` 的资产。
- 用户要求“只看重点”时再附加简短摘要。
- 若返回内容、错误信息或调试信息中包含完整 `api-key`、`secret-key` 或其他凭证，先脱敏再输出。

## Safety Rules

- 所有真实资产变动动作（下单、撤单、划转）默认必须先等待用户手动回复 `Confirm`，随后才允许用 `--confirm CONFIRM` 执行。
- 影响余额但不一定立即成交的动作也按真实资产变动处理，包括但不限于限价挂单、批量下单、撤单、划转、杠杆下单与撤单。
- 查询类动作可直接执行，包括但不限于余额查询、订单查询、成交查询、行情查询、挂单查询。
- 如用户明确要求跳过确认，可使用 `--no-confirm-gate`（高风险，仅在用户明确授权时使用）。
- 任何时候都不要在控制台或回复中输出完整凭证；如果脚本异常导致潜在泄漏风险，优先概述错误，不直接转抄原始敏感内容。

## Examples

查询现货资产：
```bash
python /root/.codex/skills/chainup-spot/scripts/chainup_api.py spot_account --query-json '{}'
```

现货市价下单（真实请求，需要确认）：
```bash
python /root/.codex/skills/chainup-spot/scripts/chainup_api.py spot_create_order \
  --body-json '{"symbol":"BTC/USDT","volume":"100","side":"BUY","type":"MARKET"}' \
  --confirm CONFIRM --show-todo
```

现货下单补充约定：
- `type=MARKET` 且 `side=BUY` 时，`volume` 按计价币传值。
- 例如 `ETH/USDT` 传 `volume=1` 且 `side=BUY`，表示用 `1 USDT` 市价买入 ETH。
- `type=MARKET` 且 `side=SELL` 时，`volume` 按基础币数量传值。
- 例如 `ETH/USDT` 传 `volume=0.1` 且 `side=SELL`，表示市价卖出 `0.1 ETH`。
- `type` 为非市价单时，`volume` 仍按基础币数量理解。
- `spot_create_order` 返回成功后，下一步默认不是停在下单回执，而是继续调用 `spot_get_order` 查询最终订单信息。

杠杆限价下单：
```bash
python /root/.codex/skills/chainup-spot/scripts/chainup_api.py margin_create_order \
  --body-json '{"symbol":"BTC/USDT","volume":"0.001","side":"BUY","type":"LIMIT","price":"30000"}' \
  --confirm CONFIRM --show-todo
```

## References

- 鉴权与签名：[`references/authentication.md`](./references/authentication.md)
- 现货接口：[`references/spot-endpoints.md`](./references/spot-endpoints.md)
- 杠杆接口：[`references/margin-endpoints.md`](./references/margin-endpoints.md)
