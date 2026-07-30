- **문자를 숫자들로 인코딩**함 
```python
from sklearn.preprocessing import LabelEncoder

le = LabelEncoder()
le.fit() # 리스트, 시리즈 등등

le.transform() # 여기에 전달된 값이 fit 과정에 있었다면 해당하는 숫자로 반환

le.inverse_transform() # 숫자를 전달하면 해당하는 종류로 반환

le.fit_transform() # fit + transform 한번에 진행
```

- 여러 column에 따로 적용하고 싶다면
```python
df[['col1', 'col2', ...]].apply(LabelEncoder().fit_transform())
```

## 범주 숫자 순서 바꾸기
- 참고 : `LabelEncoder()` 자체는 `np.unique()`에서 불러온 값 순서대로 저장됨.

```python
le = LabelEncoder()
le.fit(df['col'])

# fit 결과 classes_ 속성이 생긴다. 이를 바꿀 수 있음
le.classes_ = np.array(['']) # 원하는 순서대로 집어넣음

df['col'] = le.transform(df['col'])
```

```python
# 위 과정을 함수로 만들 수도 있음
def column_label_order(df, col, order_lst):
  LE = LabelEncoder()
  LE.fit(df[col])
  LE.classes_ = np.array(order_lst)
  df[col] = LE.transform(df[col])
```
- 여기서 중요한 건 **데이터프레임과 column을 따로 불러야 한다**는 거임
	- 만약 `series` 로 간다면, 그 시리즈만을 수정하는 것이 됨
	- 반면 `df[col]` 형태는 해당 데이터프레임의 열을 수정하는 것이 된다