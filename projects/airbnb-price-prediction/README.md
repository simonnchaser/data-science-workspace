# Airbnb Price Prediction

뉴욕 Airbnb 숙소 데이터를 기반으로 가격 예측을 위한 전처리와 feature engineering을 수행한 프로젝트입니다.  
단순 숙소 메타데이터뿐 아니라 관광지 접근성, 지하철 접근성, 범죄 관련 지역 특성을 결합해 분석용 데이터셋을 구성했습니다.

## Project Goal

- Airbnb 숙소 가격에 영향을 줄 수 있는 지역 기반 요인을 반영한 분석용 데이터셋 구축
- 전처리, 결측치 처리, 이상치 처리, 로그 변환 과정을 체계적으로 정리
- 추후 회귀/머신러닝 모델링에 바로 활용할 수 있는 최종 테이블 생성

## Data

### Raw data

다음 원본 데이터들을 활용했습니다.

- `AB_NYC_2019.csv`
- `nyc_attractions_verified.csv`
- `nyc-transit-subway-entrance-and-exit-data.csv`

원본(raw) 데이터는 용량 및 관리 측면 때문에 Git에는 포함하지 않고, 로컬에서만 관리합니다.

### Processed data

- `data/processed/aribnb_final_df.csv`

최종 전처리 및 feature engineering이 반영된 분석용 데이터셋입니다.

- 행 수: 48,895
- 열 수: 45

포트폴리오에서 결과를 바로 확인할 수 있도록 processed 데이터는 저장소에 포함했습니다.

## Project Structure

```text
airbnb-price-prediction/
├── data/
│   ├── raw/
│   └── processed/
├── notebooks/
│   ├── airbnb_preprocessing.ipynb
│   ├── airbnb_feature_engineering.ipynb
│   └── 은수님_vif_airbnb.ipynb
└── README.md
```

## Workflow

### 1. Data preprocessing

`notebooks/airbnb_preprocessing.ipynb`

주요 작업:

- `name`, `host_name` 결측치 처리
- `last_review`, `reviews_per_month` 결측치 처리
- 범죄 비율 관련 결측치 처리
- 이상치 점검 및 처리
- 가격 변수 log 변환

### 2. Feature engineering

`notebooks/airbnb_feature_engineering.ipynb`

생성한 주요 파생 변수:

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
- `crime_count_0.5km`, `crime_count_1km`, `crime_count_2km`
- `felony_count_*`
- `misdemeanor_count_*`
- `night_crime_count_*`
- `felony_ratio_*`
- `misdemeanor_ratio_*`
- `night_crime_ratio_*`

## Final Dataset

최종 데이터셋 `aribnb_final_df.csv`에는 다음 범주의 변수가 포함됩니다.

- Airbnb 기본 listing 정보
- 위치 정보 (`latitude`, `longitude`, `neighbourhood_group`, `neighbourhood`)
- 숙소 운영 정보 (`room_type`, `minimum_nights`, `availability_365` 등)
- 리뷰 관련 정보 (`number_of_reviews`, `reviews_per_month`)
- 관광지, 지하철, 범죄 관련 파생 변수

대표 컬럼:

- `price`
- `attraction_count_2km`, `attraction_count_3km`
- `distance_to_city_center`
- `distance_to_nearest_station`
- `station_count_300m`, `station_count_500m`, `station_count_1km`
- `crime_count_0.5km`, `crime_count_1km`, `crime_count_2km`
- `felony_ratio_0.5km`, `misdemeanor_ratio_1km`, `night_crime_ratio_2km`

## How to Use

1. raw 데이터를 `data/raw/` 아래에 준비합니다.
2. `airbnb_preprocessing.ipynb`를 실행해 전처리를 수행합니다.
3. `airbnb_feature_engineering.ipynb`를 실행해 지역 기반 파생 변수를 생성합니다.
4. 결과물인 `data/processed/aribnb_final_df.csv`를 분석 또는 모델링에 사용합니다.

## Notes

- raw 데이터는 `.gitignore`로 제외되어 있습니다.
- processed 데이터는 포트폴리오 확인용으로 버전 관리합니다.
- 추후 모델링 결과가 추가되면 성능 비교와 해석 섹션을 별도로 확장할 수 있습니다.
