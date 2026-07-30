- 한 테이블이 다른 테이블의 밑에 가도록 이어 붙임

## pd.concat()을 반복하는 건 비효율적
- 왜? : 추가적인 공간 확보 -> 원래 데이터들을 복사 붙여넣기 하는 과정이 `pd.concat()`인데, 이를 계속 반복하기 때
- 이 경우 각 테이블을 데이터프레임으로 만들어 리스트에 넣은 뒤, 테이블을 다 넣었으면 `pd.concat(lst)`을 하면 됨
```python
import pandas as pd

lst = []

for i in range(N):
	df = to_frame(table(i)) # 임의의 데이터프레임을 만듦
	lst.append(df)
	
df = pd.concat(lst)
```

- 테이블을 dict로 구현한 뒤 lst에 append하고, 마지막에 한꺼번에 데이터프레임으로 바꾸는 방법도 있는데 위 방법이 더 편해보임
```python
rows = []

for i in range(1, 1000):
    # Instead of generating a dataframe, generate a dictionary
    dictionary = generate_dictionary()
    rows.append(dictionary)

combined = pd.DataFrame(rows)
```
