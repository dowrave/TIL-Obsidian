- `FIFO(First In First Out)`
- 큐를 사용하면 데이터를 추가한 순서대로 제거할 수 있다

- **리스트**로도 가능하다
	- `pop(0)`은 0번째 인덱스를 제거함
	- `insert(0, x)`은 0번째에 인덱스를 삽입함
	- 그러나 **추천하지 않음** : 파이썬의 리스트는 무작위 접근에 최적화되었기 때문에 `O(n)`에 의해 데이터가 많아질수록 성능이 불리해짐
	- 데이터를 넣을 때마다 인덱스를 한 칸씩 밀어야 하는 걸 생각해보자

- **Deque(덱)**
	- `from collections import deque`
	- `Double-Ended Queue` : 데이터를 양방향에서 추가하고 제거할 수 있다.
	 - 기본적으로 `append(), pop()` 외에도 0번째 인덱스에 값을 추가, 제거할 수 있는 `appendleft(), popleft()`가 있다.
	 - 데이터의 흐름은 뒤에서 앞으로 흐르게 된다.
	 - 기본적으로 `append() - popleft()`를 쌍으로 쓰거나, `appendleft() - pop()`을 쌍으로 쓰면 된다.
```python

>>> from collections import deque
>>>
>>> queue = deque([4, 5, 6])
>>> queue.append(7)
>>> queue.append(8)
>>> queue
deque([4, 5, 6, 7, 8])
>>> queue.popleft()
4
>>> queue.popleft()
5
>>> queue
deque([6, 7, 8])
```
- 리스트와의 차이점 : **링크드 리스트를 쓴다**
	- 따라서 임의의 인덱스에 무작위 접근할 수 없고, **정해진 순서대로 따라 들어가야 하므로** 무작위 접근엔 $O(n)$ 이 된다. 
	- 반면 어떤 값을 **맨 앞 / 맨 뒤에 추가하고 제거하는 데는 $O(1)$이 된다.**  

- 추가 : 도움이 될 만한 메서드
```python
dq.rotate(-1) # dq.append(dq.popleft())
dq.rotate(1) # dq.appendleft(dq.pop())

dq.index(value) # 어떤 value의 idx를 찾아줌

dq.reverse() # 뒤집기
```



- `queue.queue`라는 것도 있는데 여기선 생략함
