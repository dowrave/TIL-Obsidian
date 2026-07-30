- 어떤 열을 datetime으로 바꾸고 싶을 때 사용한다
```python
pd.to_datetime('date_column')
```

- `datetime` 자료형이 되면, 이런 방식으로 추출하고 싶은 것만 추출할 수 있음
```python
df['time'].dt.date
```