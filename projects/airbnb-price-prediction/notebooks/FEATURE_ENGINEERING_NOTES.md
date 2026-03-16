# Airbnb Feature Engineering Notes

`airbnb_feature_engineering.ipynb`는 Airbnb 원본 listing 데이터에 관광지, 지하철, 범죄 데이터를 결합해 지역 기반 파생변수를 생성하는 노트북이다. 이 단계의 목적은 단순 숙소 메타데이터만으로 설명하기 어려운 입지 특성과 주변 환경 정보를 수치화해, 이후 전처리와 모델링 단계에서 활용할 수 있는 분석용 컬럼을 만드는 데 있었다.

## 1. 입력 데이터

사용한 데이터는 크게 네 종류다.

- Airbnb listing 데이터
- 관광지 좌표 데이터
- 지하철역 좌표 데이터
- 범죄 발생 좌표 데이터

노트북에서는 Airbnb 숙소의 위도와 경도를 기준으로, 각 외부 데이터와의 거리 또는 반경 내 개수를 계산하는 방식으로 파생변수를 생성했다.

## 2. 관광지 관련 파생변수

관광지 데이터의 좌표를 활용하여 각 숙소 주변의 관광지 접근성을 수치화했다.

생성 변수:

- `attraction_count_2km`
- `attraction_count_3km`
- `distance_to_city_center`
- `is_manhattan`

### 생성 방식

- `attraction_count_2km`, `attraction_count_3km`
  - 각 숙소 좌표를 기준으로 관광지 좌표까지의 Haversine 거리를 계산
  - 반경 2km, 3km 안에 포함되는 관광지 개수를 집계

- `distance_to_city_center`
  - Times Square 좌표를 도시 중심점으로 설정
  - 각 숙소에서 Times Square까지의 거리를 계산

- `is_manhattan`
  - `neighbourhood_group == 'Manhattan'` 여부를 0/1 이진 변수로 변환

## 3. 지하철 접근성 파생변수

지하철역 데이터는 역 이름과 위도, 경도 기준으로 중복을 제거한 뒤 사용했다.

생성 변수:

- `distance_to_nearest_station`
- `station_count_300m`
- `station_count_500m`
- `station_count_1km`

### 생성 방식

- `distance_to_nearest_station`
  - 각 숙소에서 모든 지하철역까지의 거리를 계산
  - 가장 가까운 역까지의 최소 거리를 저장

- `station_count_300m`, `station_count_500m`, `station_count_1km`
  - 각 숙소를 기준으로 반경 300m, 500m, 1km 내에 포함되는 역 개수를 계산

이 변수들은 단순히 "역이 가까운지"뿐 아니라 "주변 역 밀도가 얼마나 높은지"를 함께 반영하기 위한 목적에서 생성되었다.

## 4. 범죄 관련 파생변수

범죄 데이터는 `BallTree`와 Haversine 거리 기반 반경 탐색을 이용해 계산했다.

생성 변수는 총 세 부류로 나뉜다.

### 4.1 전체 범죄 건수

- `crime_count_0.5km`
- `crime_count_1km`
- `crime_count_2km`

각 숙소 반경 0.5km, 1km, 2km 안에서 발생한 전체 범죄 건수를 집계했다.

### 4.2 범죄 유형별 건수

- `felony_count_0.5km`
- `felony_count_1km`
- `felony_count_2km`
- `misdemeanor_count_0.5km`
- `misdemeanor_count_1km`
- `misdemeanor_count_2km`
- `night_crime_count_0.5km`
- `night_crime_count_1km`
- `night_crime_count_2km`

범죄 유형별로 데이터를 나누어 반경별 건수를 계산했다.

- `felony_count_*`
  - 중범죄 건수
- `misdemeanor_count_*`
  - 경범죄 건수
- `night_crime_count_*`
  - 야간 범죄 건수

야간 범죄는 신고 시각 기준 `22시 이상 또는 6시 미만`으로 정의했다.

### 4.3 범죄 비율 변수

- `felony_ratio_0.5km`, `felony_ratio_1km`, `felony_ratio_2km`
- `misdemeanor_ratio_0.5km`, `misdemeanor_ratio_1km`, `misdemeanor_ratio_2km`
- `night_crime_ratio_0.5km`, `night_crime_ratio_1km`, `night_crime_ratio_2km`

각 반경별 전체 범죄 수 대비 유형별 범죄가 차지하는 비율을 계산했다.

예:

- `felony_ratio_1km = felony_count_1km / crime_count_1km`

이 비율 변수는 단순한 범죄 총량뿐 아니라, 주변 범죄의 구성 특성을 함께 반영하기 위해 생성했다.

## 5. 최종 산출물

feature engineering 단계가 끝나면, Airbnb 원본 변수에 관광지, 지하철, 범죄 관련 파생변수가 결합된 최종 `airbnb_df`가 완성된다.

이 데이터셋은 이후 단계에서

- 결측치 처리
- 이상치 제거
- 로그 변환
- 피처 선택
- 모델링

에 사용되는 기반 데이터 역할을 한다.

## 6. 정리

이 노트북의 핵심은 숙소 가격에 영향을 줄 수 있는 지역적 맥락을 정량화했다는 점이다.

- 관광지 접근성은 관광지 밀도와 도심 거리로 표현
- 지하철 접근성은 최근접 역 거리와 주변 역 개수로 표현
- 범죄 환경은 총량, 유형별 건수, 비율로 세분화해 표현

즉, `airbnb_feature_engineering.ipynb`는 이후 전처리와 모델링 단계의 성능을 높이기 위한 공간적 파생변수 생성 단계로 정리할 수 있다.
