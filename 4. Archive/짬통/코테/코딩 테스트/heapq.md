```python
import heapq

# 최소힙
heapq.heappush(arr, value)
heapq.heappop(arr)
heapq.heapify(arr)

# 최대 힙
heapq.heappush(heap, (-value, value)) # 앞쪽 원소가 정렬 기준임을 이용해 튜플로 넣고, 뒤의 값을 이용하는 방식.
```

- `heapq.nlargest(n, arr)`
```python
x = [1, 2, 3, 4, 100, 102, 101]
heapq.nlargest(len(x), x)

# [102, 101, 100, 4, 3, 2, 1]
```
- `nlargest`를 쓰면 최댓값부터 가장 큰 n개의 수를 뽑을 수 있다. **최소힙으로 구현하되 최댓값을 제외하고 싶다면 위 결과로 나온 리스트에서 1번째 인덱스부터 다시 인덱싱해서 힙으로 만들면 됨.**