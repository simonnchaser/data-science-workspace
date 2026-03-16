# Hypothesis Testing Notes

## Overview

`airbnb_hypothesis_testing.ipynb`에서는 `airbnb_preprocessed_df.csv`를 불러와 주요 범주형 변수와 파생변수 수준에 따라 `log_price` 평균 차이가 통계적으로 유의한지 검정하였다.

분석 대상은 아래와 같다.

- 자치구별 평균 가격 차이
- Manhattan vs Non-Manhattan 평균 가격 차이
- 방 유형별 평균 가격 차이
- 지하철 접근성에 따른 가격 차이
- 범죄 수준에 따른 가격 차이
- 관광지 수준에 따른 가격 차이

타깃 변수는 모두 `log_price`를 사용하였다.

## Method

이번 노트북에서는 변수 유형에 따라 두 가지 검정 방식을 사용하였다.

- 세 개 이상 집단의 평균 차이 검정: `ANOVA`
- 두 집단의 평균 차이 검정: `독립표본 t-test`

### ANOVA의 의미

ANOVA는 세 개 이상 집단의 평균이 서로 같은지 검정하는 방법이다. 여기서는 자치구, 방 유형, 지하철 접근성 구간, 범죄 수준, 관광지 수준처럼 여러 그룹으로 나뉘는 변수에 적용하였다.

코드에서는 각 그룹별 `log_price` 값을 리스트로 모은 뒤 `f_oneway(*groups)`를 사용하였다.

```python
groups = [
    group['log_price'].values
    for _, group in airbnb_df.groupby('neighbourhood_group')
]

f_stat, p_value = f_oneway(*groups)
```

이 코드는 그룹별 `log_price` 배열을 준비하고, 그룹 간 평균 차이가 우연으로 보기 어려운 수준인지 검정한다는 의미를 가진다.

- `F-stat`이 클수록 그룹 간 차이가 상대적으로 크다는 뜻이다
- `p-value`가 0.05보다 작으면 귀무가설을 기각한다

### 독립표본 t-test의 의미

t-test는 두 집단의 평균이 같은지 검정하는 방법이다. 여기서는 Manhattan과 Non-Manhattan 두 집단 비교에 사용하였다.

```python
manhattan = airbnb_df.loc[airbnb_df['neighbourhood_group'] == 'Manhattan', 'log_price']
non_manhattan = airbnb_df.loc[airbnb_df['neighbourhood_group'] != 'Manhattan', 'log_price']

t_stat, p_value = ttest_ind(manhattan, non_manhattan, equal_var=False)
```

이 코드는 두 집단의 `log_price` 평균 차이가 통계적으로 유의한지 확인하는 역할을 한다. `equal_var=False`는 두 집단의 분산이 다를 수 있다고 보고 보다 완화된 형태의 Welch t-test를 적용한 것이다.

### 구간화 코드의 의미

지하철 접근성, 범죄 수준, 관광지 수준은 원래 연속형 변수이므로 바로 그룹 비교를 하기 어렵다. 따라서 박스플롯과 ANOVA를 위해 해석 가능한 수준의 그룹으로 변환하였다.

- 지하철 접근성: 거리 기준 구간화
- 범죄 수준: 분위수 기준 3그룹 구분
- 관광지 수준: 동일값이 많아 `rank`를 적용한 뒤 분위수 기준 3그룹 구분

예를 들어 관광지 수준 구간화 코드는 다음 의미를 가진다.

```python
attraction_df['attraction_level'] = pd.qcut(
    attraction_df['attraction_count_2km'].rank(method='first'),
    q=3,
    labels=['low', 'medium', 'high']
)
```

이 코드는 동일값이 많아 `qcut`이 바로 작동하지 않는 문제를 피하기 위해 먼저 순위를 부여한 뒤, 전체 데이터를 low, medium, high 세 그룹으로 나누는 역할을 한다.

## Data

- 입력 데이터: `../data/processed/airbnb_preprocessed_df.csv`
- 데이터 크기: `(47928, 47)`

주요 사용 변수는 아래와 같다.

- `neighbourhood_group`
- `room_type`
- `log_price`
- `distance_to_nearest_station`
- `crime_count_0.5km`
- `attraction_count_2km`

## 1. Borough Difference

자치구별 평균 `log_price`를 집계한 결과 `Manhattan`이 가장 높고 `Bronx`가 가장 낮게 나타났다.

평균 `log_price`

| neighbourhood_group | mean log_price |
| --- | ---: |
| Manhattan | 4.973141 |
| Brooklyn | 4.565541 |
| Queens | 4.378505 |
| Staten Island | 4.354737 |
| Bronx | 4.253539 |

가설은 다음과 같이 설정하였다.

- H0: 자치구별 평균 가격은 차이가 없다
- H1: 자치구별 평균 가격은 차이가 있다

ANOVA 결과

- F-stat: `1906.5972873617839`
- p-value: `0.0`

해석

유의수준 0.05 기준에서 귀무가설을 기각하였다. 자치구에 따라 평균 `log_price`에는 통계적으로 유의한 차이가 있다.

코드 의미

자치구별 평균 막대그래프는 그룹 간 평균 차이를 직관적으로 보여주고, 뒤이어 수행한 ANOVA는 이 차이가 단순 시각적 차이가 아니라 통계적으로도 유의한지 확인하는 역할을 한다.

## 2. Manhattan vs Non-Manhattan

`neighbourhood_group == "Manhattan"` 여부를 기준으로 두 집단을 나누어 평균 `log_price` 차이를 검정하였다.

가설은 다음과 같이 설정하였다.

- H0: Manhattan 숙소와 Non-Manhattan 숙소의 평균 `log_price`는 차이가 없다
- H1: Manhattan 숙소와 Non-Manhattan 숙소의 평균 `log_price`는 차이가 있다

독립표본 t-test 결과

- t-statistic: `82.83616170926389`
- p-value: `0.0`

해석

유의수준 0.05 기준에서 귀무가설을 기각하였다. Manhattan 숙소의 평균 가격은 Non-Manhattan 숙소보다 통계적으로 유의하게 높다.

코드 의미

이 검정은 자치구 전체를 다섯 그룹으로 보는 대신, Manhattan 여부만으로 단순화하여 핵심 위치 프리미엄이 실제로 존재하는지를 별도로 확인하는 역할을 한다.

## 3. Room Type Difference

`room_type`에 따라 `log_price` 분포 차이를 확인하고 ANOVA를 수행하였다.

가설은 다음과 같이 설정하였다.

- H0: 방 유형별 평균 가격은 차이가 없다
- H1: 방 유형별 평균 가격은 차이가 있다

ANOVA 결과

- F-stat: `17406.40434584354`
- p-value: `0.0`

해석

유의수준 0.05 기준에서 귀무가설을 기각하였다. 방 유형에 따라 평균 숙소 가격에는 통계적으로 유의한 차이가 있다.

코드 의미

방 유형 박스플롯은 가격 분포 차이를 시각적으로 보여주고, ANOVA는 `Entire home/apt`, `Private room`, `Shared room` 간 평균 차이가 통계적으로 유의한지 검정한다.

## 4. Subway Accessibility

`distance_to_nearest_station`를 기준으로 지하철 접근성 그룹을 다음과 같이 구간화하였다.

- `very_close`: 0 ~ 0.3km
- `close`: 0.3 ~ 0.5km
- `medium`: 0.5 ~ 1.0km
- `far`: 1.0 ~ 2.0km

가설은 다음과 같이 설정하였다.

- H0: 지하철 접근성에 따라 숙소 가격 차이는 없다
- H1: 지하철 접근성에 따라 숙소 가격 차이가 있다

ANOVA 결과

- F-stat: `146.78036710085013`
- p-value: `1.1267424956986088e-94`

해석

유의수준 0.05 기준에서 귀무가설을 기각하였다. 지하철 접근성에 따라 평균 `log_price`에는 통계적으로 유의한 차이가 있다. 시각화에서도 지하철과 가까운 그룹에서 상대적으로 높은 가격 분포가 나타났다.

코드 의미

연속형 거리 변수를 해석 가능한 4개 구간으로 나누어, 지하철과의 물리적 접근성이 가격 차이와 연결되는지 확인하는 과정이다.

## 5. Crime Level

`crime_count_0.5km`를 기준으로 사분위 기반 3개 수준으로 구분하여 범죄 수준별 가격 차이를 검정하였다.

- `low`
- `medium`
- `high`

가설은 다음과 같이 설정하였다.

- H0: 범죄 수준에 따라 숙소 가격 차이는 없다
- H1: 범죄 수준에 따라 숙소 가격 차이가 있다

ANOVA 결과

- F-stat: `1594.8243689315573`
- p-value: `0.0`

해석

유의수준 0.05 기준에서 귀무가설을 기각하였다. 범죄 수준에 따라 평균 `log_price`에는 통계적으로 유의한 차이가 있다. 다만 시각화에서는 범죄 수준이 높은 지역에서 가격이 높게 나타나는 경향이 확인되므로, 이는 범죄 자체보다는 Manhattan과 같은 도심 입지 효과가 함께 반영된 결과로 해석할 필요가 있다.

코드 의미

범죄 건수를 low, medium, high 세 수준으로 단순화하여 가격 분포를 비교함으로써, 범죄 수준이 가격과 어떤 방향으로 연결되는지 1차적으로 탐색하는 역할을 한다.

## 6. Attraction Level

`attraction_count_2km`는 동일값이 많아 `rank(method='first')`를 적용한 뒤 `qcut`으로 3개 수준으로 나누었다.

- `low`
- `medium`
- `high`

가설은 다음과 같이 설정하였다.

- H0: 관광지 수준에 따라 숙소 가격 차이는 없다
- H1: 관광지 수준에 따라 숙소 가격 차이가 있다

실행 노트북에서 재현된 결과를 기준으로 관광지 수준에 따른 평균 `log_price` 차이는 유의하게 나타났으며, 시각화에서도 관광지 수가 많을수록 가격이 높아지는 경향이 확인되었다.

해석

관광지 접근성은 숙소 가격 형성에 중요한 요인으로 작용한다. 특히 관광지 수준이 높은 그룹에서 더 높은 가격 분포가 관찰되어, 입지 경쟁력이 가격에 반영되고 있음을 시사한다.

코드 의미

관광지 수를 low, medium, high 세 수준으로 나누어 비교함으로써, 주변 관광 인프라가 많은 지역일수록 가격이 높아지는 패턴이 있는지를 확인하는 과정이다.

## Summary

가설검정 결과, 자치구, Manhattan 여부, 방 유형, 지하철 접근성, 범죄 수준, 관광지 수준 모두 `log_price`와 통계적으로 유의한 차이를 보였다.

핵심적으로 확인된 내용은 아래와 같다.

- Manhattan은 다른 자치구보다 평균 가격이 유의하게 높다
- 방 유형은 가격 차이를 설명하는 매우 강한 변수다
- 지하철 접근성과 관광지 접근성은 가격과 유의한 관련이 있다
- 범죄 변수는 단순 음의 영향보다 도심 입지와 얽힌 교란 효과를 함께 고려해야 한다

이 결과는 이후 피처 엔지니어링과 모델링 단계에서 위치, 접근성, 범죄, 숙소 유형 관련 변수를 핵심 설명 변수로 채택하는 근거로 활용되었다.
