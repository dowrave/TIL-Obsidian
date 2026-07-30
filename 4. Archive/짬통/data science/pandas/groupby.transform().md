
### 집계함수 적용한 열 추가하기(`transform`)
```python
# 목표 ) 각 기수 별로 출전한 경기 수를 열로 만들고 싶음

# 이거 써라
test = start_df.copy()
test['jockey_appear'] = test.groupby('jockey')['race_date'].transform('count')
```
- `transform`을 쓰지 않으면 `groupby -> merge -> 겹치는 열 정리`의 돌아가는 작업을 거쳐야 한다.
- **즉 `SQL`의 `OVER PARTITION BY` 작업은 `Pandas`에서 `Groupby()[].transform()`으로 이뤄진다는 것이다.**  

### transform을 쓰지 못하는 케이스
```python
# 목표 : 모든 jockey 옆에 dirt_apperance & turf_apperance가 붙는 것
jockey_df['dirt_appearance'] = jockey_df[jockey_df['course_type'] == 'D'].groupby('jockey')['race_id'].transform('count')
jockey_df['turf_appearance'] = jockey_df[jockey_df['course_type'].isin(['T', 'I', 'O'])].groupby('jockey')['race_id'].transform('count')
```
- `transform`을 적용한 열이 붙는 조건은 앞의 `course_type`부터 시작된다. 
	- `dirt_appearance` 열은 `course_type == D & jockey`에만 붙고
	- `turf_appearance` 열은 `course_type == T or I or O & jockey`에만 붙는다.

- 해결법) 위에서 `groupby`를 한 테이블을 따로 뺀 다음 `merge`하는 방법을 썼음
	- 