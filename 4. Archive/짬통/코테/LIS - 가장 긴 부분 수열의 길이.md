- **배열의 정렬 순서를 유지하면서** **뒤에 오는 숫자는 앞의 숫자보다 커야 함(같아도 안됨)**
- **DP가 어떤 것을 의미하는지를 특히 유의할 것!**
---
1. DP에 들어가는 값의 정의 : `DP[i]`는 i번째 인덱스에서 **끝나는** 최장 증가 부분 수열의 길이를 의미함
	- i번째 인덱스 값을 포함한다는 의미
2. `DP[i]`값은 1부터 시작함

- 따라서 dp의 각 인덱스`i`에 대해서 (반복문 1)
- 처음부터 `i-1`번째 인덱스까지 반복문을 돌린다(반복문 2)
	- 반복문 2의 구성 : `dp[i]`와 이전 인덱스의 값`dp[j]`를 비교했을 때, `dp[i]`가 `dp[j]`보다 크다면
		- `dp[i]`는 `dp[j] + 1`과 `dp[i]` 값 중 큰 값을 선택한다

```python
lst = [] # 배열
dp = [1] * n
for i in range(n):
	for j in range(i):
		if lst[i] > lst[j]:
			dp[i] = max(dp[i], dp[j] + 1)
```

- 위 방법은 $O(N^2)$ 이기 때문에, 이분 탐색을 이용하는 방법이 있다.

## 이분 탐색 이용
- $O(N log(N))$이다. 
```python
def bi_search(lst, target):
	"""
	lst는 dp array
	target은 원래 리스트의 값 
	"""
	left = 0
	right = len(lst) - 1
	
    while left < right:
        mid = (left + right) // 2
        if dp[mid] < target:
            left = mid + 1
        else:
            right = mid
    return right

dp = []
dp.append(lst[0]) # dp에 lst[0]을 넣는다
dp_idx = 0 # dp에 값을 넣는 기준 인덱스 : 마지막 값

for i in range(1, n):
    if dp[dp_idx] < lst[i]:
        dp.append(lst[i])
        dp_idx += 1
    else:
        new_idx = bi_search(dp, lst[i]) # lst[i]가 들어갈 곳을 찾는다
        dp[new_idx] = lst[i]

# dp의 길이를 물어보니까
print(dp_idx + 1)

```

![[LIS-1 1.jpg]]

## 역추적까지 들어간 경우
- LIS의 길이와 **그 구성 원소를 같이 나열**해달라고 했을 때의 풀이법이다.

```text
LIS 구성은 이런 방식으로 이뤄진다
arr = [10, 20, 30, 20]
dp = [1, 2, 3, 2]

- 3번 인덱스의 값이 2인 이유는, 
- LIS에 들어갔을 때 10 20 30이 2번 인덱스까지의 LIS 구성이고
- 3번 인덱스가 들어가면 LIS의 1번 인덱스에 들어가므로, 그 길이는 2를 가지기 때문임

- 아래의 두 방식의 원리는 동일함
```


### 1. $O(N^2)$
```python
import sys

N = int(input()) # 길이
arr = list(map(int, input().split())) # 배열

dp = [1] * N # 초기화는 1로 해준다 : dp는 LIS에 해당 인덱스의 "값"이 들어갔을 때, 그 값 & 인덱스까지 LIS가 가질 수 있는 최대 길이가 됨
dp[0] = 1

# 각 인덱스는 부모 노드(즉 LIS를 구성하는 바로 이전의 작은 값)를 가리킴
path = [-1] * N

for i in range(N):
	for j in range(i):
		if arr[j] < arr[i]:
			dp[i] = max(dp[j] + 1, dp[i])
			path[i] = j # i는 j 뒤에 붙는다는 얘기가 되므로


# 역추적
now_length = max(dp)
print(now_length) # 최대 길이
ans = []

for i in range(N-1, -1, -1):
	if dp[i] == now_length:
		ans.append(arr[i])
		now_length -= 1
        
print(*ans[::-1])
```

### 2. $O(Nlog(N))$


```python
from bisect import bisect_left

N = int(input())
arr = list(map(int, input().split()))

LIS = [arr[0]]
record = [0] * N # 해당 인덱스의 값이 LIS에 들어갔을 때 그 값까지의 "길이" : O(N^2)과 다른 방식임에 유의!
record[0] = 1

for i in range(1, N):
    if arr[i] > LIS[-1]:
        LIS.append(arr[i])
        record[i] = len(LIS)
        
    else:
        idx = bisect_left(LIS, arr[i])
        LIS[idx] = arr[i]
        record[i] = idx + 1 # 인덱스는 0부터 시작하니까 길이는 1을 더해준다
        
now_length = len(LIS)

ans = []
for i in range(N - 1, -1, -1): # record에 대해 반복문이 돌아야 함! 
    if record[i] == now_length:
        now_length -= 1
        ans.append(arr[i])

print(len(ans))  
print(*ans[::-1])
```