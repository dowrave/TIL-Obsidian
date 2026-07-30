- 랜덤 포레스트 분류기.
```python
from sklearn.ensemble import RandomForestClassifier
clf = RandomForestClassifier(n_estimators = 100,
							 criterion = 'gini'
							 max_depth = None,
							 oob_score = True
)
```
- `n_estimators` : 랜덤 포레스트 나무의 수
- `criterion` : 트리에서 노드를 나눌 때의 지표
- `max_depth` : 총 몇 층으로 만들 것인지 
	- `None` : 각 노드에 들어간 데이터의 수가 `min_sample_splits`보다 작아질 때까지 계속 쪼개진다.
- `oob_score` : `bootstrap = True`일 때만 사용 가능, 일반화 점수에 `oob` 샘플을 씀

----------------------------------
하이퍼 파라미터 튜닝과 관련한 설명
1. `n_estimators` : 과적합을 유발하진 않으나, 시간복잡도를 증가시킬 수 있다.
2. `max_depth` : 트리의 최대 높이를 제어한다. **정확도에 있어 가장 중요한 파라미터 중 하나**이다.
	- 기본값 `None`은 모든 리프 노드가 순수하거나 `min_sample_splits` 미만이 될 때까지 트리가 계속 성장함을 의미함
3. `min_sample_split` : 이 값 이상의 데이터를 갖고 있는 노드는 분할된다.
	- 값을 높이면 총 분할 수를 줄일 수 있어 과적합을 줄일 수 있지만, 너무 크다면 과소적합 가능성도 있다.(너무 적은 하이퍼파라미터를 반영하기 때문에)
	- 일반적으로 `2~6`의 값을 쓰며, 기본값은 `2`이다.
4. `min_samples_leaf` : 분할된 후 노드가 가져야 하는 최소한의 데이터 수. 
5. `max_features` : 랜덤 포레스트에서 사용하는 피쳐의 개수.
	`auto = sqrt` : 전체 데이터의 $\sqrt{n}$ 개의 피쳐만을 선택함
	`log2` : $log_2{n}$ 개의 피쳐를 선택함
	`None` : $n$개의 피쳐를 선택함
6. `max_leaf_nodes` : `None`이라면 트리는 끝없이 자란다. 역시 `split`을 제어함.
7. `max_samples` : 몇 개의 샘플만을 선택할 것인가