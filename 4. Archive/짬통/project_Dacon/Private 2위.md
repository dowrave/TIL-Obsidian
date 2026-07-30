
`AutoML` 모듈 중 `Pycaret을 활용`해 (**Gradient Boosting Classifier, Extra Tree Classifier**) 블렌딩해(모델 가중치 반반) 최종 모델로 사용했습니다.  
실험과정에서 
모델 튜닝과정에서 **사이킷런의 RandomGrid, Optuna, Tune-sklearn 모듈 3가지로 튜닝하며 AUC, F1 스코어 중심으로 모델 선정**했습니다.  
  
전처리는 포지션 15개를 공격수/미드필더/수비수/골키퍼 4가지 항목으로 압축했고, (High-Medium-Low)항목을 수치형으로 변환해 사용했습니다.  
'Age'항목은 카테고리 변수로 시도해봤으나, 수치형으로 테스트 했을 때 성능이 더 좋았고,   
전체적인 수치형 데이터는 단순 zscore로 정규화 해서 사용했습니다. (추가로 사이킷런의 minmax, maxabs, robust 스케일러 테스트)   
또한 'Yeo-Johnson' 변환도 성능이 떨어져 제외하였습니다.

```python
!pip install pycaret[full]
!pip install imbalnced-learn

import pandas as pd
import numpy as np

train = pd.read_csv('./train.csv')
test = pd.read_csv('./test.csv')
df_train = pd.read_csv('./train.csv')
df_test = pd.read_csv('./test.csv')
```

- ['CAM', 'CB', 'CDM', 'CF', 'CM', 'GK', 'LB', 'LM', 'LW', 'LWB','RB', 'RM', 'RW', 'RWB', 'ST']를 포지션에 따라 공격,미드필더, 수비수, 골키퍼로 압축
- ['AttackingWorkRate'], ['DefensiveWorkRate'] 에 대해 'High', 'Medium', 'Low'를 각각 9,5,1로 numeric 데이터로 변환

```python
defender = ["CB", "LB", "RB", "LWB", "RWB"]
attacker = ["ST", "CF", "LW", 'RW']
midfielder = ["CAM", "CDM", "CM", "LM", "RM"]
goalkeeper = ["GK"]

  

for i in range(len(df_train)):
    if(df_train['AttackingWorkRate'][i] == "High"):
        df_train['AttackingWorkRate'][i] = int(9)
   
    elif(df_train['AttackingWorkRate'][i] == "Medium"):
        df_train['AttackingWorkRate'][i] = int(5)

    elif(df_train['AttackingWorkRate'][i] == "Low"):
        df_train['AttackingWorkRate'][i] = int(1)

  

for i in range(len(df_train)):
    if(df_train['DefensiveWorkRate'][i] == "High"):
        df_train['DefensiveWorkRate'][i] = int(9)

    if(df_train['DefensiveWorkRate'][i] == "Medium"):
        df_train['DefensiveWorkRate'][i] = int(5)

    if(df_train['DefensiveWorkRate'][i] == "Low"):
        df_train['DefensiveWorkRate'][i] = int(1)

  

for i in range(len(df_train['Position'])):

    if(df_train['Position'][i] in defender):
        df_train['Position'][i] = "Defender"

    elif(df_train['Position'][i] in attacker):
        df_train['Position'][i] = "Attacker"

    elif(df_train['Position'][i] in midfielder):
        df_train['Position'][i] = "Midfielder"

    elif(df_train['Position'][i] in goalkeeper):
        df_train['Position'][i] = "Goalkeeper"

for i in range(len(df_test)):
    if(df_test['AttackingWorkRate'][i] == "High"):
        df_test['AttackingWorkRate'][i] = int(9)

    elif(df_test['AttackingWorkRate'][i] == "Medium"):
        df_test['AttackingWorkRate'][i] = int(5)

    elif(df_test['AttackingWorkRate'][i] == "Low"):
        df_test['AttackingWorkRate'][i] = int(1)

  

for i in range(len(df_test)):

    if(df_test['DefensiveWorkRate'][i] == "High"):
        df_test['DefensiveWorkRate'][i] = int(9)

    if(df_test['DefensiveWorkRate'][i] == "Medium"):
        df_test['DefensiveWorkRate'][i] = int(5)

    if(df_test['DefensiveWorkRate'][i] == "Low"):
        df_test['DefensiveWorkRate'][i] = int(1)

  

for i in range(len(df_test['Position'])):

    if(df_test['Position'][i] in defender):
        df_test['Position'][i] = "Defender"

    elif(df_test['Position'][i] in attacker):
        df_test['Position'][i] = "Attacker"

    elif(df_test['Position'][i] in midfielder):
        df_test['Position'][i] = "Midfielder"

    elif(df_test['Position'][i] in goalkeeper):
        df_test['Position'][i] = "Goalkeeper"
 
df_train.to_csv('feature_arrange.csv')
df_train.head()

df_test.to_csv('feature_arrange_test.csv')
df_test.head()
```

- 모듈 & 하이퍼파라미터 튜닝
```python
import random
import os
import pandas as pd
import numpy as np
import warnings
warnings.filterwarnings('ignore')

from IPython.core.interactiveshell import InteractiveShell
InteractiveShell.ast_node_interactivity = "all"

class CFG:
    user_seed = 42
    target = 'Prospect'

def seed_everything(seed):
    random.seed(seed)
    os.environ['PYTHONHASHSEED'] = str(seed)
    np.random.seed(seed)

seed_everything(CFG.user_seed)

train = pd.read_csv('./feature_arrange.csv')
test = pd.read_csv('./feature_arrange_test.csv')

```

- `Pycaret` 모델 세팅
	- Stratified K-fold (3fold), SMOTE 아닌 imblearn 모듈 over sampling 사용해서 클래스 불균형 완화
	- tune_model로 모델 튜닝시 search 알고리즘으로 1) sklearn-RandomGridSearch, 2) Optuna, 3) scikit-optimize 알고리즘으로 n_iter 50, 100, 150 결과 비교 (아래에는 결과 도출에 필요한 코드만 작성)
```python
my_model = setup(session_id=CFG.user_seed,
                 data=train,
                 target=CFG.target,
                 normalize=False, normalize_method='zscore',
                 transformation=False,  
                 ignore_features = ['ID'],
                 numeric_features = ['Age'],
                 categorical_features = ['Position', 'PreferredFoot', 'AttackingWorkRate', 'DefensiveWorkRate'],
                 fold_strategy='stratifiedkfold',
                 fold=3,
                 fold_shuffle=True,
                 fix_imbalance = True, # data imbalance 를 sampling method로 보정
                 fix_imbalance_method = imblearn.over_sampling.RandomOverSampler(),
                 use_gpu=True)
```

- `GBM`
```PYTHON
model_gbc = create_model('gbc')

model_gbc3_1 = tune_model(model_gbc, search_library = 'scikit-optimize', optimize='f1', n_iter=100, choose_better=True)
```

- `Extra Tree Classifier`
```python
model_et = create_model('et')

model_et3_1 = tune_model(model_et, search_library = 'scikit-optimize', optimize='f1', n_iter=100, choose_better=True)
```

- Ensemble
```python
blended = blend_models(estimator_list = [model_gbc3_1, model_et3_1], fold = 3, method = 'soft', probability_threshold = 0.5)
```

- 제출
```python
final_model = finalize_model(blended)
pred = predict_model(final_model, data = test)

submit = pd.read_csv('./sample_submission.csv')
submit[CFG.target] = pred['Label']
submit.to_csv('dacon_submit.csv', index = False)
```