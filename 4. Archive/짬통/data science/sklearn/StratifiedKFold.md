- 설명 : 훈련 데이터를 n개의 split으로 쪼개는데, 매번 쪼개지는 데이터들이 다르다. 아래 예제는 1/10등분을 10번 하는 것
```python
from sklearn.model_selection import StratifiedKFold

skf = StratifiedKFold(n_splits = 10, 
					  shuffle  = True, 
					  random_state = 0)
skf.get_n_splits(train_X, train_y)
```