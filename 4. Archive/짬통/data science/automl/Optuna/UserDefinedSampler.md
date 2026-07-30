-   나만의 샘플링 알고리즘을 실험할 수 있다
-   작업별로 알고리즘을 구현한다
-   다른 최적화 라이브러리를 래핑해서 Optuna 파이프라인에 통합한다

- Sampling 원리는 앞에서 설명했으니 패스
- [BaseSampler](https://optuna.readthedocs.io/en/stable/reference/samplers/generated/optuna.samplers.BaseSampler.html#optuna.samplers.BaseSampler)를 참고하면 좋다
## 예제 : Simulated Annealing Sampler
```python
  

import numpy as np
import optuna


class SimulatedAnnealingSampler(optuna.samplers.BaseSampler):

    def __init__(self, temperature = 100):
        self._rng = np.random.RandomState() # 난수생성기 객체를 만듦
        self._temperature = temperature
        self._current_trial = None

    def sample_relative(self, study, trial, search_space):

        if search_space == {}:
            return {}

        # Simulated Annealing
        # 1. Transition Probability 계산
        prev_trial = study.trials[-2]
        if self._current_trial is None or prev_trial.value <= self._current_trial.value:
            probability = 1.0
        else:
            probability = np.exp(
                (self._current_trial.value) - prev_trial.value
            ) / self._temperature
        self._temperature *= 0.9

        # 2. 이전 결과가 받아진다면 Current Value를 옮김
        if self._rng.uniform(0, 1) < probability: # .uniform : 균등분포
            self._current_trial = prev_trial

        # 3. 현재 지점의 근처에서 파라미터를 샘플링함
        # 여기서 샘플링된 파라미터는 다음 목적 함수의 실행에서 통과될 변수
        params = {}
        for param_name, param_distribution in search_space.items():
            if (
                # isinstance(a, b) : a가 b 타입이면 True, 아니면 False를 반환함
                not isinstance(param_distribution, optuna.distributions.FloatDistribution
                or (param_distribution.step is not None and param_distribution.step != 1)
                or param_distribution.log
            ):
                msg = (
                    "Only suggest_float() with 'step' 'None' or 1.0 and 'log' 'False' is supported"
                )
                raise NotImplementedError(msg)

            current_value = self._current_trial.params[param_name]
            width = (param_distribution.high - param_distribution.low) * 0.1
            neighbor_low = max(current_value - width, param_distribution.low)
            neighbor_high = min(current_value + width, param_distribution.high)
            params[param_name] = self._rng.uniform(neighbor_low, neighbor_high)
        return params

    def infer_relative_search_space(self, study, trial):
        return optuna.samplers.intersection_search_space(study)

    def sample_independent(self, study, trial, param_name, param_distribution):
        independent_sampler = optuna.samplers.RandomSampler()
        return independent_sampler.sample_independent(study, trial, param_name, param_distribution)
```

### 실사용
```python
def objective(trial):
    x = trial.suggest_float('x', -10, 10)
    y = trial.suggest_float('y', -5, 5)
    return x**2 + y

sampler = SimulatedAnnealingSampler()
study = optuna.create_study(sampler = sampler)
study.optimize(objective, n_trials = 100)

best_trial = study.best_trial
print('Best Value : ', best_trial.value)
print('Parameters that achieve the best value : ', best_trial.params)
```
-   위 최적화에서 `x`, `y` 파라미터는 `SimulatedAnnealingSampler.sample_relative` 메서드를 사용해서 샘플링됨

엄밀히 말해, 1번째 Trial에서 `sample_independent` 메서드가 샘플링에 사용되었음. 왜냐면 `infer_relative_search_space()` 내의 `intersection_search_space()`는 완료된 trial이 없으면 탐색 공간을 추론하지 못하기 때문임