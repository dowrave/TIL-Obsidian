- 자료구조 `Queue`를 이용함
```python
from collections import deque

graph = [[] for _ in range(N + 1)]
visited = [0] * (N + 1)
dq = deque([])

def bfs(start_node):

	visited[start_node] = 1
	dq.append(start_node)
	
	while dq:
		now_node = dq.popleft()
		for i in graph[now_node]:
			if visited[i] == 0:
				visited[i] = 1
				dq.append(i)
```

- 예제들은 모두 가장 기본적인 것만 구현한 거고 더 디테일한 거는 처한 상황마다 다를 거임