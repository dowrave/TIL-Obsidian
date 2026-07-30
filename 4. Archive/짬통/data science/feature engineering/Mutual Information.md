[깃허브 정리 내용](https://github.com/dowrave/TIL/blob/main/FeatureEngineering/2_MutualInformation.ipynb)
새로운 데이터셋을 볼 때, 아무런 안내도 없이 수백, 수천 개의 특성을 마주하게 된다. **어디에서 시작할지 모르겠다면, `Mutual Information`을 살펴보는 것이 좋다.**

- 사용과 해석이 쉬움
- 효율적인 연산
- 이론적으로 잘 정립됨
- 오버피팅에 강함
- 선형적인 관계만을 볼 수 있는 `Correlation`에 비해, 어떤 종류의 관계성이라도 볼 수 있다.

### 측정하는 것
- `불확실성`을 측정한다.
	- 한 `Quantity`에 대한 지식이 생겼을 때, 이 지식으로 다른 `Quantity`의 불확실성을 얼마나 줄일 수 있는가?를 의미한다.
	- 정보 이론에서는 불확실성을 `엔트로피`라고 한다. "해당 변수의 발생을 설명하기 위해 필요한 예/아니오 질문의 갯수"를 의미함.
	- `MI`는 피쳐에 대해 기대하는 답변의 수이다.

### MI Score 해석하기
- 최솟값은 0으로 두 변수가 독립적임을 의미한다
- 최댓값은 없지만 2를 넘기 쉽지 않다. 로그 스케일이기 때문.

### 기억할 것들
- `MI`는 타겟을 예측하는 특성의 잠재력을 파악하는 데 도움이 된다.
- 단, 특성 간의 상호작용을 탐지할 수는 없다. 일변량 측정법이다.
- 실제 유용도는 어떤 모델을 함께 쓰는가에 의존한다. 타겟 간의 관계를 모델이 학습할 수 있을 때 유용하다.
	- 높은 MI 스코어를 가졌다고 해서 모델이 이를 바로 이용할 수 있다는 의미는 아니며, 연결을 표시하기 위해 특성을 변환할 필요가 있을 수 있다.

### 어떻게 씀?
- 타겟값이 연속형이면 `regression`, 범주형이면 `classif`을 쓴다.
```python
from sklearn.feature_selection import mutual_info_regression 
from sklearn.feature_selection import mutual_info_classif 

X = df.copy()
y = X.pop('price') # 타겟

# Label Encoding은 이런 방법도 있다
for colname in X.select_dtypes('object'):
    X[colname], _ = X[colname].factorize()

# 모든 이산형 변수가 Int dtype을 가짐 
discrete_features = X.dtypes == int

# 여기만 봐도 무방함 ##################################################
mi_scores = mutual_info_regression(X, y, discrete_features = discrete_features)
mi_scores = pd.Series(mi_scores, name = 'MI Scores', index = X.columns)
mi_scores = mi_scores.sort_values(ascending = False)
```
- `discrete_features` : 이산형 특성은 따로 `labelencoding`을 한 뒤 따로 전달한다.

#### 주의할 점
- 예제에서도 보여주듯,  `MI Scores`에서 수치가 낮다고 해서 아예 무시할 만한 내용도 아니다. 자체로는 점수가 낮은 특성이더라도 다른 특성에 영향을 미치는 요소일 수 있기 때문이다(`Horsepower`와 `Price`에 들어가는 `hue :  Fuel-type`)
- 이러한 `특성 간의 상호작용`을 탐색하는 데 있어, **도메인에 대한 지식**은 많은 도움이 될 수 있다.
