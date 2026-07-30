- 2개 이상의 데이터가 있는 데이터들을 True로 갖는 Mask
```python
mask = df[col].value_counts() >= 2 # col에 대한 True, False로 나옴
df[df[col].map(mask)] # 시리즈에만 적용 가
```