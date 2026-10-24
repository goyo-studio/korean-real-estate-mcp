# Usage / 사용법

Endpoint: `POST https://realestate-api.goyostudio.io/mcp` (MCP Streamable HTTP, stateless, no auth).
Send `Accept: application/json, text/event-stream`.

## Tool: `get_korean_real_estate_transactions`

| Parameter | Required | Description |
|---|---|---|
| `q` | yes | Natural-language Korean address and/or complex name, e.g. `강남구 대치동 은마 아파트 매매`. Max ~30 Korean characters, max 8 tokens. Period words are ignored; use `startDate`/`endDate`. Hint words (아파트/오피스텔…, 매매/전세/월세) are inferred as `type`/`tradeType`/`rentSubtype`. |
| `type` | no | Property type. Default `apartment` unless inferred from `q`. Rent is available for apartment/officetel/rowhouse/detached only; presale/land/commercial/industrial are sale-only. |
| `tradeType` | no | `sale` (매매, default) or `rent` (전월세). |
| `rentSubtype` | no | With `tradeType=rent` only: `jeonse` (전세) or `wolse` (월세). |
| `startDate` | no | `YYYY-MM`, inclusive. Default: 11 months before `endDate`. Max range 24 months. |
| `endDate` | no | `YYYY-MM`, inclusive. Default: current month. |
| `limit` | no | Max transaction rows (default 100). Statistics always cover the full matching set. |
| `areaMin` / `areaMax` | no | Exclusive area (전용면적) band in ㎡, inclusive. A "84 type" is usually 84.0–85.9㎡: `areaMin=84, areaMax=86`. |
| `groupBy` | no | `month` → `stats_by_month[]`. Use it for trend (추이) questions. |

### Response

Flat JSON: period statistics (avg/min/max price in KRW, or deposit/monthly-rent stats for rentals), individual transactions newest first, `resolved_*` fields showing how the query was resolved, `resolved_note` for inferences, `missing_months` for coverage gaps, and `source` attribution.

- Ambiguous location → HTTP 422 with `candidates[]`. Pick one and call again.
- Unresolvable location → 404.
- Rate limit (50 requests / 10 s per IP) → 429 with `Retry-After`.

## Examples / 예시

| 질문 | arguments |
|---|---|
| 은마아파트 최근 매매 | `{"q": "강남구 대치동 은마 아파트 매매", "limit": 5}` |
| 마포구 아파트 전세 월별 추이 | `{"q": "마포구 아파트 전세", "groupBy": "month"}` |
| 헬리오시티 전용 84 매매 | `{"q": "송파구 헬리오시티", "areaMin": 84, "areaMax": 86}` |
| 2025년 상반기 분당 오피스텔 월세 | `{"q": "분당구 오피스텔 월세", "startDate": "2025-01", "endDate": "2025-06"}` |

## Raw protocol check

```sh
curl -s https://realestate-api.goyostudio.io/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -d '{"jsonrpc":"2.0","id":1,"method":"tools/list"}'
```

## REST

The same data is available without MCP: `GET https://realestate-api.goyostudio.io/api/v1/real-estate/transactions?q=...`. OpenAPI: https://realestate-api.goyostudio.io/openapi.json

## Terms

https://realestate-api.goyostudio.io/terms — AS IS, no warranty, not investment advice. Source: 국토교통부 실거래가 공개시스템 (MOLIT) via data.go.kr.
