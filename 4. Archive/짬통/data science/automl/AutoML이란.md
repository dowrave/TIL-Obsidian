[원문](https://medium.com/daria-blog/automl-%EC%9D%B4%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%BC%EA%B9%8C-1af227af2075)
현실에서 머신러닝 모델링을 하려면
	- 문제 정의
	- 데이터 수집
	- 전처리
	- 모델 학습 & 평가
	- 서비스 적용
이라는 과정을 거치기까지 많은 전문가들의 시간과 노력이 요구된다.

`AutoML`은 이러한 위의 과정을 되풀이하면서 발생하는 비효율적인 작업을 최대한 자동화하여 생산성과 효율을 높이기 위해 등장했다. 데이터 전처리, 알고리즘 선택, 튜닝까지의 모델 개발자의 개입을 최소화하기 위한 연구가 오랫동안 진행되어 왔다.

### AutoML로 해결하려는 문제
1. **하이퍼파라미터 성능 & 평가, 최상의 성능을 갖는 모델 찾는 과정을 자동화**(`CASH` : Combined Algorithm Selection and Hyperparameter optimization)

2. **인공 신경망** 기술을 쓸 때, **문제에 적합한 아키텍처를 찾는 과정을 자동화**(Neural Architecture Search : `NAS`)

#### CASH
- 모델 개발자는 학습 준비를 마친 후, 주어진 컴퓨팅 자원과 시간을 최대한 활용하여 최상의 모델을 만들어야 한다. 즉, 적합한 알고리즘 + 성능을 극대화할 하이퍼파라미터를 찾아야 한다.

- CASH 문제에 대한 접근법
	- 전통적인 최적화 : `grid search`, [[RandomizedSearchCV]], `genetic algorithm`, `simulated annealing`
	- 강화학습 : `successive-halving`, `hyperband` [[HyperBand]]
	- [[BayesianOptimization]] : `Gaussian Process(GP)`, `Sequential Model-based Algorithm Confiugration(SMAC)`, `Fast Bayesian Optimization for Data Analytic Pipelines(FLASH)`, `FABOLAS`
	- 스까 : `BOHB`, `Matrix Factorization`

#### NAS
- 2019년 논문에 의하면 NAS가 기존의 사람에 의해 설계된 모델의 성능보다 뛰어날 수 있음이 보고되었다.
- NAS 문제를 풀기 위한 방법은, `탐색 공간 정의`, `알고리즘 선택`에 따라 나뉠 수 있다.
- CASH와 유사하게 `random search`, `BO`, `Evolutionary Algorithm`, `RL` 등의 탐색 알고리즘을 적용하는 방법들이 연구되어 왔다
- 최근에는 탐색 공간을 연속 변수로 `relaxation`하여 `gradient-based optimization` 방법을 적용해 탐색을 더 효율화하는 연구도 진행되고 있다. 

- NAS 주요 방법들
	- Evolutionary Algorithm : `NEAT`, `AmeobaNet`, `Hierarchical NAS`, `JASQNet`
	- Reinforcement Learning : `NAS with RL` , `NASNet`, `MnasNET`, `ENAS`
	- Relaxation : `One-Shot NAS`, `DARTS`, `ProxylessNAS`
