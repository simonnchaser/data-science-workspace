# Airbnb Modeling Notes

`airbnb_modeling.ipynb`는 feature selection 단계에서 저장한 데이터셋을 불러와 회귀 모델을 비교하는 노트북이다. 이 문서는 선형 모델용 데이터셋과 트리 모델용 데이터셋을 각각 어떻게 사용했고, 어떤 지표로 비교했는지 공유용으로 정리한 문서다.

## 1. 모델링 데이터셋

모델링 단계에서는 하나의 데이터셋으로 모든 모델을 돌리지 않고, 모델 특성에 맞는 입력 세트를 나눠 사용했다.

### 선형 모델용 데이터셋

- 파일: `../data/processed/airbnb_linear_features_df.csv`
- 용도:
  - `Linear Regression`
  - `Ridge`
  - 비교용 `Gradient Boosting`, `XGBoost`

이 세트는 다중공선성과 해석 가능성을 고려해 정리된 피처들로 구성되었다.

### 트리 모델용 데이터셋

- 파일: `../data/processed/airbnb_tree_features_df.csv`
- 용도:
  - 트리 기반 모델의 1차 실험 데이터셋

이 세트는 원본 변수와 반경 기반 파생변수를 더 넓게 유지한 상태로 구성되었다.

최종 모델링 단계에서는 [`최종_모델링_airbnb.ipynb`](/Users/junhapark/data-science-workspace/projects/airbnb-price-prediction/notebooks/최종_모델링_airbnb.ipynb)에서 확장 피처 세트를 다시 구성해 아래 모델을 비교했다.

- `Linear Regression`
- `Ridge`
- `Gradient Boosting`
- `XGBoost`
- `LightGBM`

## 2. 타깃 변수

모든 회귀 모델의 타깃은 `price`가 아니라 `log_price`를 사용했다.

이유:

- `price`는 오른쪽 꼬리가 긴 분포를 보임
- `log_price`는 분포가 더 안정적임
- 극단적으로 높은 가격의 영향이 완화됨
- 선형 모델과 트리 모델을 같은 타깃 기준으로 비교 가능

## 3. 평가 방식

모델 성능은 단일 train/test split뿐 아니라 `5-fold cross validation` 기준으로 비교했다.

설정:

- `KFold(n_splits=5, shuffle=True, random_state=42)`

사용한 평가 지표:

- `RMSE`
- `MAE`
- `R²`

이렇게 구성한 이유는 특정 split 하나에 우연히 성능이 잘 나오거나 나쁘게 나오는 문제를 줄이고, 모델의 평균적인 일반화 성능을 보기 위함이다.

## 4. 선형 모델 비교

선형 모델용 데이터셋에서는 아래 모델들을 비교했다.

- `Linear Regression`
- `Ridge`
- `Gradient Boosting Regressor`
- `XGBoost Regressor`

이 단계의 목적은 두 가지였다.

- 선형 회귀를 baseline으로 삼기
- 같은 피처 세트에서 비선형 모델이 얼마나 개선되는지 확인하기

### 선형 모델용 피처 세트 기준 교차검증 결과

- `Linear Regression`
  - `CV_RMSE_mean = 0.437774`
  - `CV_MAE_mean = 0.332224`
  - `CV_R2_mean = 0.542752`
- `Ridge`
  - `CV_RMSE_mean = 0.437777`
  - `CV_MAE_mean = 0.332229`
  - `CV_R2_mean = 0.542746`
- `Gradient Boosting`
  - `CV_RMSE_mean = 0.427098`
  - `CV_MAE_mean = 0.323952`
  - `CV_R2_mean = 0.564779`
- `XGBoost`
  - `CV_RMSE_mean = 0.422787`
  - `CV_MAE_mean = 0.320825`
  - `CV_R2_mean = 0.573517`

해석:

- `Ridge`는 `Linear Regression`과 거의 같은 성능을 보였다.
- 선형 피처 세트 기준에서도 비선형 모델이 더 좋은 성능을 보였다.
- 같은 피처를 썼을 때 `XGBoost`가 가장 우수했다.

## 5. 선형 모델 해석

선형 모델은 예측 성능뿐 아니라 해석 가능성을 위해 사용했다.

- `Linear Regression`, `Ridge`는 `coef_`를 통해 주요 변수와 방향성을 확인
- 계수의 부호는 가격에 대한 방향성을 해석하는 데 활용
- 다만 피처 스케일 차이가 있기 때문에 절대계수 크기는 조심해서 해석

즉, 선형 모델은 최종 성능용 모델이라기보다 baseline과 해석용 기준 모델의 역할을 한다.

## 6. 최종 모델링 비교

최종 모델링 단계에서는 확장 피처 세트를 바탕으로 `GridSearchCV`를 적용해 모델별 하이퍼파라미터를 탐색했다. `cv=5` 설정으로 5-Fold Cross Validation을 수행했고, `R²`를 기준으로 최적 조합을 선택한 뒤 test set 성능을 비교했다.

### 최종 모델 비교 결과

| 모델 | Best CV Score | Test RMSE | Test MAE | Test R² |
| --- | ---: | ---: | ---: | ---: |
| LightGBM | 0.6507 | 0.3773 | 0.2844 | 0.6556 |
| XGBoost | 0.6468 | 0.3798 | 0.2869 | 0.6509 |
| Gradient Boosting | 0.6394 | 0.3851 | 0.2918 | 0.6412 |
| Ridge | 0.5766 | 0.4172 | 0.3189 | 0.5789 |
| Linear Regression | 0.5766 | 0.4172 | 0.3189 | 0.5789 |

해석:

- 비선형 트리 기반 모델이 선형 회귀형 모델보다 전반적으로 더 우수한 성능을 보였다
- `LightGBM`이 가장 높은 `Best CV Score`와 `Test R²`, 가장 낮은 오차를 기록해 최종 예측 성능 모델로 선정되었다
- `XGBoost`는 LightGBM과 유사한 성능을 보인 차선 모델이었다
- `Linear Regression`은 baseline 및 해석용 기준 모델로 유지하였다

## 7. 최종 모델 판단

현재까지 비교 결과를 기준으로 하면 역할은 다음처럼 정리할 수 있다.

- 해석 및 baseline:
  - `Linear Regression`
- 최종 예측 성능:
  - `LightGBM`
- 차선 성능 모델:
  - `XGBoost`

즉, 프로젝트 구조상 가장 설득력 있는 결론은 아래와 같다.

- 선형 회귀는 변수 선택 과정과 계수 해석에 적합한 기준 모델
- 트리 기반 모델은 실제 예측 성능 개선에 더 적합
- 최종 성능 기준으로는 `LightGBM`이 가장 좋은 회귀 모델

## 8. 다음 단계

모델링 단계에서 이어서 할 수 있는 작업은 아래와 같다.

- `LightGBM` feature importance와 SHAP 결과 정리
- 중요도가 낮은 변수 제거 후 재학습
- LightGBM과 XGBoost 추가 튜닝
- README 또는 발표 자료용 성능 비교 표 정리
