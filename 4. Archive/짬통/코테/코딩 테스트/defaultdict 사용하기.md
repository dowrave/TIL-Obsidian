- 왜 쓰는가? : `if dct[i] == 1` 같은 조건문을 쓸 때, 일반적인 `dict`의 경우 키 값 `i`가 없다면 오류를 반환한다. 따라서 이를 더 용이하게 하기 위해 사용함.

```python
from collections import defaultdict

dct = defaultdict(int) # 안에 타입을 넣어줘야 아래처럼 쓸 수 있음
for i, j in items:
	dct[j] += 1
print(dct)
``` 

