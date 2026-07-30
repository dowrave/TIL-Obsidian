- `BasePruner`
	-   가지치기가 필요하면 True, 아니면 False를 반환함
	-   모든 `trial` 객체에 `get_trial()`로 접근할 수 있고, 각 trial의 중윗값을 받을 수 있음

## 예제
```python
import numpy as np
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.linear_model import SGDClassifier

import optuna
from optuna.pruners import BasePruner
from optuna.trial._state import TrialState

class LastPlacePruner(BasePruner):
    
    def __init__(self, warmup_steps, warmup_trials):
        self._warmup_steps = warmup_steps
        self._warmup_trials = warmup_trials
        
    def prune(self, study = "optuna.study.Study", trial = "optuna.trial.FrozenTrial") -> bool:
        
        # 가장 최근 trial 점수 가져옴
        step = trial.last_step

        if step:
            this_score = trial.intermediate_values[step]
            
            # 같은 단계에서 시도된 다른 trial들의 스코어를 가져옴
            completed_trials = study.get_trials(deepcopy = False, states = (TrialState.COMPLETE,))
            other_scores = [
                t.intermediate_values[step] for t in completed_trials if step in t.intermediate_values
            ]
            other_scores = sorted(other_scores)
            
            # 같은 시도에서 측정된 다른 완료된 trial보다 값이 낮다면 가지치기함
            # Step은 objective function에서 정의된 넘버링을 따라감 : 여기서는 0
            if step >= self._warmup_steps and len(other_scores) > self._warmup_trials:
                if this_score < other_scores[0]:
                    print(f"prune() True : Trial {trial.number}, Step {step}, Score {this_score}")
                    return True
                
        return False
```

### 사용
```python
def objective(trial):
    iris = load_iris()
    classes = np.unique(iris.target)
    X_train, X_valid, y_train, y_valid = train_test_split(
        iris.data, iris.target, train_size = 100, test_size = 50, random_state = 0
    )
    
    loss = trial.suggest_categorical('loss', ['hinge', 'log', 'perceptron'])
    alpha = trial.suggest_float('alpha', 1e-5, 1e-3, log = True)
    clf = SGDClassifier(loss = loss, alpha = alpha, random_state = 0 )
    score = 0
    
    for step in range(0, 5):
        clf.partial_fit(X_train, y_train, classes = classes)
        score = clf.score(X_valid, y_valid)
        
        trial.report(score, step)
        
        if trial.should_prune():
            raise optuna.TrialPruned()
    
    return score

pruner = LastPlacePruner(warmup_steps = 1, warmup_trials = 5)
study = optuna.create_study(direction = 'maximize', pruner = pruner)
study.optimize(objective, n_trials = 100)
```