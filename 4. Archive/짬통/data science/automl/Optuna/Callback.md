### 예제 ) Pruning이 연속으로 일어났을 때 중지하기
```python
import optuna

class StopWhenTrialKeepBeingPrunedCallback:
    
    def __init__(self, threshold: int):
        self.threshold = threshold
        self._consequtive_pruned_count = 0
    
    # def func() -> None : 리턴값 명시해줌
    def __call__(self, study: optuna.study.Study, trial: optuna.trial.FrozenTrial) -> None:
        
        if trial.state == optuna.trial.TrialState.PRUNED:
            self._consequtive_pruned_count += 1
        
        else:
            self._consequtive_pruned_count = 0
            
        if self._consequtive_pruned_count >= self.threshold:
            study.stop()

def objective(trial):
    if trial.number > 4:
        raise optuna.TrialPruned
        
    return trial.suggest_float('x', 0, 1)

import logging
import sys

optuna.logging.get_logger('optuna').addHandler(logging.StreamHandler(sys.stdout))

study_stop_cb = StopWhenTrialKeepBeingPrunedCallback(2)
study = optuna.create_study()
study.optimize(objective, n_trials = 10, callbacks = [study_stop_cb])

```