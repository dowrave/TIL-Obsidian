- 공분산행렬을 나타낼 때 주로 사용 -> **상관관계 시각화**
```python
sns.heatmap(df.corr(), 
		   annot = True,
		   vmin = , 
		   vmax = , 
		   fmt = ...,
		   cmap = ,
		   ) 
```
- `annot = True` : 데이터 내에 상관계수 값을 띄울 것인가
- `fmt` : `annot`값을 띄웠을 때, 어떻게 띄울 것인가 (ex : '.1f' - 소수점 첫째 자리)

### Mask
- **`mask = True`인 지점은 표시되지 않음**
```python
mask = np.ones_like(df.corr(), dtype = bool)

sns.heatmap(df.corr(), 
		   annot = True,
		   mask = mask,
		   ...) 
```

- 히트맵의 일부만 보기  (UpperTriangle)
```python
# upper triangle 날리기
corr = df.corr()  
mask = np.zeros_like(corr, dtype=bool)  
mask[np.triu_indices_from(mask)] = True 
sns.heatmap(corr,
			mask=mask, 
			...)
```
- `np.triu_indices_from()` : 전달된 `array`의 `upper triangle`을 가져옴. (`np.tril_indices_from()`은 `lower triangle`이겠쥬?)

- 값에 제한 주기
	방법 1. `vmin`, `vmax` 범위를 준다.
		- `vmin`보다 작은 값은 `cmap`의 하한 색깔을, `vmax`보다 큰 값은 `cmap`의 상한 색깔을 갖는다.
	방법 2. 여러 mask를 적용한다
		- 구분은 `|`로 해준다
		- `mask`에는 `True/False Array` 가 와야 한다. 그러면 이렇게 갈 수 있음.
```python
sns.heatmap(corr,
		   mask = mask | np.where(corr > 0.5, 0, 1)) # mask는 True를 제외하므로 거꾸로 줌
```
- [[np.where]] 