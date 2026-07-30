- 어떤 경기에서 특정 시간 간격으로 위도, 경도 데이터가 주어졌다.
	- 이 때, 매 순간의 속도를 측정할 수 있을 것이다. 이 때 다음 row의 데이터를 이전 row로 옮길 필요가 있다.
	- 근데 주자별로 구분은 되어야 한다(다음 인덱스가 다른 주자의 정보라면 이전 row로 옮기면 안됨)
	- 이 때 `groupby`를 쓸 수 있다
```python
test_race['next_lat'] = test_race.groupby('program_number')['latitude'].shift(-1)
test_race[test_race['program_number'] == '1'][:]
```
- `groupby`로 묶었을 때 데이터의 순서가 보존되는가를 살펴볼 필요가 있는데, 잘 보존되는 것 같다.