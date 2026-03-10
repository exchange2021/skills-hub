# skills-hub

[English](./README.md) | [简体中文](./README.zh-CN.md)

`skills-hub` 是一个 Codex Skills 的集中仓库。当前仓库包含 `chainup-spot` 技能，用于 ChainUp/OpenAPI V2 的现货与杠杆交易场景，并采用 script-first 运行方式，所有 HTTP 请求统一经由一个 Python 入口脚本发出。

## 仓库目标

- 统一沉淀可复用的 skill 模板与执行流程
- 集中管理交易所鉴权与请求逻辑
- 通过 Git 管理 skill 版本与协作
- 降低新 skill 的接入与维护成本

## 仓库结构

```text
skills-hub/
├── README.md
├── README.zh-CN.md
└── chainup-spot/
    ├── SKILL.md
    ├── agents/
    │   └── openai.yaml
    ├── references/
    │   ├── authentication.md
    │   ├── spot-endpoints.md
    │   └── margin-endpoints.md
    └── scripts/
        └── chainup_api.py
```

## 已包含技能

### `chainup-spot`

`chainup-spot` 是一个面向 ChainUp/OpenAPI V2 的 script-first 技能，目标是避免在对话里反复手写签名与请求逻辑，并默认返回交易所原始 JSON。

当前支持的能力包括：

- 现货公共行情查询：
  - `spot_ping`
  - `spot_time`
  - `spot_symbols`
  - `spot_depth`
  - `spot_ticker`
  - `spot_trades`
  - `spot_klines`
- 现货签名交易：
  - `spot_create_order`
  - `spot_test_order`
  - `spot_batch_orders`
  - `spot_get_order`
  - `spot_cancel_order`
  - `spot_batch_cancel`
  - `spot_open_orders`
  - `spot_my_trades`
- 账户相关签名接口：
  - `spot_account`
  - `asset_transfer`
  - `asset_transfer_query`
- 杠杆相关签名接口：
  - `margin_create_order`
  - `margin_get_order`
  - `margin_cancel_order`
  - `margin_open_orders`
  - `margin_my_trades`

## 运行入口

所有 HTTP 调用统一通过以下脚本发出：

```bash
python chainup-spot/scripts/chainup_api.py <action>
```

当前运行时行为：

- 统一处理 `X-CH-APIKEY`、`X-CH-SIGN`、`X-CH-TS`
- 统一附带 `Content-Type`、`admin-language`、`User-Agent`
- `GET` 动作使用 `--query-json`
- `POST` 动作使用 `--body-json`
- 真实资产变动动作要求 `--confirm CONFIRM`
- `spot_account` 默认只返回非零余额
- `spot_create_order` 成功后应继续补查 `spot_get_order`

## 配置方式

技能按以下优先级读取配置：

1. CLI 参数：`--base-url`、`--api-key`、`--secret-key`
2. `/root/TOOLS.md`
3. 环境变量：
   - `CHAINUP_BASE_URL`
   - `CHAINUP_API_KEY`
   - `CHAINUP_SECRET_KEY`

注意：

- 不要把生产环境密钥提交到仓库
- 不要在示例、日志或文档里完整展示密钥

## 在 Codex 中使用

把技能目录复制到本地 Codex skills 目录即可，例如：

```bash
cp -a chainup-spot ~/.codex/skills/
```

示例命令：

```bash
python ~/.codex/skills/chainup-spot/scripts/chainup_api.py spot_symbols --query-json '{}'
python ~/.codex/skills/chainup-spot/scripts/chainup_api.py spot_account --query-json '{}'
python ~/.codex/skills/chainup-spot/scripts/chainup_api.py spot_create_order \
  --body-json '{"symbol":"BTC/USDT","volume":"100","side":"BUY","type":"MARKET"}' \
  --confirm CONFIRM
```

## 安全约束

- 查询类动作可直接执行
- 下单、撤单、划转等真实资产变动动作必须先获得明确确认
- 不要绕过 `chainup_api.py` 改写成临时 `curl` 或第二套签名实现
- 日志、报错、示例中不得泄露生产凭证

## 贡献约束

- 每个 skill 必须包含 `SKILL.md`
- `references/` 需要与最新接口和鉴权规则保持一致
- `scripts/chainup_api.py` 的实际行为要与文档同步
- 禁止提交生产环境密钥、令牌或私有凭证
- 当 action、必填参数或安全门槛发生变化时，必须更新文档说明

## 版本建议

- 有条件时优先走分支和 PR 流程
- 推荐提交前缀：`feat:` / `fix:` / `docs:` / `refactor:`
- 如有外部行为变化，需要在提交信息里明确说明接口或运行时影响
