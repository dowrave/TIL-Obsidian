- N * N 크기의 체스판에 N개의 퀸을 놓는 경우의 수를 구하시오
- [[백트래킹]]으로 풀 수 있음

## 풀이

```python
lst = [-1] * N
visited = [False] * N 
```
- `lst` : 인덱스는 n번째 줄(0 ~ N-1), 인덱스의 값 v는 n번째 줄의 v번째 칸(0~N-1)
- `visited` : 여기의 인덱스는 위에서 `v값`임

```python
# 값 N에 대해서 N * N 크기의 체스판에 N개의 퀸을 놓는 경우의 수
def n_queens(x): # x는 인덱스
	if x == N:
		# 종결조건
		return

	for i in range(N):
	
		if visited[i]:
			continue

		lst[x] = i # condition(x)에서 쓰기 위해 미리 값을 넣어둔다
		if condition(x):
			visited[i] = True
			n_queens(x+1)
			visited[i] = False
		lst[x] = -1

def condition(x):
	for j in range(x): # x번째 인덱스 이전까지 반복을 돌림
		if abs(lst[x] - lst[j]) == abs(j - x): # 대각선 조건(2개 모두 포함하는 식)
			return False
	return True
```
- `visited`를 따로 만들어두지 않을 경우 `condition`에서 이전에 등장한 적 있는지 여부를 추가로 검토할 필요가 있다. 그리고 그게 더 오래 걸린다.