# Azure Databricks Claude 모델 운영 가이드

실무 중심 Azure Databricks 서버리스 배포 · Claude 모델 호출(serving / gateway) · 사용량·비용 모니터링 가이드입니다.

## 보기

- **온라인(GitHub Pages)**: https://yunkyoungaum.github.io/ADB_guide/
- **단일 HTML**: [`index.html`](./index.html) — 사이드바 목차, Mermaid 다이어그램, 다크/라이트 테마 포함

## 수정 방법

이 사이트는 단일 파일 `index.html`로 구성되어 있습니다. 추후 수정은 두 가지 경로가 있습니다.

1. **직접 편집** — `index.html`을 수정하고 `main` 브랜치에 커밋하면 GitHub Pages가 자동 재배포합니다.
   - GitHub 웹에서 `index.html` → 연필(✏️) 아이콘으로 바로 편집 가능
2. **소스 기준 편집** — `source/` 폴더에 장별 마크다운 원본이 있습니다. 내용을 여기서 관리한 뒤 `index.html`에 반영하세요.

## 구성

```
index.html            게시용 단일 HTML (본문)
source/               장별 마크다운 원본
  SUMMARY.md
  guides/00-overview ~ 06-dashboards
  appendix/           추론 테이블 · 검증 체크리스트 · 용어집
.nojekyll             Jekyll 처리 비활성화 (정적 HTML 그대로 게시)
```

> ⚠️ 초안(Draft). 식별자는 모두 placeholder로 마스킹되어 있습니다.
