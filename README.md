# skills-hub

`skills-hub` is a centralized repository for Codex Skills. It currently includes the `chainup-spot` skill for ChainUp/OpenAPI V2 spot and margin trading workflows, with a script-first runtime that sends all HTTP requests through a single Python entrypoint.

## Purpose

- Consolidate reusable skill templates and execution workflows
- Keep exchange auth and request logic centralized
- Manage skill versions and collaboration through Git
- Enable fast and stable onboarding for new skills

## Repository Structure

```text
skills-hub/
├── README.md
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

## Included Skill

### `chainup-spot`

`chainup-spot` is a script-first skill for ChainUp/OpenAPI V2. It is designed to reduce repeated signing logic in prompts and to return the exchange's raw JSON by default.

Current capabilities include:

- Public spot market data queries:
  - `spot_ping`
  - `spot_time`
  - `spot_symbols`
  - `spot_depth`
  - `spot_ticker`
  - `spot_trades`
  - `spot_klines`
- Signed spot trading:
  - `spot_create_order`
  - `spot_test_order`
  - `spot_batch_orders`
  - `spot_get_order`
  - `spot_cancel_order`
  - `spot_batch_cancel`
  - `spot_open_orders`
  - `spot_my_trades`
- Signed account actions:
  - `spot_account`
  - `asset_transfer`
  - `asset_transfer_query`
- Signed margin actions:
  - `margin_create_order`
  - `margin_get_order`
  - `margin_cancel_order`
  - `margin_open_orders`
  - `margin_my_trades`

## Runtime Entry

All HTTP calls are sent via:

```bash
python chainup-spot/scripts/chainup_api.py <action>
```

Key runtime behavior:

- Centralized signing for `X-CH-APIKEY`, `X-CH-SIGN`, and `X-CH-TS`
- Shared headers for `Content-Type`, `admin-language`, and `User-Agent`
- `GET` actions use `--query-json`
- `POST` actions use `--body-json`
- Mutating actions require `--confirm CONFIRM`
- `spot_account` filters balances to non-zero `free` or `locked`
- `spot_create_order` is expected to be followed by `spot_get_order`

## Configuration

The skill reads config in this priority order:

1. CLI flags: `--base-url`, `--api-key`, `--secret-key`
2. `/root/TOOLS.md`
3. Environment variables:
   - `CHAINUP_BASE_URL`
   - `CHAINUP_API_KEY`
   - `CHAINUP_SECRET_KEY`

Secrets must never be committed, echoed in full, or copied into documentation examples.

## Usage in Codex

Copy the skill directory from this repository into your local Codex skills directory, for example:

```bash
cp -a chainup-spot ~/.codex/skills/
```

Example invocations:

```bash
python ~/.codex/skills/chainup-spot/scripts/chainup_api.py spot_symbols --query-json '{}'
python ~/.codex/skills/chainup-spot/scripts/chainup_api.py spot_account --query-json '{}'
python ~/.codex/skills/chainup-spot/scripts/chainup_api.py spot_create_order \
  --body-json '{"symbol":"BTC/USDT","volume":"100","side":"BUY","type":"MARKET"}' \
  --confirm CONFIRM
```

## Safety Rules

- Query actions can be executed directly
- Real asset-changing actions require explicit user confirmation
- Do not replace the Python entrypoint with ad hoc `curl` or duplicated signing code
- Keep examples and logs free of production credentials

## Contribution Guidelines

- Every skill must include a `SKILL.md`
- Keep `references/` aligned with the latest exchange endpoints and auth rules
- Keep runtime behavior in sync with `scripts/chainup_api.py`
- Never commit production secrets, tokens, or private credentials
- Document behavior changes when action names, required params, or safety gates change

## Versioning Recommendations

- Use feature branches and merge via pull requests when practical
- Recommended commit prefixes: `feat:` / `fix:` / `docs:` / `refactor:`
- If external behavior changes, describe the API or runtime impact clearly in the commit message
