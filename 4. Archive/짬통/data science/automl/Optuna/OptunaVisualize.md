- lightgbm을 통한 예제
```python
import lightgbm as lgb
import numpy as np
import sklearn.datasets
import sklearn.metrics
from sklearn.model_selection import train_test_split

import optuna
from optuna.trial import Trial as trial
from optuna.visualization import (plot_contour, plot_edf, plot_intermediate_values, plot_optimization_history, 
                                  plot_parallel_coordinate, plot_param_importances, plot_slice)

SEED = 42
np.random.seed(SEED)
```
```python
def objective(trial):
    data, target = sklearn.datasets.load_breast_cancer(return_X_y = True)
    train_x, valid_x, train_y, valid_y = train_test_split(data, target, test_size = 0.25)
    dtrain = lgb.Dataset(train_x, label = train_y)
    dvalid = lgb.Dataset(valid_x, label = valid_y)
    
    param = {
        'objective' : 'binary',
        'metric' : 'auc',
        'verbosity' : -1,
        'boosting_type' : 'gbdt',
        'bagging_fraction' : trial.suggest_float('bagging_fraction', 0.4, 1.0),
        'bagging_freq' : trial.suggest_int('bagging_freq', 1, 7),
        'min_child_samples' : trial.suggest_int('min_child_samples', 5, 100)
    }
    
    # Adding Pruning Callback
    pruning_callback = optuna.integration.LightGBMPruningCallback(trial, 'auc')
    gbm = lgb.train(
        param, dtrain, valid_sets = [dvalid], verbose_eval = False, callbacks = [pruning_callback]
        )

    preds = gbm.predict(valid_x)
    pred_labels = np.rint(preds)
    accuracy = sklearn.metrics.accuracy_score(valid_y, pred_labels)

    return accuracy
```
```python
study = optuna.create_study(
    direction = 'maximize',
    sampler = optuna.samplers.TPESampler(seed = SEED),
    pruner = optuna.pruners.MedianPruner(n_warmup_steps = 10),
)
study.optimize(objective, n_trials = 100, timeout = 600)
```

### 시각화하기
- 아마 `plotly` 기반인 것 같음
```python
# 전체 최적화 과정
plot_optimization_history(study)

# 중간값들 시각화
plot_intermediate_values(study)

# 하이퍼파라미터 쌍과 성능 시각화
plot_parallel_coordiante(
						 study,
						# params = ['bagging_freq', 'bagging_fraction']
						)

# 하이퍼파라미터 간의 관계 시각화
plot_contour(study,
			# params = ['bagging_freq', 'bagging_fraction']
			)

# Slice Plot
plot_slice(
		   study,
		 # params = ['bagging_freq', 'bagging_fraction']
		  )

# 하이퍼파라미터의 중요도 관찰
plot_param_importances(study)

# 다른 지표에 하이퍼파라미터가 관여하는 바 관찰하기
plot_param_importances(study,
					  target = lambda t : t.duration.total_seconds(),
					  target_name = 'duration')

# 경험적 누적분포 함수(Empirical Distribution Function)
plot_edf(study)
```
- `edf` : 반복된 시행을 통해 확률 변수가 일정 값을 넘지 않을 확률을 유추함