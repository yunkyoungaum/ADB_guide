# 1. 워크스페이스 배포

이 장의 목표는 Azure Databricks 워크스페이스를 **서버리스 컴퓨트 모드**로 정확히 배포하는 것입니다. 이 부분에서 가장 흔하게 발생하는 함정은 "서버리스로 만들었다고 생각했는데 실제로는 Hybrid로 만들어지는" 문제입니다. 이 장은 그 함정을 피하는 데 초점을 맞춥니다.

## 1.1 컴퓨트 모드 비교

Azure Databricks 워크스페이스는 두 가지 컴퓨트 모드로 만들 수 있습니다.

| 항목 | Hybrid (클래식) | Serverless |
| --- | --- | --- |
| 컴퓨트 위치 | 사용자 구독 내 VNet/VM | Databricks가 관리하는 서버리스 풀 |
| 관리형 리소스 그룹 | 생성됨 (`databricks-rg-*`) | 생성되지 않음 (`managedResourceGroupId`가 `null`) |
| 커스텀 VNet / 암호화 | 지원 | 미지원 |
| Access Connector | 사용 가능 | **사용 불가** |
| 초기 운영 부담 | 높음 (인프라 직접 관리) | 낮음 |

이 가이드는 서버리스를 전제로 진행합니다. 단, 추론 테이블(inference table)이 필요하면 서버리스로는 Access Connector를 만들 수 없어 제약이 생깁니다. 자세한 내용은 [부록 A](../../appendix/inference-tables.md)를 참고하세요.

## 1.2 ⚠️ 핵심 함정 — Serverless와 함께 쓸 수 없는 것들

> **경고** — `--compute-mode Serverless`는 다음과 **동시에 사용할 수 없습니다.**
> - `managedResourceGroupId` (관리형 리소스 그룹 지정)
> - 커스텀 VNet
> - 커스텀 암호화(CMK)
> - Access Connector

특히 주의할 점은, **ARM 템플릿 본문에 관리형 리소스 그룹을 넣으면 워크스페이스가 조용히 Hybrid로 생성된다**는 것입니다. 오류가 나지 않기 때문에 배포가 성공한 것처럼 보이지만, 실제로는 서버리스가 아닙니다.

## 1.3 배포 명령

먼저 로그인하고 리소스 그룹을 만듭니다.

```bash
az login

az group create \
  --name <resource-group> \
  --location <region>
```

워크스페이스를 서버리스 모드로 생성합니다. 관리형 리소스 그룹을 **지정하지 않는 것**이 핵심입니다.

```bash
az databricks workspace create \
  --resource-group <resource-group> \
  --name <workspace-name> \
  --location <region> \
  --sku premium \
  --compute-mode Serverless
```

> `--sku premium`은 AI Gateway와 시스템 테이블 기반 모니터링을 사용하기 위한 전제입니다. 실제 SKU 티어는 조직 계약에 따라 확인하세요.

## 1.4 검증

배포가 **실제로 서버리스인지** 반드시 확인합니다. 세 가지를 모두 통과해야 합니다.

```bash
az databricks workspace show \
  --resource-group <resource-group> \
  --name <workspace-name> \
  --query "{computeMode: computeMode, managedRg: managedResourceGroupId}"
```

| 검증 항목 | 기대값 |
| --- | --- |
| `computeMode` | `Serverless` |
| `managedResourceGroupId` | `null` |
| 리소스 그룹 내 워크스페이스 리소스 | 워크스페이스 1개만 존재 |

## 다음 단계

워크스페이스가 서버리스로 확인되면, Unity Catalog가 자동 활성화되어 있습니다. 다음 장에서 카탈로그 이름을 확인하고 서버리스 컴퓨트를 켭니다.
