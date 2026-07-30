- BayesianOptimization을 적용한 하이퍼 파라미터 탐색. 
- `GridSearchCV`와 `RandomizedSearchCV`를 합친 형태
```python
from skopt import BayesSearchCV

# 파라미터도 그냥 최소, 최댓값만 주면 됨

params_bs = {'max_depth' : (5, 10),
          'n_estimators' : (50, 500),
          'bootstrap' : [True, False],
          'max_features' : ['sqrt', None]
          }
          
# BayesSearch : 모두 디폴트 값임
model = BayesSearchCV(estimator = clf,
                           search_spaces = params_bs,
                           n_iter = 50,
                           cv = None,
                           n_jobs = 1, 
                           verbose = 0,
                           #scoring = ['f1', 'accuracy'],
                           #refit = 'accuracy',
                           random_state = None)
```
- 특이하게 `search_spaces`라는 표현을 쓴다. 
- `param_bs`에 대한 설명 : `value`값에 단일값을 쓸 수 없음. 최소한 튜플 형태로 와야 하는 것 같다.
- `scoring` : 여러 스코어가 온다면 `refit`은 어떤 값을 기준으로 다시 훈련할지를 정할 필요가 있다.
	- 사용 보류 : `mean_test_score`라는 정체 모를 에러가 뜨기 때문에;
[문서](https://scikit-optimize.github.io/stable/modules/generated/skopt.BayesSearchCV.html)[이론 설명 블로그](https://wooono.tistory.com/102) 