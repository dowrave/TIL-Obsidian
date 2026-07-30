- `barplot` 내부에 여러 범주가 들어가서 총합과 범주를 동시에 표시해주는 그래프.

#### 1. df.plot
- 코드가 단순하다는 큰 장점이 있음
- 참고) **같은 Row에 있는 다른 Column들을 쌓는 방식**임
	- 즉 같은 Column의 다른 Row들을 쌓는 방식이 아니다.
```python
df.plot(kind='bar', stacked=True)
```

#### 2. matplotlib
```python
fig, ax = plt.subplots()

# 범주 1에 대한 막대그래프를 그린다
ax.bar(agg_tips.index, agg_tips['Male'], label='Male')

# 범주 2에 대한 막대그래프의 시작점은, 이전에 그린 범주의 윗점이다.
ax.bar(agg_tips.index, agg_tips['Female'], 
	   bottom=agg_tips['Male'], # 요게 핵심
       label='Female')
```

- 반복문으로 구현하기 : `bottom`값을 계속 갱신하면 됨
```python
fig, ax = plt.subplots()

# 1번째 btm 값을 잡음
bottom = np.zeros(len(agg_tips))

for i, col in enumerate(agg_tips.columns):
  ax.bar(agg_tips.index, agg_tips[col], bottom=bottom, label=col) # 그림을 그리고
  bottom += np.array(agg_tips[col]) # bottom을 갱신함
```
- 색을 넣고 싶다면 `ax.bar(color = )`을 이용할 수 있다.

#### 3. seaborn
- **seaborn 자체 기능이 없음!**
- 대신 일종의 야매를 부릴 수 있다.
```python
ax = sns.histplot(
    tips,
    x='day',
    # Use the value variable here to turn histogram counts into weighted
    # values.
    weights='tip',
    hue='sex',
    multiple='stack',
    palette=['#24b1d1', '#ae24d1'],
    # Add white borders to the bars.
    edgecolor='white',
    # Shrink the bars a bit so they don't touch.
    shrink=0.8
)
```