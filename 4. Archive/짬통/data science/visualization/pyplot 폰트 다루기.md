1. `fontsize` 인자를 만진다
```python
plt.title('set title', fontsize = 25)
plt.xlabel('xlabel', fontsize = 20)
```

2. 전체 그래프에 대한 설정을 하고 싶다면 `plt.rcParams`에 `key를 전달한다`
```python
parameters = {'axes.labelsize': 25, 'axes.titlesize': 35}
plt.rcParams.update(parameters)

# Key 종류 (x, y가 있다면 하나만 씀)
# axes.labelsize : x, y label의 크기
# axes.titlesize : 축 제목의 크기 
# figure.titlesize : 그림 제목의 크기
# xtick.labelsize : 눈금의 글꼴 크기(x)
# legend.fontsize : 범례의 글꼴 크기
# legend.title_fontsize : 범례 제목의 글꼴 크기
```

3. 