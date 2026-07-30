- Optuna를 쓰지 않았을 때, 썼을 때, AskTell 인터페이스를 썼을 때를 비교해본다

### 1. Optuna 없음
```python
import numpy as np
from sklearn.datasets import make_classification
from sklearn.linear_model import LogisticRegression
from sklearn.model_selection import train_test_split

import optuna


# optuna 없을 때
X, y = make_classification(n_features = 10)
X_train, X_test, y_train, y_test = train_test_split(X, y)

C = 0.01
clf = LogisticRegression(C = C)
clf.fit(X_train, y_train)
val_accuracy = clf.score(X_test, y_test)
```

### 2. Optuna 적용
```python
def objective(trial):
    X, y = make_classification(n_features = 10)
    X_train, X_test, y_train, y_test = train_test_split(X, y)
    
    C = trial.suggest_float("C", 1e-7, 10.0, log = True)
    solver = trial.suggest_categorical('solver', ('lbfgs', 'saga'))
    
    clf = LogisticRegression(C = C, solver = solver)
    clf.fit(X_train, y_train)
    val_accuracy = clf.score(X_test, y_test)
    
    return val_accuracy

study = optuna.create_study(direction = 'maximize')
study.optimize(objective, n_trials = 10)
```
- 이거를 돌리면 이런 문구가 나옴
```
ConvergenceWarning: The max_iter was reached which means the coef_ did not converge
```
- 추가적인 인수가 필요하다면 `class`로 정의할 필요가 있음

### 3. Optuna : Ask & Tell
```python
study = optuna.create_study(direction = 'maximize')

n_trials = 10

for _ in range(n_trials):
    trial = study.ask() # trial은 Trial임. FrozenTrial이 아님!
    
    C = trial.suggest_float("C", 1e-7, 10.0, log = True)
    solver = trial.suggest_categorical('solver', ('lbfgs', 'saga'))
    
    clf = LogisticRegression(C=C, solver = solver)
    clf.fit(X_train, y_train)
    val_accuracy = clf.score(X_test, y_test)
    
    # tell 메서드를 이용해 trial과 objective_value를 같이 준다
    study.tell(trial, val_accuracy)
```
- 2번째와의 차이라면  `Objective`를 정의하지 않고, `study.ask()` ~ `study.tell()`로 이어지는 구간 내에 정의되었다는 거임
	- `study.ask()` : 하이퍼파라미터 샘플링하는 `trial` 생성
	- `study.tell()` : `trial`을 완료함
- 즉 `Objective Function`을 정의하지 않고 하이퍼파라미터 최적화를 할 수 있다는 것임

### Ask & Tell 다른 예제
```python
from sklearn.datasets import load_iris
from sklearn.linear_model import SGDClassifier

X, y = load_iris(return_X_y = True)
X_train, X_valid, y_train, y_valid = train_test_split(X, y)
classes = np.unique(y)
n_train_iter = 100

study = optuna.create_study(
    direction = 'maximize',
    pruner = optuna.pruners.HyperbandPruner(
        min_resource = 1, max_resource = n_train_iter, reduction_factor = 3
    )
)

for _ in range(20):
    trial = study.ask()
    
    alpha = trial.suggest_float('alpha', 0.0, 1.0)
    
    clf = SGDClassifier(alpha = alpha)
    pruned_trial = False
    
    for step in range(n_train_iter):
        clf.partial_fit(X_train, y_train, classes = classes)
        
        intermediate_value = clf.score(X_valid, y_valid)
        trial.report(intermediate_value, step)
        
        if trial.should_prune():
            pruned_trial = True
            break
            
    if pruned_trial:
        study.tell(trial, state = optuna.trial.TrialState.PRUNED)
    else:
        score = clf.score(X_valid, y_valid)
        study.tell(trial, score)
```

## Ask & Tell : 하이퍼파라미터 분포 사전 정의
- `distribution`을 정의하고 `study.ask()`에 전달하면 됨
```python
study = optuna.create_study(direction = 'maximize')
n_trials = 10

for _ in range(n_trials):
    trial = study.ask(distributions)
    
    # 사전에 분포를 정의했기 떄문에 `ask()`를 통해 안에 파라미터로 들어가 있음
    C = trial.params['C']
    solver = trial.params['solver']
    
    clf = LogisticRegression(C = C, solver = solver)
    clf.fit(X_train, y_train)
    val_accuracy = clf.score(X_test, y_test)
    
    study.tell(trial, val_accuracy)
```

## Batch Optimization
- Batch 처리도 가능함 (`ask`는 각 `trial`에 대해, `tell`은 `batch`에 대해 이루어지고 있음!)
```python
def batched_objective(xs: np.ndarray, ys: np.ndarray):
    return xs ** 2 + ys

# 총 30개의 Trial : 
batch_size = 10
study = optuna.create_study(sampler = optuna.samplers.CmaEsSampler())

for _ in range(3): # 각각이 1개의 batch
    trial_numbers = []
    x_batch = []
    y_batch = []
    
    for _ in range(batch_size):

        trial = study.ask()
        trial_numbers.append(trial.number)
        x_batch.append(trial.suggest_float('x', -10, 10))
        y_batch.append(trial.suggest_float('y', -10, 10))
        
    x_batch = np.array(x_batch)
    y_batch = np.array(y_batch)
    objectives = batched_objective(x_batch, y_batch)
    
    for trial_number, objective in zip(trial_numbers, objectives):
        study.tell(trial_number, objective)
```