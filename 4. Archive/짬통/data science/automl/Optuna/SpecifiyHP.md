### 1. `enqueue_trial()` : **우선적으로 탐색하고 싶은 하이퍼파라미터 세트가 있을 때 사용**

```python
# 참고 ) 모델은 lightgbm
def objective(trial):

	# ...

    param = {
        'objective' : 'binary',
        'metric' : 'auc',
        'verbosity' : -1,
        'boosting_type' : 'gbdt',
        'bagging_fraction' : min(trial.suggest_float('bagging_fraction', 0.4, 1.0 + 1e-12), 1),
        'bagging_freq' : trial.suggest_int('bagging_freq', 0, 7),
        'min_child_samples' : trial.suggest_int('min_child_samples', 5, 100),
    }

	return accuracy
    


study = optuna.create_study(direction = 'maximize',
                           pruner = optuna.pruners.MedianPruner())

study.enqueue_trial(
    {
        'bagging_fraction' : 1.0,
        'bagging_freq' : 0,
        'min_child_samples' : 20
    })

study.enqueue_trial({
    'bagging_fraction' : 0.75,
    'bagging_freq' : 5,
    'min_child_samples' : 20
})

optuna.logging.get_logger('optuna').addHandler(logging.StreamHandler(sys.stdout))
study.optimize(objective, n_trials = 100, timeout = 600)
```
- `enqueue_trial`로 지정된 하이퍼파라미터 세트부터 우선적으로 탐색한다.

### 2. `add.trial()` : 하이퍼파라미터 세트 쌍 & 그 결과를 알고 있을 때 사용
- `optimize`는 위와 같다고 가정
```python

# objective는 optimize에서 전달한다
study = optuna.create_study(direction = 'maximize', pruner = optuna.pruners.MedianPruner())

# 결과를 미리 아는 경우 1
study.add_trial(
    optuna.trial.create_trial(
        params = {
            'bagging_fraction' : 1.0,
            'bagging_freq' : 0,
            'min_child_samples' : 20,
        },
        distributions = {
            'bagging_fraction' : optuna.distributions.FloatDistribution(0.4, 1.0 + 1e-12),
            'bagging_freq' : optuna.distributions.IntDistribution(0, 7),
            'min_child_samples' : optuna.distributions.IntDistribution(5, 100),
        },
        value = 0.94
    )
)

# 결과를 미리 아는 경우 2
study.add_trial(
    optuna.trial.create_trial(
        params={
            "bagging_fraction": 0.75,
            "bagging_freq": 5,
            "min_child_samples": 20,
        },
        distributions={
            "bagging_fraction": optuna.distributions.FloatDistribution(0.4, 1.0 + 1e-12),
            "bagging_freq": optuna.distributions.IntDistribution(0, 7),
            "min_child_samples": optuna.distributions.IntDistribution(5, 100),
        },
        value=0.95,
    )
)

study.optimize(objective, n_trials=100, timeout=600)
```
-  `add_trial()` 할 때마다 `distribution`을 번거롭게 일일이 전달해야 하는가?
	- **줘야 함** : `distribution`이 없으면 `params`의 `key`값들이 어디서 왔는지를 모르기 때문임
