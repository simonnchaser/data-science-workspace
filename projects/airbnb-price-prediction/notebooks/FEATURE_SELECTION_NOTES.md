# Airbnb Feature Selection Notes

`airbnb_feature_selection.ipynb`는 `airbnb_preprocessing.ipynb`에서 저장한 전처리 결과를 불러와, 모델 유형에 맞는 입력 피처를 고르는 과정을 정리한 노트북이다. 이 단계에서는 선형 회귀용 피처 세트와 트리 기반 모델용 피처 세트를 분리해 설계했다.

## 1. 입력 데이터

- 입력 파일: `../data/processed/airbnb_preprocessed_df.csv`
- 기준 데이터:
  - 결측치 처리 완료
  - 이상치 처리 완료
  - 로그 변환 컬럼 생성 완료
  - 파생변수 생성 완료

즉, feature selection 단계에서는 추가 전처리보다 "어떤 피처를 남기고 어떤 피처를 제외할지"에 집중했다.

## 2. 선형 모델용 피처 선택 원칙

선형 회귀 계열 모델은 다중공선성에 민감하기 때문에, 선형 모델용 피처는 비교적 보수적으로 골랐다.

### 공통 제거

- `id`, `host_id`, `price` 제거
- `price`는 타깃의 원본 값이므로 입력 피처에서 제외하고 `log_price`를 사용

### 범주형 처리

- `neighbourhood_group`, `room_type`는 `pd.get_dummies(..., drop_first=True)`로 더미화

### 1차 필터링

- `log_price`와의 상관계수를 확인
- 반경 기반 파생변수 중 이미 팀에서 선택한 후보를 우선 유지
- 상관계수가 매우 낮은 변수는 선형 모델용 후보에서 우선 제외

### 중복 변수 정리

- `is_manhattan`과 `neighbourhood_group_Manhattan`은 거의 같은 의미를 가지므로 둘 중 하나만 유지
- 원본/로그본이 같이 존재하는 변수는 선형 모델에서는 로그본 채택
  - `minimum_nights` -> `log_minimum_nights`
  - `calculated_host_listings_count` -> `log_calculated_host_listings_count`
  - `number_of_reviews` -> `log_number_of_reviews`
  - `reviews_per_month` -> `log_reviews_per_month`
  - `distance_to_nearest_station` -> `log_distance_to_nearest_station`

### 그룹 단위 점검

선형 모델용에서는 아래처럼 의미가 겹치는 변수들을 그룹으로 묶어 상관행렬과 VIF를 함께 검토했다.

- location
  - `attraction_count_3km`
  - `distance_to_city_center`
  - `neighbourhood_group_Manhattan`
- crime
  - `night_crime_ratio_2km`
  - `felony_count_2km`
  - `misdemeanor_ratio_2km`
- station
  - `station_count_1km`
  - `log_distance_to_nearest_station`
- room type
  - `room_type_Private room`
  - `room_type_Shared room`

이 과정의 목적은 상관계수 순위만 보는 데서 끝내지 않고, 비슷한 정보를 다른 방식으로 담고 있는 변수들을 함께 검토해 최종 세트를 안정화하는 데 있었다.

## 3. 선형 모델용 최종 피처 세트

선형 회귀용으로 저장한 데이터셋은 아래 피처들과 `log_price`로 구성했다.

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
- `log_price`

출력 파일:

- `../data/processed/airbnb_linear_features_df.csv`

추가 정리

- `log_price`와의 상관계수 절댓값이 `0.1`보다 낮은 변수는 선형 회귀형 모델의 핵심 후보군에서 우선 제외했다
- 초기 전체 후보군에서는 일부 `neighbourhood_group` 더미 변수의 공선성이 높게 나타났기 때문에, location theme을 별도로 재검토한 뒤 `neighbourhood_group_Manhattan`만 유지했다
- 최종 선형 회귀형 피처 세트는 해석 가능성과 다중공선성 관리에 초점을 맞춘 baseline용 세트다

## 4. 트리 모델용 피처 선택 원칙

트리 기반 모델은 선형 모델보다 다중공선성에 덜 민감하고, 비선형 관계와 구간 분할을 통해 변수를 활용할 수 있다. 그래서 트리 모델용 피처 세트는 선형 모델보다 정보 보존을 우선했다.

### 기본 방향

- `id`, `host_id` 제거
- `log_price`는 타깃으로 유지
- 입력 피처에서는 `log_price`를 제외한 `log_` 컬럼을 일단 제거
- 즉, 트리 모델에서는 원본 피처 중심으로 먼저 실험

### 왜 로그 피처를 일단 제외했는가

- 트리 모델은 스케일과 선형성에 덜 민감함
- 왜도가 큰 변수를 원본으로 넣어도 잘 작동하는 경우가 많음
- 따라서 1차 실험은 "원본 변수 중심 + 넓은 정보 보존" 전략으로 시작

## 5. 트리 모델용 피처 세트

트리 모델용 데이터셋은 다음과 같은 방향으로 구성했다.

- 범주형 원본 유지
  - `neighbourhood_group`
  - `room_type`
- 수치형 원본 유지
  - `minimum_nights`
  - `number_of_reviews`
  - `reviews_per_month`
  - `calculated_host_listings_count`
  - `availability_365`
- 위치 및 접근성 변수
  - `attraction_count_2km`, `attraction_count_3km`
  - `distance_to_city_center`
  - `is_manhattan`
  - `distance_to_nearest_station`
  - `station_count_300m`, `station_count_500m`, `station_count_1km`
- 범죄 및 반경 기반 변수
  - `crime_count_*`
  - `felony_count_*`
  - `misdemeanor_count_*`
  - `night_crime_count_*`
  - `felony_ratio_*`
  - `misdemeanor_ratio_*`
  - `night_crime_ratio_*`
- 타깃
  - `log_price`

출력 파일:

- `../data/processed/airbnb_tree_features_df.csv`

이후 최종 모델링 단계에서는 트리 기반 모델의 확장 피처 세트에 다음 정보가 함께 반영되었다.

- 파생변수
  - `night_crime_ratio_2km`
  - `misdemeanor_ratio_2km`
  - `felony_count_2km`
  - `attraction_count_3km`
  - `distance_to_city_center`
  - `distance_to_nearest_station`
  - `station_count_1km`
- 위치 및 숙소 특성
  - `latitude`
  - `longitude`
  - `availability_365`
- 로그 변환 변수
  - `log_minimum_nights`
  - `log_number_of_reviews`
  - `log_reviews_per_month`
  - `log_calculated_host_listings_count`
- 범주형 더미 변수
  - `room_type_Private room`
  - `room_type_Shared room`
  - `neighbourhood_group_Brooklyn`
  - `neighbourhood_group_Manhattan`
  - `neighbourhood_group_Queens`
  - `neighbourhood_group_Staten Island`

## 6. 정리

feature selection 단계의 핵심은 모델에 따라 피처를 다르게 다루는 것이었다.

- 선형 모델:
  - 상관계수와 다중공선성을 더 엄격하게 반영
  - 해석 가능한 소수의 피처 중심
- 트리 모델:
  - 상관계수만으로 변수 제거하지 않음
  - 더 넓은 후보를 유지하고 모델 중요도로 다시 판단

이 분리를 통해 같은 전처리 결과를 기반으로 하되, 모델 특성에 맞는 입력 데이터를 따로 설계할 수 있었다.
