- 예제는 `SQLite`를 이용했으며 `DB`의 URL을 세팅하면 `PostgreSQL`이나 `MySQL`을 이용할 수도 있다

1. 새로운 Study 만들기
```python
import logging
import sys
import optuna

# 빨간 바탕에 뜨는 메시지 끄기
# optuna.logging.set_verbosity(optuna.logging.WARNING)
optuna.logging.set_verbosity(optuna.logging.INFO) # 기본 설정

# 메시지를 보여주기 위한 StreamHandler를 추가
optuna.logging.get_logger('optuna').addHandler(logging.StreamHandler(sys.stdout))

study_name= 'example-study'
storage_name = 'sqlite:///{}.db'.format(study_name)

study = optuna.create_study(study_name = study_name,
                            storage = storage_name)

def objective(trial):
    x = trial.suggest_float('x', -10, 10)
    return (x-2) ** 2

study.optimize(objective, n_trials = 3)
```

2. Study 재개하기
	- `create_study`로 하되, 파라미터들에 주목하자
```python
study = optuna.create_study(study_name = study_name,
                            storage = storage_name,
                            load_if_exists = True)

study.optimize(objective, n_trials = 3)
```

3. Study 기록으로 보기
```python
study = optuna.create_study(study_name = study_name,
                            storage = storage_name,
                            load_if_exists = True)

df = study.trials_dataframe(attrs = ("number", 'value', 'params', 'state'))

print(df)

print("Best params: ", study.best_params)
print("Best value: ", study.best_value)
print("Best Trial: ", study.best_trial)
print("Trials: ", study.trials)
```