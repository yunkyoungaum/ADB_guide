# 부록 B. 검증 체크리스트

배포부터 모니터링까지 릴리스 전 확인 항목입니다. 각 항목은 본문의 "확인 포인트"와 연결됩니다.

## 배포

- [ ] `computeMode`가 `Serverless`이다
- [ ] `managedResourceGroupId`가 `null`이다
- [ ] 관리형 리소스 그룹(`databricks-rg-*`)이 생성되지 않았다
- [ ] 리소스 그룹에 워크스페이스 리소스 1개만 존재한다

## Unity Catalog / 서버리스

- [ ] `SHOW CATALOGS`로 워크스페이스 ID 접미사가 붙은 실제 카탈로그 이름을 확인했다
- [ ] 서버리스 SQL 웨어하우스를 생성했다
- [ ] 관리형 Delta 테이블 INSERT/SELECT가 성공한다

## 모델 호출

- [ ] AI Gateway 호출이 `200`을 반환한다
- [ ] 인증은 `Authorization: Bearer`를 사용한다(`x-api-key` 아님)
- [ ] 모델 이름은 `system.ai.claude-*` 형식이다
- [ ] 토큰은 환경 변수로만 다루고, 프롬프트/응답 본문은 로그에 남기지 않는다

## 모니터링

- [ ] 시스템 테이블 적재 지연을 감안해 충분히 대기한 뒤 조회했다
- [ ] `system.ai_gateway.usage`에 트래픽이 적재되었다
- [ ] `latency_ms` / `ttfb_ms`가 정상 범위로 조회된다

## 비용

- [ ] DBU 단가를 하드코딩하지 않고 `system.billing.list_prices`를 조인했다
- [ ] `sku_name`이 티어와 일치한다(`PREMIUM_ANTHROPIC_MODEL_SERVING` 등)
- [ ] `list_cost_usd`가 0이 아닌 값을 반환한다

## 대시보드

- [ ] 커스텀 대시보드를 배포하고 `published` 처리했다
- [ ] 조회 기간을 넉넉히(초기 검증 시 180일까지) 설정해 빈 화면을 피했다
- [ ] 위젯 5종에 데이터가 채워진다

## 추론 테이블 (해당 시)

- [ ] 추론 테이블이 조직 요건인지 확인했다
- [ ] 요건이면 external storage 카탈로그를 먼저 구성했다(default storage 미지원)
- [ ] 커스텀 model service를 만들고 inference tables를 활성화했다
