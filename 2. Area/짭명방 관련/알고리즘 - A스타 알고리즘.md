시작 노드 ~ 목표 노드까지의 최단 경로를 찾는 데 사용된다. `휴리스틱 함수(대충 어림한 함수)`를 이용해 탐색 방향을 결정, 이를 통해 탐색 효율을 높인다. 현재 노드에서 목표 노드까지의 예상 비용을 추정한다.

```python
import heapq

def heuristic(a, b):
    return abs(a[0] - b[0]) + abs(a[1] - b[1])

def astar(grid, start, goal):
    neighbors = [(0,1),(0,-1),(1,0),(-1,0)]
    
    close_set = set()
    came_from = {}
    gscore = {start:0}
    fscore = {start:heuristic(start, goal)}
    oheap = []

    heapq.heappush(oheap, (fscore[start], start))
    
    while oheap:
        current = heapq.heappop(oheap)[1]
        
        if current == goal:
            data = []
            while current in came_from:
                data.append(current)
                current = came_from[current]
            return data[::-1]

        close_set.add(current)
        for i, j in neighbors:
            neighbor = current[0] + i, current[1] + j
            tentative_g_score = gscore[current] + heuristic(current, neighbor)
            if 0 <= neighbor[0] < grid.shape[0]:
                if 0 <= neighbor[1] < grid.shape[1]:                
                    if grid[neighbor[0]][neighbor[1]] == 1:
                        continue
                else:
                    # 격자 밖
                    continue
            else:
                # 격자 밖 
                continue
                
            if neighbor in close_set and tentative_g_score >= gscore.get(neighbor, 0):
                continue
                
            if  tentative_g_score < gscore.get(neighbor, 0) or neighbor not in [i[1]for i in oheap]:
                came_from[neighbor] = current
                gscore[neighbor] = tentative_g_score
                fscore[neighbor] = tentative_g_score + heuristic(neighbor, goal)
                heapq.heappush(oheap, (fscore[neighbor], neighbor))
                
    return False
```

## 동작 과정
1. 시작 노드를 `오픈 리스트`에 추가한다.
2. `오픈 리스트`에서 `가장 낮은 총 비용(현재까지의 총 비용 + 휴리스틱 함수값)을 가진 노드를 선택`한다.
3. 선택한 노드를 `오픈 리스트에서 제거, 클로즈 리스트에 추가`한다.
4. 선택한 노드의 `이웃 노드를 탐색`한다.
	- 이웃 노드가 `클로즈 리스트`에 있다면 무시한다.
	- 이웃 노드가 `오픈 리스트`에 없다면 `오픈 리스트`에 추가하고, 현재 노드를 부모로 설정한다. 
	- 이웃 노드가 오픈 리스트에 있고, 현재 경로를 통해 더 낮은 비용으로 도달할 수 있다면 부모를 현재 노드로 변경한다.
5. 목표 노드가 클로즈 리스트에 추가되면 탐색을 종료, 부모 노드를 따라가며 최단 경로를 구성한다.
6. 오픈 리스트가 비어있게 되면 경로가 존재하지 않는 것으로 판단, 탐색을 종료한다.

[[우선순위 큐]]를 사용해 탐색 과정에서 가장 낮은 `fscore`를 가진 노드를 선택한다. 탐색하면서 이웃 노드의 `gscore`와 `fscore`를 업데이트, 최종적으로 목표 노드에 도달 시 `came_from` 딕셔너리로 최단 경로를 구성한다.

2D 격자 맵이 기준이지만, 그래프 구조로 일반화해서 다양한 문제에 적용할 수 있다.

### fscore, gscore
- `gscore` 
	- 시작 노드 ~ 현재 노드까지의 실제 경로 비용, 거쳐온 경로의 총 비용이다.
	- 현재 노드의 `gscore`는 부모 노드의 `gscore` + `현재 노드와 부모 노드 사이의 비용`이다.

- `fscore`
	- 현재 노드 ~ 목표 노드까지의 예상 경로 비용
	- `fscore` = `gscore` + `heuristic(n)`
	- 여기서 `heuristic(n)`은 현재 노드 ~ 목표 노드까지의 예상 비용을 추정하는 휴리스틱 함수이다.
	- `fscore`는 현재 노드까지의 실제 비용 + 목표 노드까지의 예상 비용을 합친 값이다.

휴리스틱 함수는 최적의 경로를 보장하지 않지만, 대부분의 경우 효과적인 탐색을 가능하게 한다.

