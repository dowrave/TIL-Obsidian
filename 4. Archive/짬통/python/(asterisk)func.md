```python
iqr = np.subtract(*np.percentile(x, [75, 25]))
```
- 여기서 `*`의 역할이 무엇인가?
```python
print(np.percentile(x, [75, 25])) # [33.5, 1.25]
print(*np.percentile(x, [75, 25])) # 33.5 1.25
```
- 여러 개를 반환하는 함수라면, 이를 `Unpacking`해주는 역할이라고 보면 될 듯.