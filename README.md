# Korean Real Estate Transactions MCP

<!-- mcp-name: io.github.goyo-studio/korean-real-estate-mcp -->

[한국어](#한국어) · [English](#english)

Remote MCP server: `https://realestate-api.goyostudio.io/mcp` (Streamable HTTP, no authentication)

This repository holds documentation and registry metadata only. The server is operated by goyostudio; its source code is not published here.

---

## 한국어

### 무엇을 하나요

한국 주소나 아파트 단지명을 자연어로 물으면 국토교통부 실거래가 공개 데이터(data.go.kr)를 돌려주는 원격 MCP 서버입니다. 매매·전세·월세, 기간 통계(평균·최저·최고), 월별 추이, 전용면적 조건을 지원합니다. API 키가 필요 없습니다.

### 엔드포인트

| 항목 | 값 |
|---|---|
| URL | `https://realestate-api.goyostudio.io/mcp` |
| 전송 | Streamable HTTP (무상태, `POST`만. `GET`은 405) |
| 인증 | 없음 |
| 도구 | `get_korean_real_estate_transactions` (1개) |

### 예시 질문

- "강남구 대치동 은마 아파트 매매"
- "마포구 아파트 전세 월별 추이" → `groupBy: "month"`
- "송파구 헬리오시티 전용 84" → `areaMin: 84, areaMax: 86`

도구 호출 예:

```json
{
  "name": "get_korean_real_estate_transactions",
  "arguments": { "q": "강남구 대치동 은마 아파트 매매", "limit": 5 }
}
```

지역이 모호하면 HTTP 422와 `candidates[]`가 오고, 그 후보로 다시 호출합니다. 추론한 내용은 `resolved_note`에, 빠진 달은 `missing_months`에 표시됩니다. 자세한 내용은 [docs/usage.md](docs/usage.md).

### 클라이언트 설정

**Cursor** (`~/.cursor/mcp.json` 또는 프로젝트의 `.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "korean-real-estate": {
      "url": "https://realestate-api.goyostudio.io/mcp"
    }
  }
}
```

**Claude Desktop**: 설정 → 커넥터 → 사용자 지정 커넥터 추가에서 URL `https://realestate-api.goyostudio.io/mcp` 입력. 원격 커넥터를 쓸 수 없는 환경에서는 `claude_desktop_config.json`에 [mcp-remote](https://www.npmjs.com/package/mcp-remote)를 씁니다:

```json
{
  "mcpServers": {
    "korean-real-estate": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://realestate-api.goyostudio.io/mcp"]
    }
  }
}
```

**Claude Code**: `claude mcp add --transport http korean-real-estate https://realestate-api.goyostudio.io/mcp`

### 이용 제한

IP당 **50요청 / 10초**. 초과하면 HTTP 429와 `Retry-After` 헤더가 옵니다. 그 초만큼 기다린 뒤 다시 시도하세요. 현재 무료이며, 추후 x402 프로토콜 기반 소액 과금이 도입될 수 있습니다(현재 상태: `/openapi.json`의 `x-pricing`).

### 출처와 면책

- 출처: 국토교통부 실거래가 공개시스템, data.go.kr OpenAPI. 모든 응답에 `source` 필드가 있습니다.
- 정부 공식 서비스가 아니며 국토교통부와 관계가 없습니다.
- **무보증(AS-IS)**. 신고 정정·해제로 수치가 바뀔 수 있습니다. 투자·법률·세무 자문이 아닙니다.
- 약관: https://realestate-api.goyostudio.io/terms

---

## English

### What it does

A remote MCP server that answers natural-language questions about Korean addresses or apartment complexes with official real-estate transaction records from the Ministry of Land, Infrastructure and Transport (MOLIT, via data.go.kr). It covers sales, jeonse and monthly rent, period statistics (avg/min/max), monthly trends, and exclusive-area filters. No API key.

### Endpoint

| Item | Value |
|---|---|
| URL | `https://realestate-api.goyostudio.io/mcp` |
| Transport | Streamable HTTP (stateless, `POST` only; `GET` returns 405) |
| Auth | None |
| Tool | `get_korean_real_estate_transactions` (one tool) |

### Example

```json
{
  "name": "get_korean_real_estate_transactions",
  "arguments": { "q": "강남구 대치동 은마 아파트 매매", "limit": 5 }
}
```

The query `q` is Korean. Ambiguous locations return HTTP 422 with `candidates[]`; call again with one of them. Inferences are disclosed in `resolved_note`, coverage gaps in `missing_months`. See [docs/usage.md](docs/usage.md).

### Client configuration

**Cursor** (`~/.cursor/mcp.json` or `.cursor/mcp.json`):

```json
{
  "mcpServers": {
    "korean-real-estate": {
      "url": "https://realestate-api.goyostudio.io/mcp"
    }
  }
}
```

**Claude Desktop**: Settings → Connectors → Add custom connector, URL `https://realestate-api.goyostudio.io/mcp`. Where remote connectors are not available, use [mcp-remote](https://www.npmjs.com/package/mcp-remote) in `claude_desktop_config.json`:

```json
{
  "mcpServers": {
    "korean-real-estate": {
      "command": "npx",
      "args": ["-y", "mcp-remote", "https://realestate-api.goyostudio.io/mcp"]
    }
  }
}
```

**Claude Code**: `claude mcp add --transport http korean-real-estate https://realestate-api.goyostudio.io/mcp`

### Rate limit

**50 requests / 10 seconds per IP.** Over the limit you get HTTP 429 with a `Retry-After` header; wait that many seconds and retry. Currently free. Per-request micro-pricing via the x402 protocol may be introduced later (status: `x-pricing` in `/openapi.json`).

### Data source and disclaimer

- Source: 국토교통부 실거래가 공개시스템 (MOLIT) via the data.go.kr OpenAPI. Every response carries a `source` field.
- Not affiliated with, operated by, or endorsed by MOLIT or any government body.
- Provided **AS IS, without warranty**. Official reports may be corrected or cancelled after publication. Not investment, legal, or tax advice.
- Terms: https://realestate-api.goyostudio.io/terms

## License

Documentation in this repository: [MIT](LICENSE). The data returned by the service is subject to the [terms](https://realestate-api.goyostudio.io/terms) and the original data.go.kr terms.
