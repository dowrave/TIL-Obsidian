- 범주형 데이터를 모델에 넣기 위해선 수치화하는 과정이 필요하다.
- 그런데 같은 칼럼에 `0, 1, 2, 3, 4, ...` 순으로 넣으면 `크기`가 반영될 가능성이 있다.
	- `등급 A, B, C` 처럼 연속성이나 크기 속성이 있다면 크기가 반영되어야 하지만,
	- `남자, 여자` 처럼 **순위, 연속성 모두 없다면 다른 의미의 수치값으로 변경**해야 한다.
- 따라서 별도의 칼럼으로 구분해서 각 칼럼에 대해 0과 1을 대입할 필요가 있음. 이를 `벡터화`라고 함.

```python
from sklearn.preprocessing import OneHotEncoder

ohe = OneHotEncoder()
onehot = ohe.fit_transform(df[['col']])

# 위 결과를 원래의 데이터프레임에 반영하려면 아래를 따르면 된다
df = df.drop('col', axis = 1)
df = np.c_[onehot.toarray(), df]

# 아니면 df.insert()를 쓰는 방법이 있을지도?
```
[[np.r_, np_c_ - 배열 합치기]]

### 근데 이 과정은 pandas에서도 할 수 있다.
[[pd.get_dummies]]

### 다만 OneHotEncoder의 장점은, fit과 transform을 분리할 수 있다는 데에 있음.
