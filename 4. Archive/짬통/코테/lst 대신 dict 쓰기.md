1. dict는 키 값을 찾는데 $O(1)$을 쓰지만, lst는 최대 $O(n)$이 걸림
2. 공간 효율성 : Edge가 Node보다 훨씬 적은 희소 그래프라면 dict가 더 공간 효율적일 수 있다. 필요 없는 공간을 dict는 저장하지 않기 때문.
3. 업데이트 속도도 빠름.

```python
distances = {node: float('inf') for node in graph}
```