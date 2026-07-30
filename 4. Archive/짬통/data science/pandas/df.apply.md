1. 기본
```python
df['new_col'] = df['col'].apply(function)
```

2. 서로 다른 `column`에 있는 값들을 하나의 `column`에 합치기
```python
# 목표 : 2020-01-01, 12, 50 (셋 모두 string)의 값들을 하나에 column에 합치기
# 형태 : 날짜 시간:분:00
race['race_dt'] = race[['race_date', 'hour', 'minute']]
						.apply(
		                        lambda x : f"{x[0]} {x[1]}:{x[2]}:00", 
		                        axis = 1
		                        )
race['race_dt'] = pd.to_datetime(race['race_dt'])
```
- `.apply()`함수를 적용할 때 여러 `column`에 적용하고 싶다면 똑같이 `[[]]`로 묶어주면 된다.

3. **함수가 여러 인자를 받는 경우**
