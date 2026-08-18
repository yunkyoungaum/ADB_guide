# 용어집

이 가이드에서 반복적으로 등장하는 핵심 용어를 정리합니다.

**Unity Catalog (UC)**
Databricks의 통합 거버넌스 계층. 카탈로그 → 스키마 → 테이블 계층으로 데이터·모델·권한을 중앙에서 관리합니다. 서버리스 워크스페이스 생성 시 자동 활성화되며, 기본 카탈로그 이름에는 워크스페이스 ID가 접미사로 붙습니다.

**Default Storage**
서버리스 워크스페이스가 관리형 테이블 데이터를 저장하는 기본 스토리지 계층. 사용자가 외부 스토리지를 구성하지 않아도 동작하지만, **추론 테이블(inference table)은 지원하지 않습니다**(부록 A).

**AI Gateway**
여러 모델 서빙 엔드포인트 앞단에 놓여 인증·라우팅·사용량 기록을 통합 제공하는 게이트웨이. 이 가이드의 gateway 라우트(`/ai-gateway/anthropic/v1/messages`)가 이를 통해 호출되며, 사용량은 `system.ai_gateway.usage`에 기록됩니다.

**DBU (Databricks Unit)**
Databricks 사용량·과금의 기본 단위. 처리량을 정규화한 값으로, 실제 비용은 `DBU 수량 × 단가`로 계산합니다. 단가는 하드코딩하지 말고 `system.billing.list_prices`에서 가져와야 합니다(실측 목록가 약 0.105 USD/DBU).

**inference table (추론 테이블)**
모델 서빙의 요청/응답 페이로드를 자동으로 Delta 테이블에 기록하는 기능. 페이로드 수준 감사에 사용합니다. 서버리스 + Default Storage 조합에서는 지원되지 않으며, 외부 ADLS Gen2 + Access Connector가 필요합니다.

**usage_context**
모델 호출에 부착하는 귀속 태그(예: `team`, `purpose`). 클라이언트의 `--team` / `--purpose` 옵션으로 지정하며, `system.serving.endpoint_usage.usage_context` 컬럼으로 흘러들어와 팀별 사용량·비용 귀속에 사용됩니다.

**route (라우트)**
Claude 모델을 호출하는 경로 구분. `serving`(`/serving-endpoints`, `databricks-claude-*`)과 `gateway`(`/ai-gateway`, `system.ai.claude-*`) 두 가지가 있으며, **라우트가 사용량 기록 위치를 결정**합니다. 두 요청 단위 테이블은 배타적입니다.

**fallback**
엔드포인트에 올린 모델이 `429`/`5XX`로 실패하면 다음 served entity로 자동 재시도하는 AI Gateway 기능. External models 전용이며 최대 2개까지 지정할 수 있습니다. 트래픽 `0%` 모델은 fallback 전용으로 동작합니다.
