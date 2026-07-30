## 1. 선언한 차원이 VLOD에 포함될 때

### 1. 선언한 차원이 VLOD와 같을 때
> 예제 : Category, Sub-Category, avg(sales)

- 이 상황에서 아래 특성을 넣어본다
```tableau
{ FIXED [Sub-Category] : AVG([Sales])}
```
- VLOD가 `Sub-Category`이므로 동일함
- 이 때는 **보고 있는 화면과 동일하게 나옴**
	- 특이한 건, 위 특성은 디폴트로 `합계`로 들어가는데도 그래프가 동일하다.
	- (내 생각) 각 서브 카테고리에 1개의 통계값(`avg`)밖에 없으니까 그런 거 아님?


### 2. 선언한 차원이 VLOD보다 깊을 때

> 예제) Category, Sub-Category, ManuFacturer, avg(sales)
- `1.`의 예제보다 더 깊은 VLOD를 가진 상황
> 나는 Manufacturer가 없어서 Segment를 넣었다

- 위에서 만든 Fixed LOD를 그대로 넣으면?
- `Category - Sub-Category - Manufacturer` 중, `Sub-Category`가 같은 것끼리 같은 값으로 나옴
- 이는 Exclude의 동작과 동일함 : 위에서 값을 구하고, 값이 복사되어 더 깊은 차원에 대해 표시됨

## 2. 선언한 차원이 VLOD에 없을 때

### 1. 선언한 차원이 VLOD보다 얕을 때

> 예제 ) 위에서 만든 Fixed 차원(by Sub-Category) - SUM으로 변경, Sales 합계, Category

- `Fixed`와 `Sales`가 완전히 동일한 결과를 냄

#### 계산이 이뤄지는 방식
- 왜 `Sub-Category` 수준에서 합치라고 했는데 `Category` 수준에서도 동일한 결과가 나오는가?

- **`INCLUDE`와 동일**함
	1. Sub-Category에서 매출을 합계
	2. VLOD는 Sub-Category보다 얕은 수준
	3. 따라서 **`1.`에서 집계된 매출이 한 번 더 집계**됨
	4. 그 집계 방식은 `합계`

- 만약 `평균`으로 계산했을 경우를 상상해보자
> `Category`, `AVG(Sales)`, `Fixed Sub-Category avg (sales)`

- Fixed의 계산은 이렇게 이뤄지겠죠?
	1. Sub-Category의 평균을 냄
	2. 이걸 VLOD : Category 레벨에 맞춰서 값을 내야 함. 이 특성에 적용된 집계 함수를 따라 적용됨
	3. 그러나 이 특성을 다시 `avg()`한다고 해도, `AVG(Sales)`와 달라질 수 있음 : 샘플의 수가 다를 수 있기 때문에.

### 2. 선언한 차원이 VLOD와 아예 관계가 없을 때

> 예제 : State, City, AVG(Sales), {Fixed [Sub-Category] : AVG(Sales)}

- `Fixed`에서 선언된 `Sub-Category`는 VLOD의 차원인 `State, City`와 아예 따로 노는 차원임

#### 계산 방식
1. `Sub-Category`에서 1차적인 계산이 일어남
2. **태블로는 현재의 VLOD 레벨`City`에서 `Sub-Category` 차원을 넣어본 뒤, 측정값이 존재하는지 아닌지를 확인함**
	- 예제에서는 `City`마다 `Sub-Category`가 얼마나 있는지가 다름 (어떤 도시는 6개, 어떤 도시는 11개, ...)
3. VLOD 레벨인 `City`가 갖고 있는 **`Sub-Category`에 대해서만 2차 계산이 수행됨**

- 이 계산 방식은 위의 `INCLUDE`에도 적용할 수 있는 얘기임!

## 연습 
> 각 지역(Region)별로 수익을 낸 주문과 손실을 낸 주문의 매출 비중을 구하시오

1. `Region`은 들어가야 함
2. 수익을 낸 주문과 손실을 낸 주문의 비중?
	- `sum(profit)`을 넣으면 그 자체로 전체 수익이 됨

#### 모르겠다 GG -> 풀이

1. 각 지역 별로 매출 보기
> 1. `합계(Sales)`와 `Region`을 올림
> 2. `합계(Sales) - 퀵 테이블 계산 - 구성 비율`
> 3. `합계(Sales) - 다음을 사용하여 계산 - 테이블(옆으로)`
- 각 지역이 100%인 바 차트가 만들어짐

2. 수익을 낸 주문과 손실을 낸 주문의 매출 비중 보기
- 주문 별로 수익을 냈는지를 확인해야 함
- 여기서 Fixed LOD를 쓰면 됨
```tableau
{ FIXED [Order ID] : SUM([Profit]) > 0}
```
- 이렇게 작성하면 `True/False` 필드가 하나 생김
- 이걸 `색상`에 드래그앤 드랍

- VLOD는 `Region` 차원이지만, 이익 / 손실에 대한 차원은 `Fixed`의 `Order ID`에서 이뤄지고 있음
- 이 `Order ID` 차원과 `Region` 차원은 전혀 관계 없는 차원이지만, 저렇게 만든 특성을 `색상`에 올리면 주어진 VLOD에 맞춰서 화면이 나오게 됨
- 깊어진다는 것보다는 교집합을 만들었다고 봐야 하나?
- 수익을 낸 고객과 아닌 고객으로 바꿀 수도 있겠죠?

## 3. Fixed LOD의 사용법

1. **전제 데이터셋에서 집계값**을 잡을 때
> 예시) Sales 특성을 올리고, Category, SubCategory를 올리면 SubCategory에 따라 Sales가 나뉘게 될 거임
> 여기서 새로운 특성을 하나 만듦
```tableau
{ SUM(Sales) } // fixed를 쓸 필요도 없음
```
- 전체에 대해 만들어진 `Sum(sales)`는 차원이 아무리 올라가더라도 데이터셋 전체의 Sales의 합계만 나타내게 됨 (차원이 많아지면 저 값이 그냥 복사가 되는 거임)

2. **날짜 필드 활용할 떄**
- 올 해, 지난 주 등의 개념을 만들 수 있음

- 올 해
```tableau
{MAX(YEAR([Order Date]))}
```
- `측정값`이 아니라 `차원`으로 올리면 가장 최근 년도가 나옴
- 작년 : 위 값에서 1 빼면 됨

- 이게 좋은 이유 : 연도가 바뀌더라도 자동으로 가장 최근 년도를 구해줄 수 있음

3. **필터의 영향을 받지 않는 값을 만들어야 할 때**
- 차원 필터보다 빨리 오기 때문
- 그렇다고 `Fixed`가 영향을 받지 않는 건 아니고, 컨텍스트 필터에 차원 필드가 올라가면 영향을 받게 됨