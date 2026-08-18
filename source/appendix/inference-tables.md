# 부록 A. 추론 테이블과 외부 스토리지

추론 테이블(inference table)은 모델 호출의 **요청/응답 페이로드**를 Delta 테이블에 기록하는 기능입니다. 토큰 수·지연만 남기는 사용량 테이블과 달리, 프롬프트와 응답 본문까지 남기므로 디버깅·감사에 사용합니다.

## A.1 두 가지 추론 테이블을 구분하세요

이름은 같지만 구성 방식과 요건이 다릅니다.

| 구분 | Unity AI Gateway inference tables | legacy Model Serving 페이로드 로깅 |
| --- | --- | --- |
| 대상 | AI Gateway **model service** | Model Serving **엔드포인트** |
| 설정 위치 | model service 페이지 → **Inference tables** → `Set Up` | 엔드포인트의 AI Gateway 섹션 → `Enable inference tables` |
| 필요 권한 | UC 권한 (`MANAGE`, `CREATE TABLE` 등) | 동일 계열의 UC 권한 |
| 스토리지 | **external storage 카탈로그 필수** (default storage 미지원) | 구성에 따라 다름 |

이 가이드는 AI Gateway를 기준으로 하므로 **왼쪽 열**을 사용합니다. 설정 절차와 스키마는 [3.5절](../guides/03-calling-claude/README.md)에서 다룹니다.

## A.2 요건과 제약

**필요 권한**

- model service에 대한 `MANAGE` 권한 (생성자와 수정자 모두 필요)
- 대상 catalog·schema에 대한 `CREATE TABLE`, `USE CATALOG`, `USE SCHEMA` 권한
- 대상 catalog가 현재 메타스토어에 대한 OpenSharing 카탈로그가 아닐 것
- Unity Catalog가 활성화된 지원 리전 워크스페이스

**제약**

| 항목 | 내용 |
| --- | --- |
| **외부 스토리지 카탈로그 전용** | 추론 테이블은 **external storage 카탈로그에만** 만들 수 있습니다. **default storage 카탈로그는 지원되지 않습니다.** |
| 프라이빗 엔드포인트 | 프라이빗 엔드포인트로 보호된 스토리지에는 만들 수 없습니다. |
| 전달 보장 | 대개 수 분 내 적재되지만 **전달이 보장되지 않습니다**(best effort). |
| 페이로드 크기 | **10 MiB를 초과**하는 요청·응답은 기록되지 않으며, `logging_error_codes`에 `MAX_REQUEST_SIZE_EXCEEDED` 등이 남습니다. |
| 오류 응답 | `401`·`403`·`429`·`500` 응답은 기록되지 않을 수 있습니다. |

> **⚠️ 서버리스 워크스페이스라면 주의** — 서버리스 워크스페이스의 기본 카탈로그는 **default storage**를 사용합니다. 따라서 추론 테이블을 쓰려면 **A.3의 외부 스토리지 구성이 반드시 선행되어야 합니다.**

> **그 밖의 주의** — 기존 테이블을 지정할 수 없습니다. 첫 요청 이후 Databricks가 새 테이블을 자동 생성하며, 행이 나타나기까지 최대 1시간이 걸릴 수 있습니다. 테이블의 스키마·이름을 바꾸거나 삭제하면 로깅이 중단되거나 손상됩니다. 또한 inference tables는 **과금 대상 기능**입니다.

## A.3 외부 스토리지 구성 (필수)

추론 테이블은 external storage 카탈로그에만 생성되므로, 아래 구성이 **선행 조건**입니다.

> **서버리스 워크스페이스에서도 구성할 수 있습니다.** 서버리스 워크스페이스는 default storage 외에 **직접 보유한 클라우드 오브젝트 스토리지에도 카탈로그를 만들 수 있습니다.** Access Connector는 Azure 리소스로 별도 생성하는 것이므로, 워크스페이스의 컴퓨트 모드와 무관합니다.

구성 절차는 다음과 같습니다.

1. **ADLS Gen2 스토리지 계정·컨테이너** 준비
   - 계층적 네임스페이스(HNS)가 활성화돼 있어야 합니다.
   - 이그레스 비용을 피하려면 워크스페이스와 **같은 리전**에 두세요.
   - WORM(불변성) 정책이 적용된 컨테이너는 사용할 수 없습니다.
   - **프라이빗 엔드포인트로만 접근 가능한 스토리지는 사용할 수 없습니다.**
2. **Access Connector for Azure Databricks** 생성 (Azure Portal)
   - 관리 ID(시스템 할당 또는 사용자 할당)를 켜고 **Resource ID**를 기록합니다.
   - 리소스 그룹에 대한 `Contributor` 또는 `Owner` 권한이 필요합니다.
3. 스토리지 계정에서 해당 관리 ID에 **`Storage Blob Data Contributor`** 역할 부여
   - 스토리지 계정에 대한 `Owner` 또는 `User Access Administrator` 역할이 필요합니다.
4. Unity Catalog에 **storage credential** 생성 → 이를 참조하는 **external location** 생성
   - 메타스토어에 대한 `CREATE STORAGE CREDENTIAL`, `CREATE EXTERNAL LOCATION` 권한이 필요합니다.
5. 해당 external location을 사용하는 **카탈로그·스키마** 생성

```sql
CREATE CATALOG <catalog-name>
  MANAGED LOCATION 'abfss://<container>@<storage-account>.dfs.core.windows.net/<path>';

CREATE SCHEMA <catalog-name>.<schema-name>;
```

6. model service의 **Inference tables** → `Set Up`에서 위 catalog·schema를 지정

> 스토리지 계정에서 공용 네트워크 액세스를 차단했다면, **Allow Azure trusted services** 옵션을 켜야 Databricks가 접근할 수 있습니다.

## A.4 도입 판단

추론 테이블은 외부 스토리지 구성이라는 **선행 작업과 추가 비용**을 수반합니다. 다음을 먼저 확인하세요.

- **페이로드 수준 로깅이 정말 필요한가** — 토큰 수·지연·상태 코드·라우팅 정보만 필요하다면 `system.ai_gateway.usage`로 충분하며 별도 구성이 필요 없습니다([4장](../guides/04-usage-monitoring/README.md)).
- **외부 스토리지를 준비할 수 있는가** — 스토리지 계정 생성, Access Connector, RBAC 역할 부여에 Azure 구독 권한이 필요합니다.
- **감사 요건이 전달 보장을 요구하는가** — 추론 테이블은 best effort이며 일부 오류 응답과 10 MiB 초과 페이로드는 누락될 수 있습니다. 엄격한 무결성이 필요한 감사에는 부적합할 수 있습니다.

> **확인 포인트** — 추론 테이블이 조직 요건이라면, 외부 스토리지 카탈로그를 먼저 구성한 뒤 model service에서 활성화하세요. default storage 카탈로그를 지정하면 설정할 수 없습니다.
