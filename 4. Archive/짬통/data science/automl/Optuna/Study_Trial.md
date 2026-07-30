```python
import optuna
from optuna.trial import Trial as trial

def objective(trial):
    x = trial.suggest_float("x", -10, 10)
    return (x - 2) ** 2

study = optuna.create_study()
study.optimize(objective, n_trials=100)
```
- `Study` : `Trial`들의 집합
- `Trial` : 목적 함수의 단일 호출
- `Parameter` : 최적화 대상(예제에선 `x`)

- 결과 조회하기
```python
# 가장 좋은 인풋값
best_params = study.best_params
found_x = best_params["x"]

# 가장 좋은 아웃풋값
study.best_value

# 가장 좋은 값에 대한 Trial
study.best_trial

# 전체 시도
study.trials
```

- 최적화를 부분부분으로 나눠 진행할 수 있다.
- 위 상태에서 `study.optimize(objective, n_trials = 100)`을 또 하면 됨 -> `study` 객체의 `trials`는 `200`이 된다.