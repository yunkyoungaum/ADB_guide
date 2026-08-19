# 참고 문헌

이 가이드의 서술은 아래 공식 문서를 근거로 작성·검증했습니다. 제품 UI와 제약은 자주 바뀌므로, 적용 전에 최신 문서를 다시 확인하세요.

## Unity AI Gateway

| 문서 | 본문 연관 | 링크 |
| --- | --- | --- |
| AI governance with Unity AI Gateway | 3장 전반 | [Azure](https://learn.microsoft.com/azure/databricks/ai-gateway/) · [AWS](https://docs.databricks.com/aws/en/ai-gateway/) |
| Configure routing and fallbacks for model services | 3.4 Fallback·traffic split | [Azure](https://learn.microsoft.com/azure/databricks/ai-gateway/configure-traffic-splitting) · [AWS](https://docs.databricks.com/aws/en/ai-gateway/configure-traffic-splitting) |
| Create model services | 3.5 커스텀 model service | [Azure](https://learn.microsoft.com/azure/databricks/ai-gateway/create-model-services) · [AWS](https://docs.databricks.com/aws/en/ai-gateway/create-model-services) |
| Log requests and responses to inference tables | 3.5 · 부록 A | [Azure](https://learn.microsoft.com/azure/databricks/ai-gateway/inference-tables) · [AWS](https://docs.databricks.com/aws/en/ai-gateway/inference-tables) |
| Track model usage | 4장 사용량 모니터링 | [Azure](https://learn.microsoft.com/azure/databricks/ai-gateway/usage-tracking) · [AWS](https://docs.databricks.com/aws/en/ai-gateway/usage-tracking) |
| Apply rate limits to model and MCP services | 3.5 rate limits | [Azure](https://learn.microsoft.com/azure/databricks/ai-gateway/rate-limits) · [AWS](https://docs.databricks.com/aws/en/ai-gateway/rate-limits) |
| External model providers (BYOK) | 3.5 destination 구성 | [Azure](https://learn.microsoft.com/azure/databricks/ai-gateway/model-provider-services) · [AWS](https://docs.databricks.com/aws/en/ai-gateway/model-provider-services) |
| Configure governance on model serving endpoints (legacy) | 3.4 legacy 참고 | [Azure](https://learn.microsoft.com/azure/databricks/ai-gateway/configure-ai-gateway-endpoints) · [AWS](https://docs.databricks.com/aws/en/ai-gateway/configure-ai-gateway-endpoints) |

## 서버리스 컴퓨트와 스토리지

| 문서 | 본문 연관 | 링크 |
| --- | --- | --- |
| Connect to serverless compute | 2장 (활성화 단계 불필요 근거) | [Azure](https://learn.microsoft.com/azure/databricks/compute/serverless/) |
| Default storage in Databricks | 부록 A (추론 테이블 제약) | [Azure](https://learn.microsoft.com/azure/databricks/storage/default-storage) |
| Connect to an ADLS Gen2 external location | 부록 A (Access Connector 절차) | [Azure](https://learn.microsoft.com/azure/databricks/connect/unity-catalog/cloud-storage/external-locations-adls) |
| Feature and region support | 1장 · 3장 (지원 리전) | [Azure](https://learn.microsoft.com/azure/databricks/resources/feature-region-support) |

## 배포와 거버넌스

| 문서 | 본문 연관 | 링크 |
| --- | --- | --- |
| `az databricks workspace create` | 1.2 배포 명령 | [Azure CLI 레퍼런스](https://learn.microsoft.com/cli/azure/databricks/workspace#az-databricks-workspace-create) |
| Enable a workspace for Unity Catalog | 2.1 | [Azure](https://learn.microsoft.com/azure/databricks/data-governance/unity-catalog/enable-workspaces) |
| Unity Catalog privileges reference | 3.5 · 부록 A (권한 요건) | [Azure](https://learn.microsoft.com/azure/databricks/data-governance/unity-catalog/access-control/privileges-reference) |
| System tables — billing usage | 5장 비용 분석 | [Azure](https://learn.microsoft.com/azure/databricks/admin/system-tables/billing) |
| SQL warehouses API | 2.2 웨어하우스 생성 | [REST API 레퍼런스](https://docs.databricks.com/api/workspace/warehouses/create) |
| Lakeview dashboards API | 6.2 대시보드 배포 | [REST API 레퍼런스](https://docs.databricks.com/api/workspace/lakeview/create) |

## 네트워킹

| 문서 | 본문 연관 | 링크 |
| --- | --- | --- |
| Deploy Azure Databricks in your Azure VNet (VNet injection) | 부록 C.3 | [Azure](https://learn.microsoft.com/azure/databricks/security/network/classic/vnet-inject) |
| Private Link concepts | 부록 C.1 · C.2 · C.3 | [Azure](https://learn.microsoft.com/azure/databricks/security/network/concepts/privatelink-concepts) |
| Configure private connectivity to Azure resources (NCC) | 부록 C.2 아웃바운드 | [Azure](https://learn.microsoft.com/azure/databricks/security/network/serverless-network-security/serverless-private-link) |

## 가격

| 문서 | 본문 연관 | 링크 |
| --- | --- | --- |
| Unity AI Gateway pricing | 3.5 (inference tables 과금) | [databricks.com](https://www.databricks.com/product/pricing/ai-gateway) |

> **문서 버전** — 위 문서들은 2026년 8월 기준으로 확인했습니다. Databricks는 AI Gateway 관련 문서를 빈번하게 갱신하므로, 특히 **Limitations 섹션**은 적용 시점에 다시 확인하는 것을 권장합니다.
