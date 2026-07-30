[SQL vs Pandas](https://www.inflearn.com/questions/540838)

#### SQL이 편한 경우
1. 많은 테이블 JOIN
2. 여러 Column과 다양한 Aggregation을 적용한 Groupby를 많이 수행할 경우
3. 복잡한 Where 조건의 필터링
4. 집합 레벨 변경이 더 수월
5. 복잡한 데이터 가공은 SQL이 더 직관적인 코드 작성 가능
	- 개인 성향에 따라 의견이 다를 수 있다.

#### Pandas가 편한 경우
1. 특정 Column을 가공해 새로운 Column을 만들거나 수정하거나 타입을 변경할 상황이 빈번하게 많은 경우 (`수정이 잦은 경우?`)
2. 분산, 표준편차, z-score 등 다양한 통계 기반의 가공이 필요한 경우
	- Pandas 자체 통계 함수 외에도 `numpy, scipy, sklearn` 등과 호환이 훌륭함
3. ML, 시각화 패키지와 바로 호환 가능
4. 빠르다.
5. 반복문을 돌려 데이터 가공이나 조회를 할 수 있다. 

