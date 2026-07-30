
### 1. column 데이터 int ->  string 변경 
```python
# race['post_time'] 자체는 int 데이터였음
race['post_time'] = race['post_time'].astype('object')

# 내부 데이터는 int인데 column 형태는 object가 유지된다.
type(race.loc[0, 'post_time']) # int
race.info() # race['post_time'] : object
```
- 해결법 : `.astype('string')`, `.astype(str)` 
```python
race['post_time'] = race['post_time'].astype('string')
```
- `column` 내부 자료가 `string`으로 바뀜 + `column` 자체도 `string`으로 바뀜(`object`가 아님)
-------------------

### 2. 서로 다른 `column`에 있는 값들을 하나의 `column`에 합치기
```python
# 목표 : 2020-01-01, 12, 50 (셋 모두 string)의 값들을 하나에 column에 합치기
# 형태 : 날짜 시간:분:00
race['race_dt'] = race[['race_date', 'hour', 'minute']].apply(
                        lambda x : f"{x[0]} {x[1]}:{x[2]}:00", 
                        axis = 1)
race['race_dt'] = pd.to_datetime(race['race_dt'])
```
- `.apply()`함수를 적용할 때 여러 `column`에 적용하고 싶다면 똑같이 `[[]]`로 묶어주면 된다.
--------------------
### 3. 결측치 찾기
```python
df.isnull() # 모든 칸에 대해 True / False 반환
df.isnull().sum() # 모든 열에 대한 결측치 수 체크
df['A'].isnull().sum() # 특정 열에 대한 결측치 수 체크 
```
----------------

------------------------------
#### 5. 2개 이상의 열에`.apply()` 적용하기
- 오늘만 2번 헤매서 정리
```python
# 목표 : 어떤 두 열을 합치고 싶음(str로)

test['race_id'] = test[['str_race_date', 'str_race_number']]\
						.apply(
								lambda x : x[0] + ":" + x[1], 
								axis = 1 # 이게 중요함
							)
```
- 잘 적용했는데 왜 안되지?라고 생각된다면 `.apply(axis = 1)`을 빼먹지 않았는지 체크해볼 것
-----------------------
### 6. 평균을 모은 값으로 평균 내기 vs 그냥 평균 내기
- 이런 느낌이라고 생각하면 된다.
`(1+2+3+4+5) / 3` = `(3+3+3+3+3) / 3`
-------------------------------
### 7. 상관관계 분석
[링크](https://ordo.tistory.com/100)

#### 1. 상관관계
- 데이터의 스케일이 다를 경우가 잦기 때문에, 상관관계 분석을 할 때는 **공분산을 표준화**시켜 사용한다.
```python
import numpy as np

# 1.  공분산 구하기
cov = ( np.sum(X*Y) - len(X)*np.mean(X)*np.mean(Y) ) / len(X) # 공식
cov = np.cov(X, Y)[0, 1] # 넘파이 제공

# 2. 표준화된 공분산 구하기
corr = cov / ( np.std(X) * np.std(Y) ) # 공식
corr = np.corrcoef(X, Y)[0, 1] # 넘파이 제공
```
- `[0,1]`은 `2`개의 데이터를 넣었기 때문에 공분산 행렬의 `[0,1]`값을 취하는 것 같음
- `0`에 가까울수록 상관관계가 없다
	- `-1`에 가까울수록 강한 음의 상관관계
	- `1`에 가까울수록 강한 양의 상관관계
	- 통상 `0.3 ~ 0.7`이면 뚜렷하다, `0.7~1.0`이면 강하다라고 하는 듯
		- (분야마다 다르다는 얘기도 예전에 들은 것 같음)

#### 2. 검정
- 상관계수 값 자체가 유의미한가를 검정할 수 있다.
- 그 중 하나가 `p-value`로, `scipy`에 있음 [[p-value]]
```python
import scipy.stats as stats

stats.pearsonr(X , Y) # (상관계수, p-value)
```
- `p-value`는 귀무가설(`상관관계가 없다`)에 대한 검정으로, `0.05` or `0.01`보다 작으면 귀무가설을 기각(= `상관관계가 있다`)한다.
- 표준화를 하는가? 가 궁금했는데, 아래 예시를 보면 표준화가 적용된다.

#### 내 프로젝트 속 예시
```python
print(np.cov(appear_rank_df['appearance'], appear_rank_df['appear_avg_ranking'])[0, 1]) # -151.5677240920477

print(np.corrcoef(appear_rank_df['appearance'], appear_rank_df['appear_avg_ranking'])[0, 1]) # -0.6253813287496485

print(stats.pearsonr(appear_rank_df['appearance'], appear_rank_df['appear_avg_ranking'])) # (-0.6253813287496486, 1.534715287419361e-07)
```
- 위의 링크에 따르면 음의 상관관계가 유의미하게 나타남을 알 수 있다.
-----------------------------
### 8. 여러 열에 `.sort_values()` 적용하기
```python
# 목표 : 여러 열을 정렬하는데, 오름차순 / 내림차순이 섞인 경우
df.sort_values(by = ['col1', 'col2'], ascending = [False, True])
```
- 정렬할 열들을 리스트로 전달하듯, **어떻게 정렬할지도 리스트로 전달**하면 된다.
----------------------

-------------------------------

### 10 . column 옮기기
```python
df.insert(0, 'new_col_name', df.pop('col_name'))
```
- `new_col_name`은 기존 이름과 같아도 상관 없음
----------------------------------
