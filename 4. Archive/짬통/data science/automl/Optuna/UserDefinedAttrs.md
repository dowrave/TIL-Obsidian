```python
study = optuna.create_study(storage = 'sqlite:///example.db')

# 사용자 속성 정의
study.set_user_attr('contributors', ['Akiba', 'Sano'])
study.set_user_attr('dataset', 'MNIST')

# 사용자 속성 접근
study.user_attrs
```

- 이런 방법도 있음
```python
summaries = optuna.get_all_study_summaries('sqlite:///example.db')
summaries[0].user_attrs
```

### Trial에 사용자 속성 추가하기
```python
def objective(trial):
    iris = sklearn.datasets.load_iris()
    x, y = iris.data, iris.target
    
    svc_c = trial.suggest_float('svc_c', 1e-10, 1e10, log = True)
    clf = sklearn.svm.SVC(C = svc_c)
    
    # 사용자 속성 추가
    accuracy = sklearn.model_selection.cross_val_score(clf, x, y).mean()
    
    # 사용자 속성 정의
    trial.set_user_attr('accuracy', accuracy)
    
    return 1.0 - accuracy

study.optimize(objective, n_trials = 1)

# 사용자 정의 속성과 그 스코어가 기록됨
study.trials[0].user_attrs

```
- `optmize`에서 기록된 value는 `0.059999..4` (1-accuracy)
- `accuracy`에 기록된 value는 `0.94000..001`
