## 프로젝트 개요

- **목표**: 발전소 개별 호기(3~6호기)의 운전 데이터로부터, **석회석 사용량이 SOx 저감에 얼마나 영향을 미치는지 예측**하고, 주어진 조건(Inlet SOx)에 따라 **최적 석회석 사용량을 자동 산출**하는 알고리즘 개발
- **핵심 성과**: Gradient Boosting Regressor 기반 SoxDiff 예측 모델 구축 및 최적 석회석 산출 함수 구현

---

## 1. 데이터 전처리

### 1) **데이터셋 구성**

- 3~6호기 별 운전 데이터셋 로드 (`No3_data`, `No4_data`, `No5_data`, `No6_data`)
- 각 호기별 데이터에는 발전량, Inlet SOx, Outlet SOx, 석회석 사용량, 배출여부 등이 포함됨

### 2) **형식 통일 및 결측치 처리**

- `datetime` 형식 정리 및 수치형 변수 타입 변환
- 결측치는 평균값으로 대체

### 3) **이상치 제거**

![outlier](https://github.com/user-attachments/assets/f2c9f544-d7a2-4790-a536-60e70ce07aac)

- 발전량 0 또는 1인 경우, SOx 값이 0인 경우 제거
- Outlet SOx에 대해 IQR 기반의 이상치 제거

---

## 2. EDA (탐색적 데이터 분석)

### 1) **분포 분석**

- KDE Plot, Histogram을 통해 각 호기별 Inlet/Outlet SOx 분포 확인
- Outlet SOx가 기준치(예: 15, 25, 48 ppm) 미만인 경우를 별도 시각화

### 2) **SOx 감소량(SoxDiff) 생성**

- `SoxDiff = Inlet SOx - Outlet SOx` 컬럼 생성
- SoxDiff 분포 통계량 및 표준편차 확인 → 최적 조건 추론에 활용

### 3) **Inlet SOx 구간별 석회석 평균 사용량**

- Inlet SOx를 구간으로 나눈 후 각 구간별 석회석 평균 사용량 계산
![input](https://github.com/user-attachments/assets/a54ffc25-92d4-496e-99d7-8dfeebcfde76)



---

## 3. 모델: Gradient Boosting Regressor

### 1) **입력 변수 / 타겟 정의**

- 입력(X): 발전량, Inlet SOx, 석회석 사용량
- 타겟(y): SoxDiff

### 2) **학습 및 평가**

- train_test_split으로 데이터 분할 후 GBR 모델 학습
- MSE = 73.76, R² = 0.974로 **우수한 예측 성능 확보**

### 3) **GridSearchCV를 통한 하이퍼파라미터 튜닝**
![gridsearch](https://github.com/user-attachments/assets/40d1332b-f341-4fd2-8335-e3fd166bdd0d)


- `max_depth`, `learning_rate`, `n_estimators`, `min_samples_split` 등 그리드 서치 수행
- 최적 파라미터 기반으로 모델 재학습 → Cross-Validation R² = 0.969

---

## 4. 최적 석회석 산출 로직

### 1) **조건 기반 석회석 초기값 추정**

- Inlet SOx 구간별 석회석 평균값에서 일정값 차감해 초기값 설정

### 2) **최적값 반복 탐색 함수**

- 초기 석회석 값에서 시작하여 예측된 SoxDiff의 변화가 기준(threshold)보다 작아질 때까지 반복
- **조건 만족 시점의 석회석 투입량을 최적값으로 반환**



### 3) **사용자 입력 기반 반복 실행**

- 사용자가 Inlet SOx 값을 입력하면, 해당 값에 대한 최적 석회석 사용량을 자동 계산

### 4) 결과 예시

- 입력값: `Inlet SOx = 200`

![resultsample](https://github.com/user-attachments/assets/3e5d450e-b6b5-4550-93c2-d046b58e93d2)


- 도출 결과: `Optimal Limestone = 6.74`, `예측 SoxDiff = 197.57`

---

## 사용 기술 및 라이브러리

| 구분 | 기술 |
| --- | --- |
| 데이터 전처리 | `pandas`, `numpy`, `matplotlib`, `seaborn` |
| 이상치 제거 | IQR 기반 방식 |
| 모델 학습 | `GradientBoostingRegressor` from `sklearn.ensemble` |
| 모델 평가 | `mean_squared_error`, `r2_score` from `sklearn.metrics` |
| 최적화 기법 | 반복 예측 기반 탐색 알고리즘 |
| 모델 튜닝 | `GridSearchCV` from `sklearn.model_selection` |

---

## 5. 배운 점

- **모델링과 공정 도메인 지식의 결합**
    - 단순 예측에 그치지 않고, **실제 운전 조건에서 의미 있는 최적치를 유도하는 알고리즘**을 설계함
    - 도메인 기반 구간 분할과 결합된 분석이 모델 해석력 향상에 크게 기여하는 점을 배움
- **회귀 모델의 실전 적용 경험**
    - Gradient Boosting 기반 회귀 모델을 통해 예측 정확도를 확보했고, GridSearchCV로 하이퍼파라미터 튜닝을 수행하며 **모델 최적화 경험을 쌓음**
- **현장 활용 가능한 알고리즘 설계**
    - 사용자 입력에 따라 최적 석회석 사용량을 실시간으로 반환하는 함수형 알고리즘 구현
    - **모델 → 적용 → 해석 → 반복 개선**의 선순환 흐름 경험
