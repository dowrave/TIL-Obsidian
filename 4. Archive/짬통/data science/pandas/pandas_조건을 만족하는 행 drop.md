- 조건을 만족하는 row들 drop하기
```python
df.drop(df[df['col'] == 'condition'].index, #.index가 없으면 2차원 배열 -> 적용 X
	   inplace = True)
```
