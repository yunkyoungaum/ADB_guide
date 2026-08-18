# 용어집

이 가이드에서 반복적으로 등장하는 핵심 용어를 정리합니다.

**Unity Catalog (UC)**
Databricks의 통합 거버넌스 계층. 카탈로그 → 스키마 → 테이블 계층으로 데이터·모델·권한을 중앙에서 관리합니다. 서버리스 워크스페이스 생성 시 자동 활성화되며, 기본 카탈로그 이름에는 워크스페이스 ID가 접미사로 붙습니다.

**Default Storage**
서버리스 워크스페이스가 관리형 테이블 데이터를 저장하는 기본 스토리지 계층. 사용자가 외부 스토리지를 구성하지 않아도 동작하지만, **추론 테이블(inference table)은 default storage 카탈로그에 만들 수 없습니다.** 서버리스 워크스페이스에서도 직접 보유한 클라우드 오브젝트 스토리지로 카탈로그를 만들 수 있습니다(부록 A).

**AI Gateway**
여러 모델 서비스 앞단에 놓여 인증·라우팅·사용량 기록을 통합 제공하는 게이트웨이. 이 가이드의 모든 호출(`/ai-gateway/anthropic/v1/messages`)이 이를 통해 이뤄지며, 사용량은 `system.ai_gateway.usage`에 기록됩니다.

**DBU (Databricks Unit)**
Databricks 사용량·과금의 기본 단위. 처리량을 정규화한 값으로, 실제 비용은 `DBU 수량 × 단가`로 계산합니다. 단가는 하드코딩하지 말고 `system.billing.list_prices`에서 가져와야 합니다(실측 목록가 약 0.105 USD/DBU).

**inference table (추론 테이블)**
모델 호출의 요청/응답 페이로드를 Delta 테이블에 기록하는 기능. 페이로드 수준 디버깅·감사에 사용합니다. AI Gateway에서는 **커스텀 model service에서만** 활성화할 수 있으며, **external storage 카탈로그가 필요**합니다(default storage 미지원). 자세한 내용은 3.5절과 부록 A를 참조하세요.

**model service (모델 서비스)**
Unity Catalog에 등록되어 AI Gateway가 서빙하는 단위. `system.ai` 스키마의 기본 제공 모델을 바로 쓸 수도 있고, 직접 생성해 destination·fallback·traffic split을 구성할 수도 있습니다.

**destination (목적지)**
model service가 실제로 요청을 보내는 모델 백엔드. Primary 그룹의 destination들 사이에는 트래픽을 비율로 분배할 수 있고, 그 아래로 fallback 체인을 구성합니다.

**routing_information**
`system.ai_gateway.usage`에 기록되는 라우팅 결정 정보. 어떤 destination이 선택됐는지, fallback이 발생했는지를 추적할 때 사용합니다.

**fallback**
요청이 `429`/`5XX`로 실패하면 다음 destination으로 자동 재시도하는 기능. model service의 **Routing** 탭에서 우선순위를 직접 지정하며, Pay-per-token 모델도 destination으로 사용할 수 있습니다. 단 `system.ai` 기본 model API는 fallback을 "소유"할 수 없으므로 직접 model service를 생성해야 합니다(3.4절).
