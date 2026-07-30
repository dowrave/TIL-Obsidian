## Sampling Algorithm
- `Sampler` : 검색 공간을 지속적으로 좁혀줌
	- `Relative Sampling` : 샘플링 알고리즘이 매개변수 간의 상관관계를 사용할 수 있도록 **여러 매개변수의 값을 동시에 결정**함
	    -   대상 매개변수는 `infer_relative_search_space()`**에 의해 결정되는 상대 검색 공간으로 기술**됨
	-   `Independent Sampling` : **매개변수 간의 관계를 고려하지 않고 단일 매개변수의 값을 결정**함.

-   Trial 시작 시 `infer_relative_search_space()`가 호출됨 : 그 다음 `sample_relative()`가 호출되어 상대 검색 공간에서 매개변수를 샘플링함
-   목적 함수를 실행하는 동안 `sample_independent()`는 상대 검색 공간에 속하지 않는 매개변수를 샘플링하는 데 사용됨

- 샘플링 알고리즘(디폴트 : `TPESampler`)
    -   `GridSampler`
    -   `RandomSampler`
    -   `TPESampler` : Tree-structured Parzen Estimator
    -   `CmaEsSampler` : CMA-ES
    -   `PartialFixedSampler` : Algorithm to enable partial fixed parameters
    -   `NSGAIISampler` : Nondominated Sorting Genetic Algorithm II
    -   `QMCSampler` : A Quasi Monte Carlo sampling algorithm
```python
study = optuna.create_study()
print(f"Sampler is {study.sampler.__class__.__name__}")

study = optuna.create_study(sampler = optuna.samplers.RandomSampler())
print(f"Sampler is {study.sampler.__class__.__name__}")
```


## Pruning Algorithm
- 유망하지 못한 `Trial`들을 자동으로 제거해줌
	-   `MedianPruner`
	-   `NopPruner`
	-   `PatientPruner`
	-   `PercentilePruner`
	-   `SuccessiveHalvingPruner`
	-   `HyperBandPruner`
	-   `ThresholdPruner`

- 예제
```python
import logging
import sys

import sklearn.datasets
import sklearn.linear_model
import sklearn.model_selection

def objective(trial):
    iris = sklearn.datasets.load_iris()
    classes = list(set(iris.target))
    
    train_x, valid_x, train_y, valid_y = sklearn.model_selection.train_test_split(
        iris.data, iris.target, test_size = 0.25, random_state = 0
    )
    
    alpha = trial.suggest_float('alpha', 1e-5, 1e-1, log = True)
    clf = sklearn.linear_model.SGDClassifier(alpha = alpha)
    
    for step in range(100):
        clf.partial_fit(train_x, train_y, classes = classes) # 1번의 에포크만을 수행함
        
        # 중간값 report
        intermediate_value = 1.0 - clf.score(valid_x, valid_y)
        trial.report(intermediate_value, step)
        
        # 중간값에 의해 Pruning을 함 : 그 기준은 알고리즘으로 따로 있는 듯
        if trial.should_prune():
            raise optuna.TrialPruned()
            
    return 1.0 - clf.score(valid_x, valid_y)

# study를 만들 때 pruner를 지정해주면 됨
study = optuna.create_study(pruner = optuna.pruners.MedianPruner())
study.optimize(objective, n_trials = 20)
```

## Sampler와 Pruner 조합하기

- 딥러닝이 아닌 경우
	- `RandomSampler` + `MedianPruner`
	- `TPESampler` + `HyperBandPruner`

-  딥러닝인 경우
| 병렬 컴퓨팅 자원 | 범주/조건 하이퍼파라미터 | 추천 알고리즘                         |
| ---------------- | ------------------------ | ------------------------------------- |
| 제한됨           | X                        | TPE, GP-EI(저차원 & 연속적 탐색 공간) |
|                  | O                        | TPE, GP-EI(저차원 & 연속적 탐색 공간) |
| 충분함           | X                        |           CMA-ES, RandomSearch                            | 
|                  | O                        |                               RandomSearch or Genetic Algorithm

#### Optuna에선 통합된 형태로 제공하기도 한다
ex ) `XGBoostPruningCallback`
```python
pruning_callback = optuna.integration.XGBoostPruningCallback(trial, 'validation-error')
bst = xgb.train(param, dtrain, evals=[(dvalid, 'validation')], callbacks=[pruning_callback])
```
