- 특정 상황에서 최대한 깊숙이 들어가서 탐색한 다음 다시 돌아가서 탐색하는 방식
- **재귀호출**으로 구현할 수도 있고, **스택**으로 구현할 수도 있다.

- 재귀호출로 구현
```python
def dfs(graph: dict, start: int):
	visited = {i:False for i in graph.keys()}
	def search(current: int):
		for next in graph[current]:
			if not visited[next]:
				search(next)
	search(start)
```
- `visited()`는 방문 여부를 체크하는 리스트
- `next`는 인접 노드를 의미함