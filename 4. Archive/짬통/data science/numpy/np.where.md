- 기본 : 조건을 만족하는 위치의 인덱스를 반환함
```python
arr = np.arange(5, 15)
np.where(arr > 10) 
```

- 심화 : `np.where(condition, a, b)`
	- 조건을 만족하면 a, 만족하지 못하면 b로 바꿈
```python
arr = np.arange(5, 15)
np.where(arr > 10, 0, arr)
```
- 뒤에 `arr`을 주더라도 값을 그대로 잘 유지합니다.