# Airbnb Preprocessing Notes

`airbnb_preprocessing.ipynb`에서는 Airbnb 원본 및 파생 데이터를 모델링 가능한 형태로 정리하기 위한 전처리 과정을 수행했다.  
단순 결측치 처리에 그치지 않고, 반경별로 생성한 파생변수들 중 어떤 변수를 대표값으로 사용할지 상관계수와 VIF를 기준으로 검토했다.  
이 문서는 전처리 흐름과 변수 선택 근거를 포트폴리오 관점에서 요약한 기록이다.

## 1. 전처리 요약

### 결측치 처리

- `name`, `host_name` 제거
- `last_review` 제거
- `reviews_per_month` 결측치는 `0`으로 대체
- 범죄 비율 변수(`felony_ratio_*`, `misdemeanor_ratio_*`, `night_crime_ratio_*`)의 결측치는 `0`으로 처리

### 이상치 처리

- `price > 0` 조건을 만족하는 데이터만 유지
- `price` 상위 1% 이상치는 제거
- `minimum_nights` 상위 1% 이상치는 제거

### 로그 변환

왜도가 큰 수치형 변수에 대해 `log1p` 변환을 적용했다.

- `minimum_nights` → `log_minimum_nights`
- `distance_to_nearest_station` → `log_distance_to_nearest_station`
- `calculated_host_listings_count` → `log_calculated_host_listings_count`
- `number_of_reviews` → `log_number_of_reviews`
- `reviews_per_month` → `log_reviews_per_month`
- `station_count_300m` → `log_station_count_300m`
- `station_count_500m` → `log_station_count_500m`
- `price` → `log_price`

## 2. 반경별 대표 변수 선택

반경 기반 파생변수는 같은 개념을 서로 다른 거리 스케일로 계산했기 때문에 그룹 내부 상관이 높을 가능성이 컸다.  
따라서 각 그룹 안에서 `log_price`와의 상관계수가 가장 높은 변수를 대표 변수로 선택했다.

| 그룹 | 선택 변수 | `log_price`와의 상관계수 |
| --- | --- | ---: |
| attraction | `attraction_count_3km` | 0.4671 |
| station_count | `station_count_1km` | 0.3159 |
| crime_count | `crime_count_1km` | 0.2590 |
| felony_count | `felony_count_2km` | 0.2818 |
| misdemeanor_count | `misdemeanor_count_2km` | 0.2678 |
| night_crime_count | `night_crime_count_1km` | 0.2157 |
| felony_ratio | `felony_ratio_2km` | 0.1530 |
| misdemeanor_ratio | `misdemeanor_ratio_2km` | 0.2291 |
| night_crime_ratio | `night_crime_ratio_2km` | -0.3262 |

### attraction 그룹

- 그룹 내 상관관계: 약 `0.93`
- `attraction_count_3km`와 `log_price`: `0.47`
- `attraction_count_2km`와 `log_price`: `0.42`

결론: 두 변수는 매우 유사한 정보를 담고 있으므로 `log_price`와의 상관이 더 높은 `attraction_count_3km`를 대표 변수로 선택했다.

### station_count 그룹

- `station_count_1km`와 `log_price`: `0.3159`
- `station_count_500m`와 `log_price`: `0.2352`
- `station_count_300m`와 `log_price`: `0.1599`

결론: count 계열 중에서는 `station_count_1km`가 가장 설명력이 높아 대표 변수로 선택했다.

### crime 관련 그룹

- 총 범죄 수, 범죄 유형별 수, 범죄 비율 변수는 각각 별도 그룹으로 보고 대표 반경을 선택했다.
- 특히 `night_crime_ratio_2km`는 범죄 비율 계열 중 `log_price`와의 관계가 가장 강했다.

결론: 반경별 대표 변수는 유지하되, 이후 VIF 단계에서 최종 모델 후보를 다시 압축해 검토했다.

## 3. 최종 VIF 점검

초기에는 절편 없이 VIF를 계산해 일부 값이 과대평가될 가능성이 있었다.  
이후 `sm.add_constant()`를 사용해 `const`를 포함한 방식으로 다시 계산했고, 이 결과를 최종 판단 기준으로 사용했다.

개별 검토 단계에서 확인한 후보 변수는 다음과 같다.

- `log_minimum_nights`
- `log_number_of_reviews`
- `log_reviews_per_month`
- `log_calculated_host_listings_count`
- `availability_365`
- `distance_to_city_center`
- `is_manhattan`
- `log_distance_to_nearest_station`
- `attraction_count_3km`
- `station_count_1km`
- `night_crime_ratio_2km`

개별 검토 기준 VIF 결과:

| 변수 | VIF |
| --- | ---: |
| `attraction_count_3km` | 3.726072 |
| `distance_to_city_center` | 2.609597 |
| `log_reviews_per_month` | 2.576489 |
| `is_manhattan` | 2.481653 |
| `log_number_of_reviews` | 2.437605 |
| `station_count_1km` | 1.939807 |
| `log_distance_to_nearest_station` | 1.584013 |
| `night_crime_ratio_2km` | 1.495553 |
| `log_calculated_host_listings_count` | 1.426055 |
| `availability_365` | 1.326464 |
| `log_minimum_nights` | 1.318374 |

해석:

- 모든 변수의 VIF가 `5` 미만으로 나타났다.
- 최고값은 `attraction_count_3km = 3.726072`였다.
- 따라서 개별 검토 단계에서 구성한 후보 변수 세트는 심각한 다중공선성 문제가 없는 것으로 판단했다.

## 4. Station 테마 별도 점검

지하철 접근성은 하나의 주제를 여러 방식으로 표현한 변수군이기 때문에 별도로 내부 공선성을 점검했다.

검토 변수:

- `distance_to_nearest_station`
- `station_count_300m`
- `station_count_500m`
- `station_count_1km`

### station 테마 VIF 및 설명력

| 변수 | `R²` | implied VIF |
| --- | ---: | ---: |
| `station_count_500m` | 0.729194 | 3.692686 |
| `station_count_1km` | 0.629416 | 2.698444 |
| `station_count_300m` | 0.535784 | 2.154169 |
| `distance_to_nearest_station` | 0.093582 | 1.103244 |

해석:

- `station_count` 계열끼리는 일부 중복되는 정보를 담고 있다.
- 반면 `distance_to_nearest_station`은 나머지 count 변수들로 잘 설명되지 않아 상대적으로 독립적이다.
- 따라서 지하철 접근성은 `distance_to_nearest_station`과 대표 count 변수 1개를 함께 사용하는 것이 적절하다고 판단했다.
- 대표 count 변수는 `log_price`와 상관이 가장 높은 `station_count_1km`를 선택했다.

## 5. 팀별 검토 결과 반영

최종 파생변수는 각 팀원이 검토한 결과를 종합해 확정했다.

### 관광지

- VIF 계수: 약 `7`
- 그룹 내 상관관계: `0.93`
- `attraction_count_3km`와 `log_price`: `0.47`
- `attraction_count_2km`와 `log_price`: `0.42`

결론: `attraction_count_3km` 사용

### 위치

- `distance_to_city_center` VIF = `1.6250`
- `is_manhattan` VIF = `1.6250`

초기 결론: 두 변수 모두 유지 후보로 검토  
최종 결론: 이후 `neighbourhood_group` 더미화와 전체 VIF 재점검 과정에서 Manhattan 더미와 정보가 중복된다는 점이 확인되어, 최종 모델 입력에서는 `is_manhattan`을 제외했다.

### 범죄 파생변수

범죄 관련 변수는 팀원별 추가 검토를 통해 최종 조합을 다시 점검했다.

| 변수 | VIF |
| --- | ---: |
| `night_crime_ratio_2km` | 1.389689 |
| `misdemeanor_ratio_2km` | 1.232742 |
| `felony_count_2km` | 1.232110 |

결론: 세 변수 모두 다중공선성이 낮아 함께 활용 가능

## 6. 최종 사용 파생변수

팀 검토 결과를 반영한 1차 파생변수 후보는 다음과 같이 정리했다.

- `night_crime_ratio_2km`
- `misdemeanor_ratio_2km`
- `felony_count_2km`
- `attraction_count_3km`
- `distance_to_city_center`
- `is_manhattan`
- `log_distance_to_nearest_station`
- `station_count_1km`

이후 전체 피처 기준 VIF 재점검 과정에서 `is_manhattan`은 Manhattan 더미와 중복된다는 점이 확인되어 최종 모델 입력에서는 제외했다.

## 7. 정리

이번 전처리 단계에서는 단순 결측치 처리와 로그 변환을 넘어, 반경별 파생변수 중 어떤 변수를 최종 입력 변수로 유지할지 정량적으로 검토했다.  
상관계수 기반 대표 변수 선택, 절편 포함 VIF 재계산, 팀별 추가 검토를 거쳐 설명력을 유지하면서도 다중공선성이 과도하지 않은 파생변수 세트를 구성했다.

## 8. 작업 히스토리 요약

최종 피처를 확정하기 전, 먼저 `airbnb_preprocessing.ipynb`의 현재 상태를 다시 점검했다.  
이미 노트북에는 결측치 처리, 일부 불필요 컬럼 제거, 이상치 제거, 로그 변환까지 상당 부분 전처리가 진행되어 있었기 때문에, 이후 작업은 “기초 전처리”보다는 “모델 입력용 변수 선택”에 더 가까운 단계로 정리했다.

그다음에는 반경별로 생성한 파생변수들을 동일한 기준으로 비교했다.  
관광지 수, 지하철 수, 범죄 수, 범죄 비율처럼 같은 개념을 여러 거리 스케일로 만든 변수들은 구조적으로 서로 상관이 높을 가능성이 크기 때문에, 먼저 `log_price`와의 상관계수를 계산해 각 그룹에서 대표 변수를 1차로 선택했다.  
이 단계에서 중요한 목적은 “어떤 반경이 해당 개념을 가장 잘 대표하는가”를 결정하는 것이었고, 단순히 많은 변수를 남기는 것보다 해석 가능성과 설명력을 함께 확보하는 방향으로 정리했다.

이후에는 다중공선성을 점검하기 위해 VIF를 계산했다.  
처음에는 절편 없이 계산한 결과를 보았는데, 이 방식은 일부 변수의 VIF를 과대평가할 수 있다는 점을 확인했다.  
그래서 `sm.add_constant()`를 이용해 절편을 포함한 형태로 VIF를 다시 계산했고, 그 결과 실제 후보 변수들의 다중공선성은 처음 우려했던 것보다 훨씬 심하지 않다는 점을 재확인했다.  
이 과정을 통해 단순히 수치를 보는 것이 아니라, 계산 방식 자체가 해석에 미치는 영향도 함께 점검했다.

특히 `station` 관련 변수군은 별도로 다시 떼어내어 검토했다.  
`distance_to_nearest_station`, `station_count_300m`, `station_count_500m`, `station_count_1km`는 모두 “지하철 접근성”이라는 같은 주제를 다른 방식으로 표현한 변수들이기 때문에, 전체 VIF만으로 보기보다 그룹 내부 관계를 따로 보는 것이 더 적절하다고 판단했다.  
상관행렬, 절편 포함 VIF, 그리고 각 변수를 나머지 station 변수들로 회귀한 `R² / implied_vif`까지 계산한 결과, `distance_to_nearest_station`은 count 계열과 완전히 같은 정보를 담고 있지 않으며 상대적으로 독립적인 축이라는 점을 확인했다.  
그래서 station 테마에서는 `station_count_1km`와 `distance_to_nearest_station`을 함께 사용하는 방향이 타당하다고 결론지었다.

이 패턴은 이후 `attraction`과 `crime` 관련 변수군에도 그대로 확장했다.  
즉, 특정 테마에 속하는 변수들을 따로 묶고, 상관행렬과 VIF, `R²`를 같은 방식으로 비교할 수 있도록 노트북 구조를 정리했다.  
이렇게 함으로써 팀원별로 도출한 결론을 같은 기준에서 다시 확인할 수 있었고, 변수 선택 근거를 문서화하기도 쉬워졌다.

그다음 단계에서는 팀 논의 결과를 반영해 최종 파생변수 후보를 확정했다.  
이때 단순히 개인 검토 결과만 따르는 것이 아니라, 각 팀원이 본 수치와 해석을 합쳐 아래 8개를 1차 파생변수 후보로 유지하기로 했다.

- `night_crime_ratio_2km`
- `misdemeanor_ratio_2km`
- `felony_count_2km`
- `attraction_count_3km`
- `distance_to_city_center`
- `is_manhattan`
- `log_distance_to_nearest_station`
- `station_count_1km`

그 이후에는 로그 변환된 변수와 원본 변수가 동시에 남아 있으면 해석과 모델링 단계에서 중복이 생길 수 있으므로, `log_` 컬럼이 생성된 변수의 원본 컬럼은 제거하는 방향으로 정리했다.  
예를 들어 `distance_to_nearest_station`은 원본 대신 `log_distance_to_nearest_station`을 유지하는 방식으로 정리되었다.  
이 단계는 단순한 컬럼 정리가 아니라, “최종 모델에는 어떤 형태의 변수 표현을 넣을 것인가”를 결정하는 작업이었다.

마지막으로 남은 컬럼들 중 직접적인 위치 원본 정보와 범주형 변수를 다시 검토했다.  
`neighbourhood`, `latitude`, `longitude`는 이미 위치 파생변수들이 충분히 설명하고 있다고 판단하여 제외하기로 했다.  
반면 `neighbourhood_group`, `room_type`은 범주형 정보 자체에 의미가 있으므로, 회귀와 머신러닝 모델에 투입할 수 있도록 더미 변수로 변환하는 방향을 선택했다.

이후 전체 피처 기준 VIF를 다시 계산하면서 `is_manhattan`과 `neighbourhood_group_Manhattan`이 사실상 같은 정보를 담고 있다는 점을 확인했다.  
그 결과 `is_manhattan`은 최종 모델 입력에서는 제외하고, 위치 정보는 `distance_to_city_center`, `attraction_count_3km`, `station_count_1km`, `log_distance_to_nearest_station` 같은 파생변수 중심으로 설명하는 방향으로 정리했다.

정리하면, 이 작업 히스토리는 단순히 컬럼을 지우고 남긴 과정이 아니라 다음의 흐름으로 진행되었다.

1. 기존 전처리 상태 재확인
2. 반경별 파생변수 대표값 1차 선택
3. 절편 포함 VIF 재계산으로 다중공선성 재검토
4. station, attraction, crime 테마별 개별 점검
5. 팀 검토 결과를 반영한 최종 파생변수 확정
6. 로그 변환본 우선 유지 및 원본 제거
7. 위치 원본 제거 및 범주형 더미화

즉 현재의 최종 피처 세트는 한 번에 정해진 것이 아니라, 상관계수, VIF, 그룹별 해석, 팀 논의를 반복적으로 거쳐 단계적으로 정제된 결과다.

## 9. 최종 피처 선정 과정과 결과

최종 모델 입력 변수는 파생변수만 따로 고른 뒤 비파생변수와 결합하는 방식이 아니라, 현재 `airbnb_df` 상태 전체를 기준으로 다시 한 번 정리했다.  
즉, 전처리와 로그 변환이 반영된 실제 데이터프레임에서 어떤 컬럼을 남기고 어떤 컬럼을 제거할지 순차적으로 확정했다.

### 1. 로그 변환 컬럼 우선 사용

왜도가 큰 수치형 변수는 원본 대신 로그 변환 컬럼을 사용하는 방향으로 정리했다.

- `minimum_nights` → `log_minimum_nights`
- `number_of_reviews` → `log_number_of_reviews`
- `reviews_per_month` → `log_reviews_per_month`
- `calculated_host_listings_count` → `log_calculated_host_listings_count`
- `distance_to_nearest_station` → `log_distance_to_nearest_station`

이후 `log_` 컬럼이 생성된 변수의 원본은 제거하고, 로그 변환본만 남겼다.  
단, 타깃 비교를 위해 `price`는 남기고 `log_price`도 함께 유지했다.

### 2. 파생변수 정리

반경 기반 파생변수 중에서는 팀 논의를 통해 확정한 8개만 남기고 나머지는 제거했다.

- `night_crime_ratio_2km`
- `misdemeanor_ratio_2km`
- `felony_count_2km`
- `attraction_count_3km`
- `distance_to_city_center`
- `is_manhattan`
- `log_distance_to_nearest_station`
- `station_count_1km`

이 단계에서 `distance_to_nearest_station` 원본은 로그 변환본으로 대체되었고, `station_count_300m`, `station_count_500m` 및 각종 범죄 count / ratio 파생변수의 다른 반경 버전은 제거했다.  
단, `is_manhattan`은 이 시점의 1차 후보에는 포함되었지만, 이후 전체 피처 기준 재점검 과정에서 최종 입력에서는 제외되었다.

### 3. 위치 관련 원본 변수 제거

원래 위치 정보를 직접 담고 있던 변수 중 아래 세 컬럼은 제거했다.

- `neighbourhood`
- `latitude`
- `longitude`

이유는 이미 위치를 설명하는 파생변수(`distance_to_city_center`, `attraction_count_3km`, `station_count_1km`, `log_distance_to_nearest_station`)를 충분히 구성했기 때문이다.

### 4. 범주형 변수 더미화

남아 있던 범주형 변수 중 `neighbourhood_group`, `room_type`은 더미 변수로 변환했다.

- `neighbourhood_group`는 5개 카테고리
- `room_type`은 3개 카테고리
- `drop_first=True` 기준으로 더미 변수를 생성했다.

이 과정에서 각 컬럼은 문자열 범주형에서 `0/1` 수치형 컬럼으로 바뀌었다.

### 5. 최종 VIF 재점검 과정

전체 수치형 컬럼을 대상으로 다시 상관관계 매트릭스와 VIF를 점검했다.  
이 과정에서 아래 이슈를 확인했다.

- `is_manhattan`과 `neighbourhood_group_Manhattan`은 사실상 같은 정보를 담고 있어 완전 중복에 가까웠다.
- 따라서 `is_manhattan`은 제거하는 방향으로 정리했다.
- 이후 `neighbourhood_group_Brooklyn`, `neighbourhood_group_Manhattan`, `neighbourhood_group_Queens` 역시 위치 파생변수들과 정보가 많이 겹쳐 높은 VIF를 보였다.
- 이 프로젝트는 위치를 파생변수로 설명하는 방향이 더 일관적이므로, `neighbourhood_group_*` 더미 변수도 최종 모델 입력에서는 제외했다.

또한 VIF 계산에서는 타깃과 식별자 컬럼을 제외했다.

- 제외: `price`, `log_price`, `id`, `host_id`

이렇게 정리한 뒤 다시 계산한 최종 VIF 결과는 다음과 같다.

| 변수 | VIF |
| --- | ---: |
| `attraction_count_3km` | 3.810409 |
| `distance_to_city_center` | 2.580278 |
| `log_reviews_per_month` | 2.579518 |
| `log_number_of_reviews` | 2.444352 |
| `felony_count_2km` | 2.203637 |
| `station_count_1km` | 1.972559 |
| `log_distance_to_nearest_station` | 1.651309 |
| `night_crime_ratio_2km` | 1.515354 |
| `log_calculated_host_listings_count` | 1.449621 |
| `misdemeanor_ratio_2km` | 1.374651 |
| `log_minimum_nights` | 1.371358 |
| `availability_365` | 1.330654 |
| `room_type_Private room` | 1.141966 |
| `room_type_Shared room` | 1.047083 |

해석:

- 모든 최종 피처의 VIF가 `5` 미만이다.
- 따라서 최종 모델 입력 변수 세트는 다중공선성 측면에서 안정적인 수준으로 판단했다.

### 6. 최종 모델 입력 피처

최종적으로 모델 입력 피처로 사용하기로 한 변수는 다음과 같다.

- `attraction_count_3km`
- `distance_to_city_center`
- `log_reviews_per_month`
- `log_number_of_reviews`
- `felony_count_2km`
- `station_count_1km`
- `log_distance_to_nearest_station`
- `night_crime_ratio_2km`
- `log_calculated_host_listings_count`
- `misdemeanor_ratio_2km`
- `log_minimum_nights`
- `availability_365`
- `room_type_Private room`
- `room_type_Shared room`

제외한 주요 변수는 다음과 같다.

- 식별용 컬럼: `id`, `host_id`
- 타깃 관련 컬럼: `price`, `log_price`
- 직접 위치 원본: `neighbourhood`, `latitude`, `longitude`
- 중복 위치 변수: `is_manhattan`, `neighbourhood_group_*`

결론적으로 최종 피처 선정은 다음 순서로 이루어졌다.

1. 전처리 및 로그 변환
2. 반경별 대표 파생변수 선택
3. 팀 기준 최종 파생변수 확정
4. 원본 위치 변수 제거
5. 범주형 더미화
6. 전체 피처 기준 VIF 재점검
7. 최종 입력 변수 확정
