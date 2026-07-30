[깃허브 정리 내용](https://github.com/dowrave/TIL/blob/main/FeatureEngineering/6_TargetEncoding.ipynb) 
- 원핫인코딩, 레이블인코딩이랑은 또 다른 개념이다.
- `groupby()[].transform()` 을 통해, `groupby`로 묶이나 `transform` 으로 묶이나 그 내용은 동일하다.
	- 예를 들어 `'mean'`이 들어갔다면 해당 그룹들 간의 평균 값으로 데이터를 뽑을 수 있다는 거임

- 근데 이것에는 단점이 있읍니다.
	- 새로운 범주가 나타났을 때 취약함 (새로운 범주로 두지 않음)
	- 희귀한 범주 : 평균값 자체는 데이터의 수를 알 수 없음 -> 과적합 가능성

## Smoothing
- 평균값으로 타겟인코딩을 하는 것의 단점을 방지하고자 등장한 개념이다.
- **범주 내의 평균과 전체 평균을 혼합**하는 것이다.
	- 희귀값의 평균은 범주 평균에 대한 가중치가 적다
	- 누락된 범주는 전체 평균을 이용한다

$$ encoding = weight * InCategory + (1 - weight) * overall $$
$$ weight = \frac{n}{(n+m)} $$
- 여기서 $n$은 해당 범주의 총 출현 수, $m$은 `Smoothing Factor`이다. 
	- $m$ 값이 클수록 전체 추정 가중치가 커진다.
	- $m$ 값을 선택할 때, 범주가 얼마나 noisy한지를 고려해야 한다.
		- 범주의 데이터마다 값의 변동이 크다면 m값을 키워야 하고
		- 값의 변동이 작은 편이라면 m값을 작게 해도 좋다

#### 타겟 인코딩이 유용한 경우
- **카디널리티가 높은 경우(낮은 중복도, 많은 고윳값)** 
	- 원핫인코딩은 차원이 너무 커질 수 있고, 레이블 인코딩은 부적합할 수 있다.
	- 타겟 인코딩은 타겟과의 관계를 고려해 범주의 숫자를 추출해낸다.
- 사실 대부분의 Nominal 피쳐에 대해 적용해도 무방하다. 특성의 수치적인 성능이 좋지 않더라도 범주형 피쳐가 중요한 경우도 있기 때문임

#### 실제 활용 시 주의사항
- 꼭 반드시 **일부 샘플에 대해서만 평균 샘플링을 해야 한다**
	- 만약 그렇지 않다면 의미가 아예 없는 데이터도 의미 100%라고 학습하게 되기 때문임(과적합)

### Python
```python
from category_encoders import MEstimateEncoder

# 일부 샘플로만 인코딩하는 건 과적합 방지를 위해 필수적이다!
X_encode = X.sample(frac = 0.25)
y_encode = y[X_encode.index]

X_pretrain = X.drop(X_encode.index)
y_train = y[X_pretrain.index]

encoder = MEstimateEncoder(cols = ['col'], m = 5.0)
encoder.fit(X_encode, y_encode)
```
- 아예 별도의 라이브러리니까 유의 `category_encoders`
- 쉽게 생각하면 `groupby()[].transform()`을 다른 방식으로 한 거다!
	- `cols`에는 `groupby(cols)`인거고
	- `[]` 값에는 `y_encode`가 들어간 것

- 이를 시각화하는 코드는 이렇다
```python
plt.figure(dpi = 90)
ax = sns.distplot(y, kde = False, norm_hist = True)
ax = sns.kdeplot(X_train.col, color = 'r', ax = ax)
```

-----------
[Feature Engineering - Exercise](https://www.kaggle.com/code/hyeontaelee/exercise-target-encoding/edit) 내용 중
- 예제에서는 `XGBoost`를 이용했음
- 대부분의 **이름으로 되어있는 데이터는 타겟 인코딩을 시도할 만 하다** : 희귀한 범주가 있기 때문
- 오버피팅을 피하기 위해, `MEstimateEncoder`를 사용할 때 `홀드아웃` 데이터를 사용한다
- 측정 수치로 예제에선 `RMSLE`를 사용한다 : `RMSE`에 로그를 취한 값임
$$\sqrt{\frac{1}{n}\Sigma^n_{i=1}(log(p_i+1) - log(a_i + 1))^2}$$
	- 만약 인코더를 사용한 수치가 그렇지 않은 수치보다 낮게 나왔다면, 인코딩을 통해 얻은 추가 정보는 인코딩에 사용되는 데이터 손실을 보충할 수 없었을 가능성이 높다. 

##### 홀드아웃을 쓰지 않는 경우(전체 훈련 데이터에 인코딩하는 경우)
- 일부러 타겟값과 아예 **관계가 없는 특성**을 추가했을 때 발생할 문제 : 이 값으로 RMSLE 스코어를 측정하면 거의 완벽한 스코어가 나온다.
- 이게 가능한 이유는 인코더를 훈련할 때 사용한 데이터셋과 같은 세트에서 `XGBoost`를 훈련시켰기 때문이다 : 만약 훈련 시 홀드아웃 세트를 대신 사용했다면, 이 가짜 인코딩 중 어떤 것도 훈련 데이터로 전송되지 못했을 것이다(실제로 예제에서는 `df.sample(frac=)` 으로 `X_encode`와 `X_pretrain`을 구분했음)