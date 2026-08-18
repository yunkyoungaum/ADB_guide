# Azure Databricks Claude 모델 운영 가이드

> **대상 독자** · Azure Databricks를 처음 배포하고 Claude 모델 사용량을 관리해야 하는 플랫폼 엔지니어, 데이터 플랫폼 운영자
>
> **선행 지식** · Azure CLI 기본, SQL 기본. Databricks 경험은 없다고 가정합니다.
>
> **범위** · 서버리스 워크스페이스 배포 → Claude 모델 호출(serving / gateway 2가지 라우트) → 사용량·비용 모니터링 → 대시보드 구성
>
> **⚠️ 초안(Draft)** — 실측 기반으로 작성된 리뷰용 문서입니다. 본문의 수치와 명령은 검증 트래픽 147건 기준 결과이며, 식별자는 모두 placeholder로 마스킹되어 있습니다.

---

## 이 가이드가 다루는 것

이 가이드는 Azure Databricks 워크스페이스를 **서버리스 컴퓨트 모드**로 배포하고, 그 위에서 **Claude 모델을 두 가지 방법(serving 라우트 · gateway 라우트)으로 호출**한 뒤, 사용량과 비용을 **시스템 테이블 3종**으로 모니터링하기까지의 전 과정을 다룹니다.

모든 서술은 실제 배포·호출 결과에 기반합니다.

## 최종 결과물 미리보기

이 가이드를 끝까지 따르면 다음을 갖추게 됩니다.

- **서버리스 워크스페이스** — 관리형 리소스 그룹이 생성되지 않고, `managedResourceGroupId`가 `null`인 상태
- **다중 모델 클라이언트** — 여러 Claude 모델을 `--route`, `--team`, `--purpose` 태그와 함께 호출하고 실패를 격리하는 클라이언트
- **3개 시스템 테이블 기반 모니터링** — `system.serving.endpoint_usage`, `system.ai_gateway.usage`, `system.billing.usage`
- **AI/BI 대시보드** — 기본 제공 AI Gateway 대시보드 + 코드로 배포하는 커스텀 대시보드

## 전체 아키텍처

```mermaid
flowchart LR
    C[클라이언트<br/>claude_client.py] -->|--route serving| S["/serving-endpoints"]
    C -->|--route gateway| G["/ai-gateway"]
    S --> EU[system.serving.endpoint_usage<br/>usage_context 귀속]
    G --> AG[system.ai_gateway.usage<br/>latency, TTFB]
    S --> B[system.billing.usage]
    G --> B
    EU --> D1[커스텀 대시보드]
    AG --> D2[기본 AI Gateway 대시보드]
    B --> D1
```

핵심은 **라우트가 기록 위치를 결정한다**는 점입니다. serving 라우트로 부른 호출은 `system.serving.endpoint_usage`에, gateway 라우트로 부른 호출은 `system.ai_gateway.usage`에 기록됩니다. 두 요청 단위 테이블은 **배타적**입니다. 반면 청구 테이블 `system.billing.usage`는 두 라우트를 **모두** 담습니다.

## 배포 흐름 한눈에 보기

```mermaid
flowchart TD
    A[az login] --> B[리소스 그룹 생성]
    B --> C[워크스페이스 생성<br/>--compute-mode Serverless]
    C --> D[Unity Catalog 확인]
    D --> E[서버리스 기능 활성화]
    E --> F[서버리스 SQL 웨어하우스 생성]
    F --> G[검증 테이블 생성/조회]
    G --> H[Claude 모델 호출]
    H --> I[시스템 테이블 조회]
    I --> J[대시보드 배포]
```
