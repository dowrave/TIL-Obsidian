- SQL 기능으로 먼저 배웠음 : 이전`lag` / 다음`lead` row에 있는 어떤 값을 이번 row에 가져오는 것
- pandas에서는 `.shift()`로 구현할 수 있다. 
```python
df['lag(a,1)'] = df['a'].shift(1)
df['lag(a,5)'] = df['a'].shift(5)
df['lead(b,2)'] = df['b'].shift(-2)
df['lead(b,4)'] = df['b'].shift(-4)
```
- `.shift()` 내부 값이 `양수라면 이전 열(Lag)`의 값을, `음수라면 다음 열(Lead)`의 값을 가져온다.
