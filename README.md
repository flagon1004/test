---
name: daily-market-report
description: 국내 주식 일일 시황 분석 리포트를 HTML 파일로 생성하는 스킬. 사용자가 "일일 시황 만들어줘", "오늘 시황 리포트", "금일 증시 분석", "금일 시황 HTML", "daily market report" 등의 말을 하면 반드시 이 스킬을 사용할 것. Claude가 웹 검색으로 실시간 데이터를 수집하고, 다크 테마 금융 대시보드 스타일의 HTML 리포트를 생성한다. 한국어 명령에도 반드시 트리거할 것.
---

# 주간 시황 리포트 스킬

사용자가 국내 주식 일일 시황 리포트를 요청하면, 웹 검색으로 실시간 데이터를 수집하고 다크 테마 금융 대시보드 HTML을 생성한다.

## 워크플로우

### 1단계: 대상 일자 확인
사용자가 특정 날짜를 언급하지 않으면 **현재 일자** 를 기준으로 한다. 오늘 날짜를 기반한다.

### 2단계: 데이터 수집 (웹 검색)
아래 항목을 순서대로 검색한다. 검색 쿼리 예시를 참고하되, 실제 날짜를 반영해 조정한다.

| 항목 | 검색 쿼리 예시 |
|------|--------------|
| 코스피 일일 등락 | `코스피 일일 등락 [날짜]` |
| 코스닥 주간 등락 | `코스닥 일일 등락 [날짜]` |
| 주간 거래대금/외국인·기관 동향 | `코스피 외국인 기관 순매수 일일` |
| 주도 섹터 | `코스피 섹터별 등락 일일 [날짜]` |
| 주목 종목 (급등/급락/테마) | `코스피 일일 급등주 테마주 [날짜]` |
| 주요 뉴스 (증시 영향) | `한국 증시 일일 뉴스 [날짜]` |

검색은 최소 4회 이상 수행한다. 데이터가 불충분하면 추가 검색한다.

### 3단계: 데이터 정리
수집한 데이터를 아래 구조로 정리한다. 수치는 실제 검색 결과를 사용하고, 확인되지 않은 수치는 "확인불가"로 표기한다.

```
period: "3월 16일 (3/16)"
indices:
  kospi: { start: X, end: X, change: X, pct: X }
  kosdaq: { start: X, end: X, change: X, pct: X }
  daily: 각 종가 배열
flow:
  foreign: "순매수/순매도 X억원"
  institution: "순매수/순매도 X억원"
sectors: (상위 3개 강세 + 상위 2개 약세)
  - { name, pct, summary }
stocks: (주목 종목 4~6개)
  - { name, code, pct, reason }
news: (3~5건)
  - { title, summary, impact }
summary: "한 문단 핵심 요약"
```

### 4단계: HTML 생성
아래 디자인 스펙에 맞춰 단일 HTML 파일을 생성한다.

자세한 HTML 템플릿 → `references/html-template.md` 참고

---

## 디자인 스펙

### 기본 설정
- **테마**: 다크 모드 기본 (`data-theme="dark"`), 라이트 모드 토글 가능
- **폰트**: `DM Serif Display` (헤더) + `IBM Plex Mono` (수치/레이블) + `Noto Sans KR` (본문)
- **배경**: `#0a0b0e` (최외곽) → `#111318` (카드) → `#1a1d26` (내부)
- **강조색**: `#c8a96e` (골드, 메인 액센트)
- **상승**: `#e05c5c` (빨강), **하락**: `#4caf82` (초록) ← 한국 증시 기준

### CSS 변수 (필수)
```css
:root {
  --bg: #0a0b0e; --surface: #111318; --surface2: #1a1d26;
  --border: #2a2d3a; --accent: #c8a96e; --accent2: #4fc3f7;
  --up: #e05c5c; --down: #4caf82;
  --text: #e8e6e0; --muted: #7a7f8e; --dim: #4a4f5e;
  --radius: 12px;
}
```

### 레이아웃 구조
```
<header> 스티키 네비 (로고 + 섹션 점프 버튼 + 테마 토글)
<section id="s1"> 일일 개요 (지수 요약 카드)
<section id="s2"> 섹터 분석
<section id="s3"> 주목 종목
<section id="s4"> 주요 뉴스
<section id="s5"> 오늘의 총평
```

### 섹션 헤더 패턴 (통일)
```html
<div class="section-header">
  <div class="sec-icon">[이모지]</div>
  <div>
    <div class="sec-title-ko">[한글 제목]</div>
    <div class="sec-title-en">[IBM Plex Mono 영문 서브]</div>
  </div>
</div>
```

| 섹션 | 이모지 | 한글 | 영문 |
|------|--------|------|------|
| 개요 | 📊 | 일일 지수 요약 | daily Indices |
| 섹터 | 🏭 | 섹터 분석 | Sector Analysis |
| 종목 | ⚡ | 주목 종목 | Notable Stocks |
| 뉴스 | 📰 | 주요 뉴스 | News Digest |
| 총평 | 📋 | 금일 총평 | daily Summary |

---

## 파일명 규칙
`일일시황리포트_YYYYMMDD_MMDD.html`  
예: `일일시황리포트_20260310_0314.html`

---

## 품질 체크리스트
HTML 생성 전 반드시 확인:
- [ ] 수치는 모두 실제 검색 결과 기반 (추측 금지)
- [ ] 코스피/코스닥 일별 데이터 5개 포인트 확보
- [ ] 섹터 최소 3개 이상
- [ ] 종목 최소 4개 이상
- [ ] 뉴스 최소 3건 이상
- [ ] Chart.js CDN 포함
- [ ] 라이트/다크 토글 동작
- [ ] 스티키 네비 + 섹션 점프 동작
- [ ] 파일명 날짜 정확히 반영
