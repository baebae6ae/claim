# Claim Analysis System (클레임 분석 시스템)

**식품 제조업 소비자 클레임 데이터 분석 및 시각화 플랫폼**

---

## 🎯 프로젝트 개요

### 목적
식품 제조업체의 클레임(소비자 민원) 데이터를 실시간으로 수집, 분석, 시각화하여 품질 관리(QC) 의사결정을 지원하는 통합 분석 시스템입니다.

### 주요 특징
- **다차원 필터링**: 제조업체, 분류(1~3차), 품목군, 제품명 기준 동적 분석
- **시계열 통계**: ARIMA 자동 모델링, 이상치 탐지, 계절성 분석
- **실시간 시각화**: Plotly 기반 이중축 차트, 전년 비교 대시보드
- **AI 심화 해석**: Perplexity Pro API 연동 자동 분석 리포트 생성
- **품질이슈 추적**: 제조-접수 리드타임 분석, 이벤트/변경 이력 관리
- **PPM 계산**: 출고량 기반 정규화 지표(PPM, Parts Per Million) 자동 계산

---

## 📊 시스템 아키텍처 (현재 & 계획)

### 현재 구조 (Python Gradio 기반)

```
claim.py
├── 📁 Data Layer
│   ├── load_base_df(): 클레임 엑셀 로드 및 제조일자 계산
│   ├── load_output_volume(): 출고량 엑셀 읽기
│   └── 마스킹 유틸 (토큰화): 민감 정보 보호
│
├── 📁 Analysis Layer
│   ├── 시계열 통계 (_compute_trend_stats, _classify_trend)
│   ├── 수준 비교 (_recent_vs_past_level)
│   ├── 이상치 탐지 (_detect_outliers_iqr_recent)
│   ├── 리드타임 분석 (_compute_delay_stats)
│   └── 핵심 KPI 계산 (_compute_core_kpis)
│
├── 📁 Visualization Layer
│   ├── overview_analyze(): 월별 건수/PPM 그래프
│   ├── 파이 차트 (2차분류, 3차분류별)
│   └── 리드타임 히스토그램
│
├── 📁 AI Insight Layer
│   └── call_perplexity_pro(): 자동 분석 리포트 생성
│
└── 📁 Event Management
    ├── 품질이슈/변경 이력 CRUD
    └── JSON 기반 저장 (events.json)
```

## 🔧 기술 스택

### 현재 (Python)
| 카테고리 | 라이브러리 | 용도 |
|---------|----------|------|
| **UI** | Gradio | 웹 인터페이스 |
| **데이터** | Pandas, NumPy | 데이터 처리 및 집계 |
| **분석** | Statsmodels, PMDarima | 시계열 모델링(ARIMA), 자기상관 |
| **시각화** | Plotly | 대화형 차트(이중축, 히스토그램) |
| **API** | requests | Perplexity Pro API 연동 |


---

## 📈 핵심 분석 지표 (KPI)

### 1. 기술 통계 (Descriptive)
- **월평균**: 선택 기간의 월별 클레임 건수 평균
- **표준편차 & 변동계수(CV)**: 월별 변동성 크기
- **최대/최소**: 최악의 달과 최선의 달

### 2. 추세 분석 (Trend)
- **선형 추세 기울기(slope)**: 장기 상승/하강/안정 판정
- **기울기 표준화(slope_std)**: 추세 강도 평가
- **분류**: 안정 / 증가 추세 / 감소 추세

### 3. 수준 비교 (Level)
- **최근 6개월 vs 과거 6개월**: 최근 변화 방향 파악
- **동월/누적 YoY 비교**: 전년 동기 대비 성장률

### 4. 이상 신호 (Anomaly)
- **IQR 기반 이상치 탐지**: 최근 12개월 기준 비정상 월 식별
- **이상치 기준값(threshold)**: 그 이상의 값은 비정상

### 5. 시간 관계 (Temporal)
- **자기상관(ACF lag=1)**: 전월 수준이 다음 달에 미치는 영향 (0~1, 높을수록 관성 강함)

### 6. 계절성 (Seasonality)
- **진폭비(amplitude ratio)**: 월별 평균의 최대-최소 차이 / 평균 (≥0.5면 계절성 있음)

### 7. 리드타임 (Lead Time)
- **평균 리드타임**: 제조일자 → 접수일자 평균 지연일수
- **분포**: 평균, 중앙값, 사분위수, 집중 구간

### 8. 출고량 정규화
- **PPM (Parts Per Million)**: 클레임 건수 / 출고량 × 1,000,000
  - 예: 출고 100만 개 중 클레임 5건 = 5 PPM

---

## 🎨 UI/UX 플로우

### 1. 대시보드 (Overview)
```
[필터 패널] (상단 고정)
├── 제품/상품 선택 (라디오)
├── 제조업체 (드롭다운)
├── 1/2/3차분류 (계단식 드롭다운)
├── 품목군 (멀티 선택)
├── 제품명 (멀티 선택)
├── 기간 선택 (시작일~종료일)
└── 단위 선택 (건수 / PPM)

[메인 콘텐츠 영역]
├── [상단] KPI 카드 요약
│   ├ 총 건수 / 평균 / 표준편차 / CV
│   ├ 최대값(월) / 최소값(월)
│   └ 추세 판정 (증가/감소/안정)
│
├── [좌측] 월별 건수/PPM 이중축 차트 (전년 비교)
├── [우측 상] 2차분류별 파이 차트
├── [우측 중] 3차분류별 파이 차트
│
├── [하단] 제조업체별 요약 테이블
└── [하단] 월별 상세 테이블 (동월/누적 YoY)
```

### 2. 심화 분석 페이지
```
[분석 그룹 선택]
└── 동일한 필터 적용

[AI 분석 결과] (Perplexity Pro)
├── 1. 전체 수준 및 변동 구조 해석
├── 2. 추세 및 이상 신호 해석
├── 3. 공정 관점 해석
└── 4. 관리 포인트 3가지

[상세 통계 시각화]
├── 리드타임 분포 (히스토그램)
├── 시계열 그래프 (이상치 표시)
└── 자기상관 플롯 (ACF)
```

### 3. 이슈 관리 페이지
```
[이슈 등록 폼]
├── 날짜 (필수)
├── 설명 (필수)
├── 제조업체 (선택)
├── 1/2/3차분류 (선택)
├── 품목군 (다중)
└── 제품명 (다중)

[이슈 목록] (필터 적용)
├── 날짜순 정렬
├── 편집/삭제 가능
└── 차트에서 이벤트 마커로 표시
```

---

## 📋 주요 함수 맵

| 함수명 | 입력 | 출력 | 용도 |
|--------|------|------|------|
| `load_base_df()` | 엑셀 경로 | DataFrame | 클레임 데이터 로드 + 제조일자 계산 |
| `load_output_volume()` | 폴더 경로 | DataFrame | 월별 출고량 로드 |
| `get_category_values()` | DataFrame, 컬럼명 | list | 드롭다운 옵션 동적 생성 |
| `overview_analyze()` | 필터 조건 | (그래프, 테이블, HTML) | 개요 분석 |
| `analyze()` | 필터 조건 | (그래프, KPI, 분석문) | 심화 분석 + AI 해석 |
| `_compute_core_kpis()` | 시계열 데이터 | dict | 핵심 KPI 일괄 계산 |
| `call_perplexity_pro()` | KPI dict | str | AI 자동 분석 리포트 생성 |
| `add_event()` | 이슈 정보 | str | 이벤트 저장 |
| `refresh_event_list()` | 필터 조건 | (list, list) | 필터된 이벤트 목록 조회 |

---

## 🚀 설치 및 실행

### 현재 (Gradio 버전)

```bash
# 1. 환경 설정
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 2. 패키지 설치
pip install pandas numpy gradio statsmodels pmdarima plotly requests

# 3. 환경 변수 설정
export PPLX_API_KEY="your-perplexity-api-key"

# 4. 실행
python claim.py
```

### 계획 (FastAPI + React)

```bash
# 백엔드
cd backend
pip install fastapi uvicorn sqlalchemy psycopg2-binary pydantic requests

# 프론트엔드
cd frontend
npm install
npm start

# 배포 (Docker)
docker-compose up -d
```

---

## 🔐 보안 및 민감정보 관리

### 토큰화(Masking) 시스템
- **제조업체**: M001, M002, ... (실제 이름 숨김)
- **제품명**: P001, P002, ... (실제 이름 숨김)

### API 요청 시 적용
```python
# 원본 데이터: "ㅇㅇ공장", "우유팩"
# API 전송: manufacturer="M001", product="P001"

# Perplexity Pro 분석 시 토큰만 사용
# → 외부 AI 서비스에 민감 정보 노출 방지
```

---

## 📊 분석 리포트 예시 (Perplexity Pro 출력)

```markdown
## 전체 수준 및 변동 구조 해석

선택된 그룹의 월평균 클레임은 12.5건으로, 월별 표준편차 4.2건(변동계수 0.34)를 기록했습니다...

## 추세 및 이상 신호 해석

장기 추세는 경미한 증가 신호(월 0.3건 상승)를 보이며, 최근 12개월에는 2024-05월(22.1건)과 2024-08월(19.8건)이 이상치로 판정되었습니다...

## 공정 관점 해석

제조-접수 리드타임이 평균 45일로 비교적 길어, 보관·유통 환경(온습도, 진동) 관리 강화 필요...

## 관리 포인트

1. **월별 변동성 개선**: CV 0.34는 중간 수준. 분산 원인(제조사별/제품별) 분석 필요
2. **이상월 원인 파악**: 5월, 8월 피크 시점의 원인(신제품 출시? 원료 변경?) 조사
3. **리드타임 단축**: 45일 → 30일 목표로 유통망 최적화 권고
```


## 📞 연락처 및 지원

- **개발자**: [배준영[
- **문의**: [bjunyeong@sempio.com]

---


**Last Updated**: 2026-01-02
**Version**: 1.0 (Prototype)
