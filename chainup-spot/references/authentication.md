# ChainUp OpenAPI V2 Authentication

本参考基于官方文档与 FAQ：
- https://exchangedocsv2.gitbook.io/open-api-doc-v2/jian-ti-zhong-wen-v2/bi-bi-jiao-yi
- https://exchangedocsv2.gitbook.io/open-api-doc-v2/jian-ti-zhong-wen-v2/chang-jian-wen-ti

## Required Headers

- `Content-Type: application/json`
- `X-CH-APIKEY: <api_key>`
- `X-CH-SIGN: <signature>`
- `X-CH-TS: <timestamp_ms>`
- `admin-language: en_US`
- `User-Agent: chainup-spot/1.0.0 (Skill)`

## Signature Payload

官方 FAQ 基础格式：

`payload = timestamp + METHOD + requestPath + body`

按当前接入规则（与 Postman 预请求脚本一致）：

- `requestPath`：从完整 URL 中去掉 `baseUrl` 后的路径（可包含 `?query`），不能包含域名
- `timestamp`：毫秒时间戳字符串
- `METHOD`：大写 HTTP 方法
- `GET`：`payload = timestamp + "GET" + requestPath`（不拼 body，不拼 `"{}"`）
- `POST/PUT/PATCH`：`payload = timestamp + METHOD + requestPath + bodyStr`
- `bodyStr`（仅用于有请求体的方法）：
- 若原始 body 存在且去空白后不等于 `{}`，使用原始 body 字符串
- 否则固定使用 `"{}"`

通用要求：
- `POST` 的 JSON 字符串必须与实际发送体完全一致（字段名、顺序、空格）
- 若接口要求 query 参与签名，需将 query 放在 `requestPath` 中（例如 `/sapi/v2/order?orderId=...&symbol=...`）
- `X-CH-SIGN`：`HMAC_SHA256(payload, secretKey)` 后转 HEX（小写）

FAQ 示例：
- `GET`: `1588591856950GET/sapi/v1/account`
- `POST`: `1588591856950POST/sapi/v1/order/test{"symbol":"BTCUSDT","price":"9300","volume":"1","side":"BUY","type":"LIMIT"}`

Postman 等价伪代码：

```javascript
time = Date.now().toString()
requestPath = fullUrl.replace(baseUrl, "")
objStr = rawBody
if (method === "GET") {
  payload = time + method + requestPath
} else {
  bodyStr = (objStr && objStr.trim() !== "{}") ? objStr : "{}"
  payload = time + method + requestPath + bodyStr
}
sign = HmacSHA256(payload, secretKey).toHex()
headers: X-CH-SIGN, X-CH-APIKEY, X-CH-TS, admin-language
```

## HMAC SHA256 + HEX

对 `openapi.coobit.cc`，当前可用签名编码为十六进制（HEX）：

```bash
SIGN=$(printf '%s' "$PAYLOAD" \
  | openssl dgst -sha256 -hmac "$SECRET_KEY" \
  | awk '{print $2}')
```

## Curl Examples

### GET /sapi/v1/account

```bash
TS=$(date +%s%3N)
METHOD="GET"
PATH="/sapi/v1/account"
PAYLOAD="${TS}${METHOD}${PATH}"
SIGN=$(printf '%s' "$PAYLOAD" | openssl dgst -sha256 -hmac "$SECRET_KEY" | awk '{print $2}')

curl -sS "${BASE_URL}${PATH}" \
  -H "Content-Type: application/json" \
  -H "X-CH-APIKEY: ${API_KEY}" \
  -H "X-CH-TS: ${TS}" \
  -H "X-CH-SIGN: ${SIGN}" \
  -H "admin-language: en_US" \
  -H "User-Agent: chainup-spot/1.0.0 (Skill)"
```

### POST /sapi/v2/order (MARKET SELL)

```bash
TS=$(date +%s%3N)
METHOD="POST"
PATH="/sapi/v2/order"
BODY='{"symbol":"BTC/USDT","volume":"0.001","side":"SELL","type":"MARKET"}'
PAYLOAD="${TS}${METHOD}${PATH}${BODY}"
SIGN=$(printf '%s' "$PAYLOAD" | openssl dgst -sha256 -hmac "$SECRET_KEY" | awk '{print $2}')

curl -sS -X POST "${BASE_URL}${PATH}" \
  -H "Content-Type: application/json" \
  -H "X-CH-APIKEY: ${API_KEY}" \
  -H "X-CH-TS: ${TS}" \
  -H "X-CH-SIGN: ${SIGN}" \
  -H "admin-language: en_US" \
  -H "User-Agent: chainup-spot/1.0.0 (Skill)" \
  -d "$BODY"
```

### GET /sapi/v2/order (Query Order)

```bash
TS=$(date +%s%3N)
METHOD="GET"
PATH="/sapi/v2/order"
QUERY="orderId=3181965742962937069&symbol=ETH%2FUSDT"
PAYLOAD="${TS}${METHOD}${PATH}?${QUERY}"
SIGN=$(printf '%s' "$PAYLOAD" | openssl dgst -sha256 -hmac "$SECRET_KEY" | awk '{print $2}')

curl -sS "${BASE_URL}${PATH}?${QUERY}" \
  -H "Content-Type: application/json" \
  -H "X-CH-APIKEY: ${API_KEY}" \
  -H "X-CH-TS: ${TS}" \
  -H "X-CH-SIGN: ${SIGN}" \
  -H "admin-language: en_US" \
  -H "User-Agent: chainup-spot/1.0.0 (Skill)"
```

## Notes

- 服务器默认 5000ms 时间窗口，可通过业务参数 `recvwindow` 自定义。
- 若报时间戳无效，先校验本机时钟，再重试。
- 若签名校验失败，重点检查：
- 拼接顺序是否严格一致
- `requestPath` 是否包含域名（不应包含）
- `GET` 是否误拼了 body / `"{}"`（`GET` 应仅签 `timestamp + GET + requestPath`）
- 非 `GET` 空 body 是否按 `"{}"` 参与签名
- `GET` 是否应把 `?query` 并入签名路径
- `POST` 的签名体是否与实际发送 JSON 完全一致
- 输出编码是否为接口实际要求（HEX 或 Base64）
