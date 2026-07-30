> 1. 시작 정점만을 MST에 포함한다.
> 2. MST 집합에 인접한 정점 중, 최소 간선으로 연결된 정점을 선택해서 트리를 확장한다.
> 3. 트리가 (N-1) 개의 간선을 가질 때까지 반복한다.

```python
import heapq

graph = [[] for i in range(v + 1)]

for i in range(e):
	a, b, w = map(int, input().split())
	# 그래프 구성은 마음대로 해도 됨(뒤의 주석처럼 해서 간선을 추가하는 느낌도 좋음)
	graph[a].append(w, b) # append(w, a, b)
	graph[b].append(w, a) # append(w, b, a)

visited = [0] * (v + 1)

def prim(start_node):
	
	heap = []
	mst = [] # 혹은 ans도 가능
	heapq.heappush(heap, (0, start_node))

	while heap:
		# weight, prev_node, now_node로 구성해도 됨
		w, now = heapq.heappop(heap) 

		if visited[now] == 0:
			visited[now] = 1
			mst.append(now_node) # 역시 (prev_node, now_node)로 구성해도 됨
			# ans += weight
			for w, next_node in graph[now_node]:
				if visited[next_node] == 0: # 사이클이 아니라면
					heapq.heappush(heap, [w, next_node]) # 푸시

	# return ans

```
