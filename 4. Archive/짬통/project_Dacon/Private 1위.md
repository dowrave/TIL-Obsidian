- Automl 모듈 중 하나인 `LightAutoml`을 사용, 파라미터 설정은 직접 값을 바꿔감
```python
import os 
import time 
import numpy as np 
import pandas as pd 
from sklearn.metrics import mean_squared_error 
from sklearn.model_selection import train_test_split 
import torch 
from lightautoml.automl.presets.tabular_presets import TabularAutoML, TabularUtilizedAutoML 
from lightautoml.tasks import Task 
from lightautoml.report.report_deco import ReportDeco
```

- 하이퍼 파라미터 & 랜덤 시드
```python
N_THREADS = 20 
N_FOLDS = 5 
RANDOM_STATE = 42 
TEST_SIZE = 0.2 
TIMEOUT = 3*3600 
TARGET_NAME = 'Prospect'

np.random.seed(RANDOM_STATE) 
torch.set_num_threads(N_THREADS)
```

- 데이터셋 불러오기 &  분리
```python
train_data = pd.read_csv('train.csv')
test_data = pd.read_csv('test.csv')

tr_data, te_data = train_test_split(train_data, 
									test_size=TEST_SIZE, 
									stratify=train_data[TARGET_NAME], 
									random_state=RANDOM_STATE) 
print('Data splitted. Parts sizes: tr_data = {}, te_data = {}'.format(tr_data.shape, te_data.shape))
```

- 수식 & task(`lightautoml`)
```python
def rmse(y_true, y_pred, **kwargs): 
return mean_squared_error(y_true, y_pred, squared = False, **kwargs) 

# lightautoml 기능
task = Task('reg', )
```

- 하이퍼파라미터 세팅
```python
lgb_params = { 'metric': 'RMSE', 
			  'lambda_l1': 1e-07, 
			  'lambda_l2': 2e-07, 
			  'num_leaves': 42, 
			  'feature_fraction': 0.55, 
			  'bagging_fraction': 0.9, 
			  'bagging_freq': 3, 
			  'min_child_samples': 19, 
			  'num_threads': 4 
			  } # cb_params의 파라미터 값 변경 
cb_params = { 'num_trees': 7000, 
			 'od_wait': 1200, 
			 'learning_rate': 0.02, 
			 'l2_leaf_reg': 64, 
			 'subsample': 0.83, 
			 'random_strength': 17.17, 
			 'max_depth': 6, 
			 'min_data_in_leaf': 10, 
			 'leaf_estimation_iterations': 3, 
			 'loss_function': 'RMSE', 
			 'eval_metric': 'RMSE', 
			 'bootstrap_type': 'Bernoulli', 
			 'leaf_estimation_method': 'Newton', 
			 'random_seed': 42, 
			 "thread_count": 4 }
```

- 훈련 & 검증
```python
%%time 
automl = TabularAutoML(task = task, 
					   timeout = TIMEOUT, 
					   cpu_limit = N_THREADS, 
					   reader_params = {'n_jobs': N_THREADS, 'cv': N_FOLDS, 'random_state': RANDOM_STATE}, 
					   general_params = {'use_algos': [['lgb', 'cb']]}, 
					   lgb_params = {'default_params': lgb_params, 'freeze_defaults': True}, 
					   cb_params = {'default_params': cb_params, 'freeze_defaults': True} ) # 모델 생성 
RD = ReportDeco(output_path = 'tabularAutoML_model_report') # 학습한 모델의 결과를 저장할 경로를 지정합니다. 
automl_rd = RD(automl) # 학습한 모델의 결과를 저장합니다. 
oof_pred = automl_rd.fit_predict(tr_data, roles = roles) # fit_predict를 통해 모델을 학습하고 예측값을 반환
```

- 모델 구조 출력
```python
print(automl_rd.model.create_model_str_desc())
```

- feature 중요도 출력
```python
fast_fi = automl_rd.model.get_feature_scores('fast') # 각 feature의 중요도를 보여줌 
fast_fi.set_index('Feature')['Importance'].plot.bar(figsize = (30, 10), grid = True) # 중요도를 그래프로 표현

accurate_fi = automl_rd.model.get_feature_scores('accurate', te_data, silent = False) # 각 feature의 중요도를 보여줌
accurate_fi.set_index('Feature')['Importance'].plot.bar(figsize = (30, 10), grid = True) # 중요도를 그래프로 표현
```

- 테스트 데이터 예측값 출력
```python
te_pred = automl_rd.predict(te_data)

print('Check scores...') 
print('OOF score: {}'.format(rmse(tr_data[TARGET_NAME].values, oof_pred.data[:, 0]))) # OOF score를 출력 
print('HOLDOUT score: {}'.format(rmse(te_data[TARGET_NAME].values, te_pred.data[:, 0]))) # HOLDOUT score를 출력
```

- 모든 훈련 데이터로 훈련 `ㅇㅎ 하이퍼파라미터 찾은 다음엔 모든 훈련 세트로 훈련시켰네`
```python
%%time automl = TabularAutoML(task = task, 
							  timeout = TIMEOUT, 
							  cpu_limit = N_THREADS,
							  reader_params = {'n_jobs': N_THREADS, 'cv': N_FOLDS, 'random_state': RANDOM_STATE}, 
							  general_params = {'use_algos': [['lgb', 'cb']]}, 
							  lgb_params = {'default_params': lgb_params, 'freeze_defaults': True}, 
							  cb_params = {'default_params': cb_params, 'freeze_defaults': True} ) # 모델을 생성 
							
oof_pred = automl.fit_predict(train_data, roles = roles) # train_data를 이용하여 모델을 학습하고, train_data에 대한 예측값을 출력
```

- 모델 구조, 스코어, 예측값 출력
```python
print(automl.create_model_str_desc())
print('Check scores...') 
print('OOF score: {}'.format(rmse(train_data[TARGET_NAME].values, oof_pred.data[:, 0]))) # OOF score를 출력

test_pred = automl.predict(test_data) # test 데이터에 대한 예측값을 출력 
print('Prediction for test_data:\n{}\nShape = {}'.format(test_pred, test_pred.shape)) # 예측값의 형태를 출력
```

- 제출
```python
output = pd.DataFrame({'ID':samp_sub.ID, TARGET_NAME: (test_pred.data[:, 0] > 0.39).astype(int)}) # 예측값을 submission 형태로 변환 
output.to_csv('submission.csv', index=False) # submission 파일을 저장
```