- 모든 분기점을 검색하면서 탐색함
- **`Queue`를 사용하는 게 일반적**이다
- 가중치가 없는 그래프일 때 최단 경로를 구할 수 있다
```python
def bfs(graph: dict, start: int):
	visited = {i : False for i in graph.keys()}
	queue = [start]
	visited[start] = True
	while len(queue) > 0:
		current = queue.pop(0)
		for next in graph[current]:
			if not visited[next]:
				queue.append(next)
				visited[next] = True
```


- 가중치가 있는 그래프에서는, [[그래프_다익스트라]]이 있다.