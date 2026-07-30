```python
graph = [[] for _ in range(N + 1)]
visited = [0] * (N + 1)
count = 1 # 방문 순서

# 그래프가 대충 주어졌다고 치자

def dfs(now_node):
	visited[now_node] = count
	for i in graph[now_node]:
		if visited[i] == 0 :
			count += 1
			dfs(i)
```

