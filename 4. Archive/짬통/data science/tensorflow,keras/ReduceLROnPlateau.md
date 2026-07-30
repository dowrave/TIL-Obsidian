- 스코어의 상승폭이 감소할 때 학습률을 감소시킴
```python
from tensorflow.keras.callbacks import ReduceLROnPlateau

reduce_lr = ReduceLROnPlateau(monitor = 'val_loss',
							 factor = 0.1, 
							 patience = 10,
							 min_lr = 0.001,
							 ) 
```
- `monitor` : 관찰되는 값
- `factor` : 학습률을 감소시킬 때, 기존 학습률에 곱해지는 값. $lr = lr \times factor$
- `patience` : 스코어가 증가하지 않는 상태를 얼마나 허용할 것인지
- `min_lr` : 감소되는 학습률의 하한