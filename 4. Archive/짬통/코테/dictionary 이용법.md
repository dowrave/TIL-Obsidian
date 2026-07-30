
## 1. 최대 Value를 갖는 Key 값 찾기

1. 1개인 경우만 조회
```python
max_key = max(dict, key = dict.get)
```
- 최댓값이 여러 개라면 모두 보여주지 않음

2. 여러 개인 경우 조회
```python
[k for k, v in dict.items() if max(dict.values()) == v]
```
- `items`로 key, value 값을 모두 끄집어낸 다음 value 값이 value 리스트 중 최댓값인 key만 찾아내는 방식
