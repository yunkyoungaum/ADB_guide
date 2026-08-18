# 6. 대시보드 구성

시스템 테이블에 데이터가 쌓였다면, 이제 이를 시각화합니다. 기본 제공 대시보드와 코드로 배포하는 커스텀 대시보드 두 가지를 다룹니다.

## 6.1 기본 제공 대시보드 — Workspace AI Gateway Usage Analytics

Databricks는 `Workspace AI Gateway Usage Analytics`라는 대시보드를 기본 제공합니다. 별도 배포 없이 바로 쓸 수 있지만 다음과 같은 **한계**가 있습니다.

- **gateway 트래픽만** 표시합니다. serving 라우트로 부른 호출은 나타나지 않습니다(4장의 배타성 때문).
- `workspace_id`로 필터링됩니다.
- 기본 조회 기간이 **7일**입니다. 초기 검증 직후에는 데이터가 적어 빈 화면처럼 보일 수 있습니다.

> serving 라우트 사용량과 비용 귀속까지 한 화면에서 보려면 커스텀 대시보드가 필요합니다.

## 6.2 커스텀 대시보드를 코드로 배포하기

커스텀 대시보드는 `dashboard.json`으로 정의하고 Lakeview API로 배포합니다.

**생성:**

```bash
curl -X POST \
  https://adb-<workspace-id>.<shard>.azuredatabricks.net/api/2.0/lakeview/dashboards \
  -H "Authorization: Bearer $DATABRICKS_TOKEN" \
  -H "Content-Type: application/json" \
  -d @dashboard.json
```

**수정(PATCH):**

```bash
curl -X PATCH \
  https://adb-<workspace-id>.<shard>.azuredatabricks.net/api/2.0/lakeview/dashboards/<dashboard-id> \
  -H "Authorization: Bearer $DATABRICKS_TOKEN" \
  -H "Content-Type: application/json" \
  -d @dashboard.json
```

배포 후에는 **게시(publish)**해야 워크스페이스 사용자에게 노출됩니다.

```bash
curl -X POST \
  https://adb-<workspace-id>.<shard>.azuredatabricks.net/api/2.0/lakeview/dashboards/<dashboard-id>/published \
  -H "Authorization: Bearer $DATABRICKS_TOKEN"
```

## 6.4 위젯 구성 (5종)

커스텀 대시보드에 배치한 위젯입니다.

| # | 위젯 | 데이터 소스 | 설명 |
| --- | --- | --- | --- |
| 1 | 라우트별 호출 수 | `endpoint_usage` + `ai_gateway.usage` | serving/gateway 호출량 비교 |
| 2 | 모델별 지연 | `system.ai_gateway.usage` | `haiku`/`sonnet`/`opus` 평균 지연 |
| 3 | 팀별 귀속 | `system.serving.endpoint_usage` | `usage_context.team` 롤업 |
| 4 | 모델별 비용(목록가) | `billing.usage` + `list_prices` | list_prices 조인 기반 |
| 5 | 토큰 추이 | 두 요청 단위 테이블 | 입력/출력 토큰 시계열 |

## 6.5 스크린샷 위치 표시자

게시 시 다음 위치에 실제 스크린샷을 삽입하세요.

- 기본 대시보드: `![기본 AI Gateway 대시보드 화면](../.gitbook/assets/ai-gateway-default-dashboard.png)`
- 커스텀 대시보드: `![커스텀 사용량 대시보드 화면](../.gitbook/assets/custom-usage-dashboard.png)`

> **확인 포인트**
> - 커스텀 대시보드가 게시(`published`)되어 다른 사용자에게 보이는가?
> - 조회 기간을 넓혔을 때 위젯 5종에 데이터가 채워지는가?
