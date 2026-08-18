# 4. 사용량 모니터링

3장에서 AI Gateway로 보낸 호출이 실제로 어느 테이블에 어떤 컬럼으로 쌓이는지를 다룹니다. 데이터는 즉시 반영되지 않을 수 있으니, 비어 있으면 잠시 후 새로고침해 다시 조회하세요.

## 4.1 3계층 모니터링 구조

사용량은 세 개의 계층으로 관찰합니다.

```mermaid
flowchart LR
    C[클라이언트] --> G[gateway 사용량<br/>ai_gateway.usage]
    G --> B[청구<br/>billing.usage]
```

1. **클라이언트** — 호출 시점의 요청/응답 (실시간, 로컬)
2. **gateway 사용량** — `system.ai_gateway.usage`
3. **청구** — `system.billing.usage`

## 4.2 사용량 테이블이 비어 있다면

AI Gateway 대시보드나 `system.ai_gateway.usage`가 계속 비어 있다면 다음을 확인하세요.

- 호출이 `{host}/ai-gateway/...` 경로로 나갔는가 (Model Serving 엔드포인트를 직접 호출하면 이 테이블에 기록되지 않습니다)
- 시스템 테이블 적재 지연을 감안해 충분히 대기했는가
- 조회 기간이 너무 좁지 않은가

> **⚠️ Fallback 사용 시** — fallback이 발생한 요청은 라우팅 결정이 `routing_information` 필드에 기록됩니다. 호출 수를 셀 때는 이 필드를 함께 확인해야 실제 시도 횟수를 정확히 파악할 수 있습니다([3.4절](../03-calling-claude/README.md) 참조).

## 4.3 테이블별 주요 컬럼

| 테이블 | 주요 컬럼 | 용도 |
| --- | --- | --- |
| `system.ai_gateway.usage` | `endpoint_name`, `status_code`, `input_tokens`, `output_tokens`, `latency_ms`, `ttfb_ms`, `routing_information` | 요청 단위 사용량 + 지연 + 라우팅 |
| `system.billing.usage` | `sku_name`, `usage_quantity`(DBU), `usage_metadata`, `custom_tags` | 청구/비용 |

## 4.4 조회 예제

`monitoring.sql`의 해당 섹션을 그대로 사용합니다.

**A. gateway 사용량 (지연 포함):**

```sql
SELECT
  request_time,
  endpoint_name,
  status_code,
  input_tokens,
  output_tokens,
  latency_ms,
  ttfb_ms
FROM system.ai_gateway.usage
WHERE request_time >= current_timestamp() - INTERVAL 24 HOURS
ORDER BY request_time DESC;
```

**B. 모델별 지연 롤업:**

```sql
SELECT
  endpoint_name,
  count(*) AS calls,
  round(avg(latency_ms)) AS avg_latency_ms
FROM system.ai_gateway.usage
WHERE status_code = 200
GROUP BY endpoint_name
ORDER BY avg_latency_ms;
```
