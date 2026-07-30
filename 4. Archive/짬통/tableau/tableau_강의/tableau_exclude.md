> 예제 : Category, Sub-Category, 평균(Sales)
> 여기서 특성 `{EXCLUDE [Sub-Category] : AVG([Sales])`을 추가함

- 이 때 일어나는 일
1. `Sub-Category`를 제외한 시점에서 `평균(Sales)`이 계산됨
2. 이렇게 계산된 결과값을 Sub-Category 수준으로 나타내기 위해, 각 Sub-Category에 복붙되어 넣어짐.

## Exclude
1. **Exclude LOD에서 선언된 차원이 반드시 VLOD에서 명시되어야 함**
2. 1차적으로 **해당 차원을 제외하고 집계**가 이뤄짐
3. VLOD에 맞춰 표현하기 위해 **2번째 단계 결과를 복제함**

- 따라서 Exclude에서 만들어진 집계 결과는 반드시 VLOD보다얕을 수밖에 없다.

---
### 예제 
- 각 도시의 소속 주에 대한 수익 기여도를 지도에 표시하시오
- `원 : 각 도시`
- `색상 : 수익 기여도`
- `크기 : 수익 또는 손실 규모`

#### 풀이
1. `State, City`를 넣음
2. 특성을 만듦
``` tableau
{EXCLUDE [City] : SUM([Profit])}
```
3. `특성`과 `Profit`을 넣음
4. 특성을 하나 더 만듦
```tableau
SUM([Profit]) / SUM([2.에서 만든 특성])
```
- `2에서 만든 특성`은 `State`단위므로 `City` 단위인 `Profit`보다 크기 때문
5. 문제 요구 사항에 따라 `지도`로 바꾸고, 도시(세부 정보 : `City`), 색상(`4.에서 만든 특성`), 크기(`Profit`)를 넣음
	- 만약 지도가 안 뜨면 국가를 미국으로 바꿔줘야 함

- 시각화를 했는데 명료해보이지 않긴 함 

#### 해답
1. City 로 지도를 만듦

2. `색상` 만들기 : 각 도시가 주에 기여하는 `Profit`의 비율-> 특성을 만들어줘야 함
```tableau
SUM([Profit]) / {EXCLUDE [City] : SUM([Profit])}
```
- 이 상태는 에러가 발생함
	- `SUM`은 현재 LOD에서 집계된 값
	- `EXCLUDE` 문은 항상 그 결과가 `Row Level`의 것이다 : 따라서 집계를 1번 더 해줘야 함 - 여기서 이미 집계를 했더라도!
		- 쉽게 생각하면 `{}`으로 단독으로 만들었더라도, 그래프에는 1번 더 집계돼서 들어가는 걸 생각하면 됨
	- 이 집계를 1번 더 하는 과정은 특성을 새로 만들어도 되지만 **단순히 `{}`밖에 집계함수를 적용하는 것도 가능**함
```tableau
SUM([Profit]) / SUM({EXCLUDE [City] : SUM([Profit])})
// 도시의 합계 / 주의 합계
```

3. `크기` 만들기 : `Profit`을 적용하면 됨
	- 근데 음수여도 **절댓값이 크면 차트에 보여지는 크기를 키우고 싶음(디폴트 : 작은 값은 크기가 작다)**
	- `마크 카드 - 크기의 SUM(Profit)`를 `ABS(SUM(Profit))`으로 바꿔주면 됨 : 절댓값

## Exclude LOD는 언제 쓰나요?
1. 차원 A에 대한 하부 차원 B의 기여도를 정규화할 때(예제)
2. 특정 차원의 한 값을 동일한 차원의 다른 값과 상대 비교할 때

#### 상대 비교 예제
> 1. Sub-Category와 합계(Sales)를 띄운다
> 2. `Paper`와 나머지를 상대적으로 비교하고 싶다. -> `Paper의 Sales` 특성을 만든다
```tableau
IIF([Sub-Category] = 'Paper', [Sales], NULL)
```
> 3. `Sales Paper` 특성을 넣는다
> 4. 전체 필드에 대해 `Sub-Category`의 Paper 값을 넣는다 : 특성을 새로 만들어야 함
```tableau
{ EXCLUDE [Sub-Category] : SUM(`Sales Paper`)}
```
- 차원이 Sub-Category 1개밖에 없으니까 모든 상황에서 `Sum('Sales-Paper')`의 값이 들어가게 되며, VLOD에선 `Sub-Category`가 포함되니까 모든 `Sub-Category`에 대해 Paper의 `Sum(Sales)`가 들어가게 됨

> 5. 마지막으로, 특성 1개를 또 만들어준다.
```tableau
SUM([Sales]) = ATTR([`4.` 특성])
```

- 이러면 Paper의 Sales 값을 기준으로 각 서브 카테고리가 얼마나 높거나 낮은지 나타낼 수 있음


---
## INCLUDE와 EXCLUDE는 활용도가 높지 않다
- 대부분의 경우 `FIXED`로 대체 가능하고, `FIXED`가 훨씬 쉽다.

### 1. FIXED가 더 유연함
- VLOD를 둔 다음에 고려해야 하는게 `EXCLUDE, INCLUDE`임
- 반면 `FIXED`는 현재 VLOD를 고려할 필요 없이 선언한 차원 내에서만 데이터를 집계하므로 더 유연하다.

### 2. 결과 종류의 유연성
- `INCLUDE, EXCLUDE`는 그 결과값으로 측정값만 갖게 됨
- **`FIXED`는 측정값이 나올 수도, 차원이 나올 수도 있다.**
	- EX) `날짜`

### 3. 작동 순서
- `INCLUDE, EXCLUDE`는 차원 필터 이후에 계산이 일어나게 됨
	- 항상 차원 필터의 영향을 받음
- `FIXED`는 차원 필터에 의한 영향을 받지 않음
	- 컨텍스트 필터를 수정하면 `FIXED`에 영향을 줄 수도 있긴 하다.