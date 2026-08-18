# 2. Unity Catalog와 서버리스 컴퓨트

워크스페이스가 서버리스로 생성되면 Unity Catalog(UC)가 자동으로 활성화됩니다. 이 장에서는 자동 생성된 카탈로그의 실제 이름을 확인하고, 서버리스 SQL 웨어하우스를 만들어 관리형 Delta 테이블로 동작을 검증합니다.

## 2.1 Unity Catalog 자동 활성화

서버리스 워크스페이스를 만들면 UC가 자동으로 켜지고, **워크스페이스별 기본 카탈로그**가 생성됩니다.

> **⚠️ 경고 — 카탈로그 이름에 워크스페이스 ID가 접미사로 붙습니다.**
> 기본 카탈로그 이름은 다음과 같은 형태입니다.
> ```
> adb_test260814_7405616053436125
> ```
> 뒤의 긴 숫자가 워크스페이스 ID입니다. **문서나 예시의 카탈로그 이름을 그대로 복사해서 쓰면 반드시 실패합니다.** 코드를 작성하기 전에 실제 이름을 먼저 확인하세요.

Catalog Explorer(UI) 또는 SQL로 실제 이름을 확인합니다.

```sql
SHOW CATALOGS;
```

## 2.2 서버리스 기능 활성화 (UI 단계)

서버리스 SQL 및 서버리스 컴퓨트는 **계정 콘솔에서 활성화**해야 합니다. 이 단계는 UI로만 가능하며 CLI로는 켤 수 없습니다.

1. 계정 콘솔(Account Console)에 관리자로 로그인합니다.
2. **Settings → Feature enablement**에서 서버리스 관련 기능을 켭니다.
3. 워크스페이스에 반영될 때까지 잠시 기다립니다.

> 이 단계는 CLI로 자동화할 수 없으므로, 배포 스크립트에 주석으로 "여기서 UI 단계 필요"라고 남겨두는 것을 권장합니다.

## 2.3 서버리스 SQL 웨어하우스 생성

서버리스 SQL 웨어하우스는 CLI 또는 REST API로 만들 수 있습니다. 두 방식을 병기합니다.

**CLI 방식:**

```bash
databricks warehouses create \
  --name "<warehouse-name>" \
  --warehouse-type PRO \
  --enable-serverless-compute true \
  --cluster-size "2X-Small"
```

**REST 방식:**

```bash
curl -X POST \
  https://adb-<workspace-id>.<shard>.azuredatabricks.net/api/2.0/sql/warehouses \
  -H "Authorization: Bearer $DATABRICKS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "<warehouse-name>",
    "warehouse_type": "PRO",
    "enable_serverless_compute": true,
    "cluster_size": "2X-Small"
  }'
```

> 토큰은 반드시 환경 변수(`$DATABRICKS_TOKEN`)로만 다루세요. 명령에 값을 직접 넣지 마세요.
