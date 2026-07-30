```python
# 1. string이 아닌 경우까지 포함하면
df['New'] = df['Old1'].astype('str') + df['Old2'].astype('str')

# 2. 2개가 넘는 데이터들을 연결한다면
df['New'] = (df[['Old1', 'Old2', ...]]
									 .astype('str')
									 .agg('-'.join, axis = 1)
									 )
```
- 아래 방법을 추천함