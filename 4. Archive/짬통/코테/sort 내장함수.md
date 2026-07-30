- 알고 있어야 하는 이유 : **단순히 정렬을 알고 있는가**를 묻는 문제가 있기 떄문(알고리즘을 구현할 필요 없이!)

- `병합 정렬`을 사용함 : $O(Nlog(N))$ 을 보장함
```python
lst.sort()
sorted(lst)

# 내림차순
lst.sort(reverse = True)
sorted(lst, reverse = True)

# 다차원 (인덱스 0 : 오름차순, 1 : 내림차순)
lst.sort(key = lambda x: (x[0], -x[1]))

# 글자 길이
lst.sort(key = len)
```
- `.sort()` 
	- 객체에 적용하는 함수임 -> 그 객체에 변경이 일어남
	- 리스트에만 사용 가능
- `sorted()`
	- Iterable 객체에 모두 사용 가능
	- 새로운 객체를 반환함
- 문자의 경우 대문자를 우선함`ord()`


#### dict에 사용
- 위 내용처럼 `sorted`만 사용 가능함
```python
x = sorted(dic.items(), key = lambda x : x[0])
```
> `dic.items()` : 딕셔너리를 이터러블한 객체로 바꿔줌