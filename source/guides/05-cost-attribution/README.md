# 5. 비용 분석과 귀속

4장의 두 요청 단위 테이블은 라우트별로 배타적이었습니다. 비용은 다릅니다. **`system.billing.usage`는 두 라우트를 모두 담는 유일한 테이블**이며, 여기서 팀별 귀속과 정확한 비용 계산을 수행합니다.

## 5.1 청구 테이블이 두 라우트를 통합한다

`system.billing.usage`는 serving·gateway를 가리지 않고 모든 사용량을 담습니다. 이때 엔드포인트 이름은 **`databricks-claude-*` 형식으로 정규화**됩니다. 즉 gateway 라우트에서 `system.ai.claude-*`로 호출했더라도, 청구 테이블에서는 `databricks-claude-*`로 나타납니다.

```sql
SELECT
  usage_metadata.endpoint_name AS endpoint,
  sku_name,
  sum(usage_quantity) AS total_dbu
FROM system.billing.usage
WHERE usage_metadata.endpoint_name IS NOT NULL
GROUP BY 1, 2
ORDER BY total_dbu DESC;
```

## 5.2 팀별 귀속 — usage_context

3장에서 붙인 `--team` / `--purpose` 태그는 `usage_context`로 흘러들어와 팀별 귀속을 가능하게 합니다.

**실측 롤업 (검증 트래픽 기준):**

| 팀 | 호출 수 | 사용 모델 수 |
| --- | --- | --- |
| `team_alpha` | 22콜 | 6모델 |
| `team_beta` | 12콜 | 8모델 |
| `team_gamma` | 10콜 | 6모델 |

```sql
SELECT
  usage_context.team AS team,
  count(*) AS calls,
  count(DISTINCT served_entity_name) AS models
FROM system.serving.endpoint_usage
WHERE usage_context.team IS NOT NULL
GROUP BY 1
ORDER BY calls DESC;
```

## 5.3 ⚠️ 비용 계산 주의 — DBU 단가를 하드코딩하지 말 것

> **경고** — DBU 단가를 코드에 하드코딩하지 마세요. 반드시 `system.billing.list_prices`를 조인해서 계산합니다.

실측 목록가는 **DBU당 약 `0.105` USD**였습니다. 일부 샘플 쿼리에 등장하는 `0.07`은 실제의 약 1/3만큼 **과소 계상**된 값입니다. 하드코딩된 단가를 그대로 쓰면 비용을 심각하게 낮게 추정하게 됩니다.

**올바른 방식 — list_prices 조인:**

```sql
SELECT
  u.usage_metadata.endpoint_name AS endpoint,
  u.sku_name,
  sum(u.usage_quantity) AS dbu,
  sum(u.usage_quantity * p.pricing.effective_list.default) AS list_cost_usd
FROM system.billing.usage u
JOIN system.billing.list_prices p
  ON u.sku_name = p.sku_name
  AND u.usage_start_time >= p.price_start_time
  AND (p.price_end_time IS NULL OR u.usage_start_time < p.price_end_time)
WHERE u.usage_metadata.endpoint_name IS NOT NULL
GROUP BY 1, 2
ORDER BY list_cost_usd DESC;
```

## 5.4 SKU는 티어에 따라 다르다

비용이 붙는 SKU 이름은 워크스페이스 티어에 따라 달라집니다. 이 검증에서는 `PREMIUM_ANTHROPIC_MODEL_SERVING`을 확인해야 했습니다. 샘플 쿼리가 0건을 반환한다면 SKU 이름이 티어와 맞지 않는 경우가 많습니다.

```sql
SELECT DISTINCT sku_name
FROM system.billing.usage
WHERE sku_name ILIKE '%ANTHROPIC%';
```

> **확인 포인트**
> - `list_prices` 조인 쿼리가 0이 아닌 `list_cost_usd`를 반환하는가?
> - `sku_name`에 `ANTHROPIC`이 포함된 행이 존재하는가?
> 둘 다 통과하면 대시보드 구성으로 넘어갑니다.
