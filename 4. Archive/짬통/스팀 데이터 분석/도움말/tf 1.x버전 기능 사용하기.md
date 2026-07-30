- 2.x버전에서 1.x버전의 기능을 사용하고 싶다면, `tf.compat.v1`에 들어가면 있다.

- 아예 복붙해서 쓰고 싶으면 이런 식으로 써도 됨
```python
import tensorflow.compat.v1 as tf
from tensorflow.compat.v1 import estimator as tf_estimator
```

- 근데 그럴거면 **그냥 텐서플로우를 1.x 버전으로 까는 게 낫지 않을까**?
	- 구버전 설치가 안된다! 2.8버전 이상밖에 없음