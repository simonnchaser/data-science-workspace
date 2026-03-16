# Airbnb Price Prediction

뉴욕 Airbnb 숙소 데이터를 기반으로 가격 예측을 수행한 프로젝트다. 단순 listing 정보뿐 아니라 관광지 접근성, 지하철 접근성, 범죄 관련 지역 특성을 결합해 파생변수를 만들고, 선형 회귀형 모델과 트리 기반 비선형 모델을 비교하였다.

## Project Goal

- Airbnb 숙소 가격에 영향을 주는 지역 기반 요인을 반영한 데이터셋 구축
- 전처리, feature engineering, feature selection 과정을 체계적으로 정리
- 선형 회귀형 모델과 비선형 트리 기반 모델을 비교해 최종 예측 모델 선정

## Data

### Raw data

주요 원본 데이터

- `AB_NYC_2019.csv`
- `nyc_attractions_verified.csv`
- `nyc-transit-subway-entrance-and-exit-data.csv`

raw 데이터는 `.gitignore`로 제외하고 로컬에서만 관리한다.

### Processed data

- `data/processed/aribnb_final_df.csv`
  - 전처리 및 feature engineering 결과
- `data/processed/airbnb_preprocessed_df.csv`
  - 전처리 결과 저장본
- `data/processed/airbnb_linear_features_df.csv`
  - 선형 회귀형 모델용 피처 세트
- `data/processed/airbnb_tree_features_df.csv`
  - 트리 기반 모델용 피처 세트

포트폴리오에서 결과를 바로 확인할 수 있도록 processed 데이터는 저장소에 포함했다.

## Project Structure

```text
airbnb-price-prediction/
├── data/
│   ├── raw/
│   └── processed/
├── notebooks/
│   ├── airbnb_preprocessing.ipynb
│   ├── airbnb_feature_engineering.ipynb
│   ├── airbnb_feature_selection.ipynb
│   ├── airbnb_modeling.ipynb
│   ├── 최종 airbnb.ipynb
│   ├── 최종_모델링_airbnb.ipynb
│   ├── FEATURE_ENGINEERING_NOTES.md
│   ├── PREPROCESSING_NOTES.md
│   ├── FEATURE_SELECTION_NOTES.md
│   └── MODELING_NOTES.md
└── README.md
```

## Workflow

### 1. Preprocessing

`notebooks/airbnb_preprocessing.ipynb`

주요 작업

- 결측치 처리
- 이상치 제거
- 로그 변환 컬럼 생성
- 전처리 결과 저장

### 2. Feature engineering

`notebooks/airbnb_feature_engineering.ipynb`  
`notebooks/최종 airbnb.ipynb`

생성한 주요 파생변수

- 관광지 접근성
  - `attraction_count_2km`
  - `attraction_count_3km`
  - `distance_to_city_center`
  - `is_manhattan`
- 지하철 접근성
  - `distance_to_nearest_station`
  - `station_count_300m`
  - `station_count_500m`
  - `station_count_1km`
- 범죄 관련 변수
  - `crime_count_*`
  - `felony_count_*`
  - `misdemeanor_count_*`
  - `night_crime_count_*`
  - `felony_ratio_*`
  - `misdemeanor_ratio_*`
  - `night_crime_ratio_*`

### 3. Feature selection

`notebooks/airbnb_feature_selection.ipynb`

- 선형 회귀형 모델용 피처 세트 구성
- 트리 기반 모델용 피처 세트 구성
- 상관계수, VIF, theme별 공선성 검토
- 모델 유형에 따라 별도 데이터셋 저장

선형 회귀형 모델용 최종 피처

- `attraction_count_3km`
- `distance_to_city_center`
- `neighbourhood_group_Manhattan`
- `night_crime_ratio_2km`
- `felony_count_2km`
- `misdemeanor_ratio_2km`
- `station_count_1km`
- `log_distance_to_nearest_station`
- `room_type_Private room`
- `room_type_Shared room`

### 4. Modeling

`notebooks/airbnb_modeling.ipynb`  
`notebooks/최종_모델링_airbnb.ipynb`

비교 모델

- 선형 회귀형 모델
  - `Linear Regression`
  - `Ridge`
- 트리 기반 비선형 모델
  - `Gradient Boosting`
  - `XGBoost`
  - `LightGBM`

검증 방식

- `Train 80% / Test 20%`
- `5-Fold Cross Validation`
- `GridSearchCV` 기반 하이퍼파라미터 탐색

## Modeling Results

최종 모델링 단계의 주요 결과

| 모델 | Best CV Score | Test RMSE | Test MAE | Test R² |
| --- | ---: | ---: | ---: | ---: |
| LightGBM | 0.6507 | 0.3773 | 0.2844 | 0.6556 |
| XGBoost | 0.6468 | 0.3798 | 0.2869 | 0.6509 |
| Gradient Boosting | 0.6394 | 0.3851 | 0.2918 | 0.6412 |
| Ridge | 0.5766 | 0.4172 | 0.3189 | 0.5789 |
| Linear Regression | 0.5766 | 0.4172 | 0.3189 | 0.5789 |

정리

- `Linear Regression`은 baseline 및 해석용 모델로 활용
- `LightGBM`은 최종 예측 성능 모델로 선정
- `XGBoost`는 LightGBM과 유사한 성능을 보인 차선 모델

## Notes

- raw 데이터는 `.gitignore`로 제외되어 있다
- processed 데이터는 포트폴리오 확인용으로 버전 관리한다
- 각 단계별 설명은 아래 문서에 정리했다
  - `notebooks/FEATURE_ENGINEERING_NOTES.md`
  - `notebooks/PREPROCESSING_NOTES.md`
  - `notebooks/FEATURE_SELECTION_NOTES.md`
  - `notebooks/MODELING_NOTES.md`
