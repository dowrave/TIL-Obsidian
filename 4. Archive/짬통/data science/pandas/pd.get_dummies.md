- 판다스 자체에서 OneHotEncoding을 수행하는 방법임.
```python
one_hot = pd.get_dummies(df['col'],
						prefix = 'col')

# 원핫인코딩 결과를 원본 데이터프레임에 반영하기
df = df.drop('col', axis = 1)
df = df.join(one_hot)
```
- 파라미터 설명
	- `prefix` : `col` 이름을 값 앞에 붙여줌
		- `prefix_sep = _` : `col` 이름과 `val` 이름 사이에 들어가는 구분자. 기본값 `_`  
- 예제
```python
df = pd.DataFrame({
          'A':['a','b','a'],
          'B':['b','a','c']
        })

one_hot = pd.get_dummies(df['B'])

df = df.drop('A', axis = 1)
df.join(one_hot)
```