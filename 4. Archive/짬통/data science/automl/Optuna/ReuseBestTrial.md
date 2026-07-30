- 하이퍼파라미터 최적화한 다음 가장 좋은 하이퍼파라미터로 목적함수를 실행할 필요가 있다.
- 혹은, Train 시간을 줄이기 위해 부분 데이터세트로 훈련시킨 다음 전체 데이터세트로 훈련시키는 경우도 있음

- 여기서는 전자를 따라감
```python
from sklearn import metrics
from sklearn.datasets import make_classification
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split

import optuna

def objective(trial):
    X, y = make_classification(n_features = 10, random_state = 1)
    X_train, X_test, y_train, y_test = train_test_split(X, y, random_state = 1)
    
    C = trial.suggest_float('C', 1e-7, 10.0, log = True)
    
    clf = LogisticRegression(C = C)
    clf.fit(X_train, y_train)
    
    return clf.score(X_test, y_test)

study = optuna.create_study(direction = 'maximize')
study.optimize(objective, n_trials = 10)

print(study.best_trial.value)
```

- 다른 평가 지표를 넣기 위해, **대부분을 공유하지만 다른 목적함수를 만들어서 모델을 재현**할 수 있다
```python
def detailed_objective(trial):
    X, y = make_classification(n_features = 10, random_state = 1)
    X_train, X_test, y_train, y_test = train_test_split(X, y, random_state = 1)
    
    C = trial.suggest_float('C', 1e-7, 10.0, log = True)
    
    clf = LogisticRegression(C = C)
    clf.fit(X_train, y_train)
    
    # 다른 지표들 계산하기 : 이 부분만 다름
    pred = clf.predict(X_test)
    
    acc = metrics.accuracy_score(pred, y_test)
    recall = metrics.recall_score(pred, y_test)
    precision = metrics.precision_score(pred, y_test)
    f1 = metrics.f1_score(pred, y_test)
    
    return acc, f1, recall, precision

# 위에서 얻은 가장 좋은 trial을 그대로 집어넣어주면 됨
detailed_objective(study.best_trial)
```
- 즉 위 과정은 (`clf.score()`값은 평균 정확도를 측정하므로)  **정확도를 최대화하는 하이퍼파라미터 쌍을 `study.best_trial`에 저장**한 다음 **그 값을 `trial`을 매개변수로 하는 목적함수에 그대로 통과시켜준 거임**(출력 값만 다르게 해줌)
- 하이퍼파라미터 쌍이 아니라 그냥 `study.best_trial`을 통과시켜준 것도 주목할 점


