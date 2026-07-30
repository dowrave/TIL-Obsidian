### 그래프 구현
```python
N, M, v = map(int, input().split())

graph = {}
for i in range(1, N+1):
    graph[i] = []

for _ in range(M):
    a, b = map(int, input().split())
    graph[a].append(b)
    graph[b].append(a)
```

### DFS
```python
dfs_visited = [0] * (N + 1)
# dfs_order = []

def dfs(node):
    dfs_visited[node] = 1
    # dfs_order.append(node)
    # graph[node].sort()
    for next_node in graph[node]:
        if dfs_visited[next_node] == 0:
            dfs(next_node)
```

### BFS
```python
bfs_visited = [0] * (N+1)
# bfs_order = []

def bfs(node):
    from collections import deque
    dq = deque([])

    dq.append(node)
    # bfs_order.append(node)
    bfs_visited[node] = 1

    while dq:
        now_node = dq.popleft()
        # graph[now_node].sort()
        
        for next_node in graph[now_node]:
            if bfs_visited[next_node] == 0:
                dq.append(next_node)
                bfs_visited[next_node] = 1
                bfs_order.append(next_node)
```