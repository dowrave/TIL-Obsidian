- **그래프 내의 모든 정점을 가장 적은 비용으로 연결**한다.


```python
# 정점 수 v, 간선 수 e
# edge_arr = [(w, a, b), ...]

def kruksal():

	# 1. 오름차순 정렬
	edge_arr.sort() 
	
	# 2. 간선 선택
	parent = [i for i in range(v + 1)]
	
	# union-find : 사이클 체크
	def find(x):
		if parent[x] != x:
			parent[x] = find(parent[x])
		return parent[x]
	
	def union(a, b):
		if a == b:
			return
	
		a = find(a)
		b = find(b)
	
		if a > b: a, b = b, a # swap
	
		parent[b] = a
	
	for w, a, b in edge_arr:
	
		# 두 노드의 루트 노드가 같다 = 연결되어 있다
		# 이들을 새로 연결한다면 사이클이 발생함
		if find(a) == find(b):
			continue
	
		union(a, b)

```

> 1. 간선을 가중치의 오름차순으로 정렬한다
> 2. 간선 리스트에서 사이클을 형성하지 않는 간선을 선택한다
>> - 낮은 가중치 값을 먼저 고른다
>> - 여기서 **사이클을 형성하지 않는다**를 점검하는 조건이 [[유니온 파인드]]이다. 
>3. 해당 간선을 MST의 집합에 추가한다.