1. `df.str.extract('()')
	- 패턴(정규표현식 가능)과 일치하는 값만 남기고 나머지는 결측치 처리해서 반환함
	- True / False를 반환하지 않는다!
2. `df.str.contains()`
	- `contains` 내부 조건에 따라 `T/F`를 반환한다.
3. `df['column'].str.replace(A, B)`
	- A를 B로 바꾼다.