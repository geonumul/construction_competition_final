# 건설 산업재해 분석: 내부 안전관리·현장 안전행동과 외부기관 조절효과

제13회 안전보건 논문공모전 출품작.

## 연구 질문 (RQ)

> 건설현장의 내부 안전 관리(A)와 실질적 안전 행동(B)이 산업재해 발생에 미치는 영향 — **외부 기관의 조절효과를 중심으로**

## 데이터

- 출처: **제10차 산업안전보건 실태조사 (2021, 건설업)** — 한국산업안전보건공단 산업안전보건연구원
- 원자료: 1,502개 사업장 (공사금액 50억 이상)
- 최종 분석 표본: **1,375개 사업장 × 17개 변수** (전처리 완료, 박천수 2023·2024·2025와 동일한 Listwise Deletion 절차)
- 원자료: `data/제10차 산업안전보건 실태조사_raw data_건설업_230824.CSV`
- 분석용 파일: `data/전처리_최종.csv` (재현은 `notebooks/01_전처리.ipynb`)

### 변수 구성 (17개)

| 구분 | 변수 (수) | 비고 |
|---|---|---|
| 종속변수 | 사고발생 (1) | 0/1 이진 |
| 독립 A (내부 안전관리) | 안전조직수준, 위원회수준, 인증보유 (3) | 0~1 또는 0/1 |
| 독립 B (현장 안전행동) | 위험성평가수준, 교육훈련도움, 정리정돈상태, 작업중지권, 작업반장기여 (5) | 0~2 또는 1~5 리커트 |
| 조절변수 | 전문지도, 고용노동부감독, 안전보건공단지원 (3) | 0/1 |
| 통제변수 | 공사규모, 발주처, 기성공정률, 공사종류, 외국인비율 (5) | 명목형 4 + 연속형 1 |

## 분석 단계

1. **Phase 1 — EDA**: 기술통계, χ²/t-test, VIF, 분포·상관 시각화
2. **Phase 2 — 전처리**: Train/Test 8:2 stratified split, 명목형 더미 변환(총 15개), LR용 Z-score 표준화, SMOTENC (학습셋 한정, ImbPipeline으로 CV fold 내부 적용)
3. **Phase 3 — 위계적 로지스틱 회귀**: M1(통제) → M2(+A) → M3(+B) → M4(+조절) → M5(+상호작용 24개), 우도비 검정·M5 주효과·24쌍 조절효과
4. **Phase 4 — 머신러닝**: LR / RandomForest / XGBoost / LightGBM, 5-Fold Stratified CV + GridSearchCV(scoring=f1), ROC·PR-AUC 포함
5. **Phase 5 — SHAP**: 전체 27변수 학습 단일 모델 기반
   - **Figure 4**: SHAP 평균 절댓값 변수 중요도 막대그래프 (전체 27변수, 통제 vs 독립·조절 시각 구분)
   - **Figure 5(a)**: Summary Plot — 전체 27변수
   - **Figure 5(b)**: Summary Plot — 통제변수 **시각적 마스킹** (독립·조절 11변수만, *재학습 없음*)
   - **Figure 6**: 정리정돈상태 SHAP Dependence
   - **Figure 7**: SHAP 상호작용 히트맵 (독립 8 × 조절 3)
   - 추가: Permutation Importance × SHAP × LR p-value 삼중 비교

## 키 페이퍼와 차별점

**키 페이퍼**: Qurat Ul Ain, S., & Rather, K. U. I. (2025). *Annals of Epidemiology*, **108**, 85-91.

본 연구는 키 페이퍼의 LR + ML + SHAP 삼중 검증 프레임워크를 따르되 다음을 확장·차별화:

1. **위계적 LR (M1~M5)**: 통제→A→B→조절→상호작용 24쌍 단계별 검정
2. **단일 모델 기반 다층 SHAP 해석**: 전체 변수로 학습한 단일 모델의 SHAP 값을, 가시성 목적으로만 컬럼 필터링하여 표시 (Figure 5(b)). 키 페이퍼처럼 부분집합으로 재학습 X — 통제변수 효과의 안전관리 변수로의 흡수·과대평가를 회피하면서 해석 가시성 확보.

## 폴더 구조

```
construction_competition_final/
├── README.md
├── .gitignore
├── requirements.txt
├── data/
│   ├── 제10차 산업안전보건 실태조사_raw data_건설업_230824.CSV  (원자료)
│   └── 전처리_최종.csv                                          (분석용)
├── notebooks/
│   ├── 01_전처리.ipynb         (원자료 → 전처리_최종.csv 재현)
│   └── 02_데이터분석.ipynb     (Phase 1~5 전체)
├── docs/
│   ├── 데이터분석_과정_및_근거.md
│   ├── 전처리_근거_및_과정.md
│   └── 참고논문_정리.md
└── results/
    ├── tables/
    └── figures/
```

## 실행 방법

```bash
# 1. 가상환경 (선택)
python -m venv .venv
.venv\Scripts\activate          # Windows
# source .venv/bin/activate     # macOS/Linux

# 2. 의존성 설치
pip install -r requirements.txt

# 3. 노트북 실행 (전처리 재현은 선택)
jupyter notebook notebooks/01_전처리.ipynb   # 원자료 → 전처리_최종.csv (생략 가능, 결과 포함됨)
jupyter notebook notebooks/02_데이터분석.ipynb
```

## 재현성

- 전역 `random_state = 42`
- Train/Test 8:2 stratified split
- SMOTENC는 학습셋에만, CV는 `imblearn.pipeline.Pipeline` 으로 fold 내부 적용 (데이터 누수 방지)
- 모든 표·그림 저장: `dpi=300`, `bbox_inches='tight'`

## 참고

- 박천수 (2023). *보건사회연구*, 43(4).
- 박천수 (2024). *노동정책연구*, 24(4). (동일 원자료, 위험성평가 영향 요인)
- 박천수 (2025). *노동정책연구*, 25(4). (동일 원자료, 안전보건사고 영향 요인)
- Reason, J. (1990, 2000). Swiss Cheese Model.
- Levine, Toffel, & Johnson (2012). *Science*, 336(6083) — 감독의 선택편향 보정 증거.
