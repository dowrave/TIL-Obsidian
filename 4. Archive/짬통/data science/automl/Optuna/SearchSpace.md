```python
import optuna
import optuna.trial.Trial as trial

def object(trial):
    optimizer = trial.suggest_categorical('optimizer', ['MomentumSGD', 'Adam'])
    
    num_layers = trial.suggest_int('num_layers', 1, 3)
    num_channels = trial.suggest_int('num_channels', 32, 512, log = True)
    num_units = trial.suggest_int('num_units', 10, 100, step = 5)
    
    dropout_rate = trial.suggest_float('dropout_rate', 0.0, 1.0)
    learning_rate = trial.suggest_float('learning_rate', 1e-5, 1e-2, log = True)
    drop_path_rate = trial.suggest_float('drop_path_rate', 0.0, 1.0, step = 0.1)
```
- `keras_tuner`에서 쓴 거랑 원리는 동일함
- 파이썬 문법처럼 쓸 수 있음 : `Branch`, `Loop`
```python
import sklearn.ensemble
import sklearn.svm
import torch
import torch.nn as nn

# Branch 예제
def objective(trial):
    classifier_name = trial.suggest_categorical('classifier', ['svc', 'RandomForest'])
    if classifier_name == 'SVC':
        svc_c = trial.suggest_float('svc_c', 1e-10, 1e10, log = True)
        classifier_obj = sklearn.svm.SVC(C = svc_c)
        
    else:
        rf_max_depth = trial.suggest_int('rf_max_depth', 2, 32, log = True)
        classifier_obj = sklearn.ensemble.RandomForestClassifier(max_depth = rf_max_depth)

# Loop 예제
def create_model(trial, in_size):
    n_layers = trial.suggest_int('n_layers', 1, 3)
    
    layers = []
    for i in range(n_layers):
        n_units : trial.suggest_int('n_units_l{}'.format(i), 4, 128, log = True)
        layers.append(nn.Linear(in_size, n_units))
        layers.append(nn.ReLU())
        in_size = n_units
        
    layers.append(nn.Linear(in_size, 10))
    
    return nn.Sequential(*layers)
```

- 주의 : 최적화의 난이도는 파라미터 수에 Exponential하게 증가하므로 중요하지 않은 파라미터는 제거해주는 게 좋다
