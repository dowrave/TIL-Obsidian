[BayesianOptimization](https://www.cognex.com/ko-kr/blogs/deep-learning/research/overview-bayesian-optimization-effective-hyperparameter-search-technique-deep-learning-1)
- 미지의 목적 함수 $f$를 최대로 만드는 최적해 $x^*$ 를 찾는 게 목표
-  $x^*$에 빠르게 접근하는 게 목표
- 구성은 2가지임
1. Surrogate Model
2. Acquisition Function

### 1. Surrogate Model
- 현재까지 조사된 $x_t, f(x_t)$ 값을 바탕으로 미지의 목적함수의 형태에 대한 확률적인 추정을 함
- 가장 많이 사용되는 확률 모델이 **Gaussian Process(GP)**임

#### Gaussian Process
- 자세한 내용은 베이지안 확률론 + 확률 / 선형대수학 까지 가므로 생략, 어떻게 동작하는지만 다룸
- 평균 함수 $\mu$와 공분산 함수 $k(x, x`)$ 를 사용하여 함수들에 대한 확률 분포를 표현함
 ![[bayesian-optimization-procedure-example.jpg]]
- 검은 점선이 실제 함수, 실선이 **추정된 함수들의 평균**, 파란 음영이 **각 위치별 표준편차**이다.
- t값(조사 수)이 커질수록 파란 음영이 감소하며, 목적함수값을 최대로 만드는 $x^*$을 찾을 확률이 계속해서 높아지게 될 것이다.

### 2. Acquisition Function
- 현재까지 조사된 결과를 바탕으로, 다음 입력값 후보 $x_{t+1}$을 추천하는 함수
- 이 때 Acquisition Function은 $x^*$을 찾는 데 있어 "가장 유용할 만한 것"이다.
- Multi-Armed Bandit에서 나오는 `Exploitation`과 `Exploration` 개념이 여기서도 적용된다.
	- `Exploitation(착취, 수탈)` : 현재까지 조사된 값 중 가장 높은 값 근처를 조사하는 것이, 최댓값을 찾는 데 유리할 수 있다.
	- `Exploration` : 한편, 불확실성이 큰 영역에 최적 입력값이 존재할 가능성이 있기 때문에, 이 곳을 탐색하는 것도 최댓값을 찾는 데 유리할 수 있다.
	- 이 두 개념은 서로 `Trade-off` 관계이며, 이 둘 간의 강도를 적절하게 조절하는 것이 입력값 탐색에 중요하다. 

#### Expected Improvement(EI)
- `Exploitation`과 `Exploration` 개념 모두 일정 수준 포함하도록 설계되었음. 
	- 현재까지 조사된 점의 함숫값 중 최대 함숫값보다 더 큰 함숫값을 도출할 확률(`Probability of Improvement : PI`)
	- 그 함숫값과 $f(x^{+})$ 간의 차이값(`Magnitude`)
- `PI`와 `Magnitude` 을 고려해 해당 입력값의 `유용성`을 나타내는 숫자(`EI`)를 출력한다.
![[Pasted image 20221122211501.png]]
- $\xi$ 값은 `Exploration - Exploitation` 간의 상대적 강도를 조절하는 파라미터이다.
	- 클수록 `Exploration` 강도가 높고
	- 작을수록 `Exploitation`  강도가 높다
- 위 그림에서 아래에 있는 초록색 영역이 `EI`값이다.

- EI 외에도 `PI`, `Upper Confidence Bound`, `Entropy Search` 등이 있다.