- 하이퍼 파라미터 찾기 : 어떤 범위를 준다.
	- `train_test_split`, `Stratified`, `CV` 개념까지 다 들어가 있음!
- [[RandomForestClassifier]]를 예시로 보자.
```python
from sklearn.model_selection import RandomizedSearchCV

params = {'max_depth' : list(range(5, 16)),
          'n_estimators' : list(range(50, 201)),
          'bootstrap' : [True, False]
          }

model = RandomizedSearchCV(estimator = ,
						  param_distributions = params,
						  n_iter =,
						  cv = None,
						  scoring = ['accuracy', 'f1'],
						  n_jobs = None,
						  refit = True)
```
- `n_iter` : 파라미터 쌍의 개수
- `n_jobs` : 병렬실행 수. 보통 `-1`로 지정해 모든 프로세서 동원함
- `cv` 
	-  `estimator`가 분류 모델이고 `y`가 이진 / 다중 클래스라면 `StratifiedKFold`가 실행된다. 나머지는 `KFold`가 실행됨.
- `refit` : 가장 좋은 파라미터로 다시 훈련함


### 결과에 대한 스코어 보기
- 플로팅에 쓸 수 있음
```python
# model은 위에서 한 것처럼 cv 객체임
model.cv_results_['params'] # 파라미터 쌍
model.cv_results_['mean_test_score'] # 테스트 스코어
```