# 기본 쿼리
#### 이미 아는 내용이라면 중복해서 설명하지 않고 쿼리문만 적어놓겠음

```SQL
SELECT *
  FROM (table_name)
```

- 쿼리로 얻은 테이블은 DB에 아무 영향을 주지 않는다. 정확히는 `SELECT` 쿼리는 테이블에 영향을 주지 않는다는 것

- `SELECT`나 `FROM` 등의 쿼리문을 대문자로 쓰지 않아도 SQL은 이해하지만, 직관성 때문에 사용함.
  - 한 줄로 쓰든 줄을 나누든 코드의 실행 자체에는 영향을 주지 않는다.

### 열 이름 바꿔서 가져오기
```SQL
SELECT west AS "West_Region",
       south As "South_Region"
  FROM (table_name)
```
- `AS`를 쓰면 변수 이름을 다른 변수로 취급할 수 있음
  - 나중에 테이블 `JOIN` 같은 상황에서도 자주 쓰니까 알아두자.

### LIMIT
```SQL
SELECT *
  FROM (table_name)
 LIMIT 100
```
- 테이블이 어떻게 생겼나 보는 게 목적인데, 모든 데이터를 조회하는 것보다 **일부만 조회하는 게 훨씬 빠름**
  - 이는 테이블의 데이터가 많을수록 더 유용함 : 10억개의 row가 있다고 생각해보자.

### WHERE
```SQL
SELECT *
  FROM (table_name)
 WHERE (column) = (value)
```
- WHERE 조건 : 어떤 Column의 값이 value인 row만을 불러옴.
  - 엑셀도 한 열에 대해 재정렬할 수 있는데 이는 다른 열들의 재정렬이 이뤄지지 않아 데이터를 망가뜨릴 수 있음
  - 한편 SQL은 데이터의 전체 열이 보존됨

#### 비교 연산자
`=`, `<> or !=`, `>`, `<`, `>=`, `<=`

##### 수치형 데이터가 아닐 경우
```SQL
SELECT *
  FROM (table_name)
 WHERE (non-numerical_column) > 'J'
```
- 알파벳 'j'로 시작하는 데이터부터만을 나열함
- 왜 `=`를 넣지 않아도 되는가? : 알파벳 'J'만이 들어가는 데이터라면 조건에 걸리지 않을 것이다. 그러나 **'Ja'같이 뒤에 글자가 추가로 있다면 'J'보다 큰 것으로 인식**함 
- 아스키 코드 숫자인가 해서 'j'로도 돌려봤는데, 똑같이 'J'를 잡아냈다. **대문자 소문자 얘기는 아니**라는 뜻.

### Column을 이용해 조회할 때 새로운 열을 만들 수도 있다.
```SQL
SELECT A, B, A + B AS NEW_COLUMN_NAME
  FROM (table_name)
```
- 새로운 COLUMN 이름에는 `''`가 들어가지 않는다.

### 논리 연산자
- `LIKE` : 정확한 값 대신 비슷한 값을 찾음
- `IN` : 포함하고 싶은 값을 특정할 수 있음
- `BETWEEN` : 특정 범위의 ROW만을 특정
- `IS NULL` : 데이터가 없는 ROW 특정
- `AND` : 2개의 조건을 만족하는 ROW 특정
- `OR` : 1개의 조건 이상을 만족하는 ROW 특정
- `NOT` : 조건을 만족하지 않는 ROW 특정
-----------
#### LIKE & ILIKE - 정규식 역할
```SQL
SELECT * 
  FROM tutorial.billboard_top_100_year_end
 WHERE "group" LIKE 'Snoop%'
```
1. `"group"` : `GROUP`은 SQL에 있는 함수이기 때문에 큰 따옴표로 감싸준 것
    - 그외에 **큰 따옴표를 쓴다면, Column 이름을 참조한다**는 뜻
    - 이는 작은 따옴표와 구분됨 : **작은 따옴표는 Column을 참조하는 게 아니라 값을 참조한다고 일단은 생각**해두자.
2. `%` : 어떤 Character든 **0개 이상** 얻는다는 뜻. 그래서 `Wildcard`라고 부름.
3. `LIKE`는 정확히 해당 문자만 와야 함 : 즉 대문자 S가 소문자 s로 쓰였다면 매칭하지 않음.
   - 이 경우 `ILIKE`를 쓸 수 있다. `ILIKE Snoop%`으로 써도 소문자 s 또한 매칭해줌.
   - `_(underscore)` : 아무 1글자를 매칭해줌

```SQL
-- ILIKE & _ 예문
SELECT * 
  FROM tutorial.billboard_top_100_year_end
 WHERE artist ILIKE 'dr_ke'
```

##### M.C. Hammer 예제
- 활동명이 `Hammer`와 `M.C. Hammer` 2가지임
- 이들을 탐색하려면 `%Hammer%`로 검색하면 됨 
  - `Hammer` 자체라는 글자 자체부터 포함하기 때문이다.

#### IN
```SQL
SELECT * 
  FROM tutorial.billboard_top_100_year_end
 WHERE year_rank IN (1, 2, 3)
```
1. `IN` 뒤에는 문자열도 올 수 있음. `('Taylor Swift', 'Drake')` 같이.
2. `Python`의 `in`이랑 용법이 좀 다르다!

#### BETWEEN
```SQL
SELECT *
  FROM tutorial.billboard_top_100_year_end
 WHERE year_rank BETWEEN 5 AND 10
 -- WHERE year_rank >= 5 AND year_rank <= 10 
```
- 주석 처리한 부분과 동일함. 아래 코드가 더 무엇을 수행하는지 직관적으로 볼 수 있다는 장점은 있다.
- `BETWEEN`은 경계 부분(`5, 10`)을 포함한다는 것만 알아두자.

#### IS NULL
```SQL
SELECT *
  FROM tutorial.billboard_top_100_year_end
 WHERE artist IS NULL
```
- 참고 : `WHERE artist = NULL`은 동작하지 않는다.


#### AND
```SQL
SELECT *
  FROM tutorial.billboard_top_100_year_end
 WHERE year = 2012
   AND year_rank <= 10
   AND "group" ILIKE '%feat%'
```
- `AND`를 여러 줄로 쓰는 게 읽을 때 더 직관적이다.

#### OR
```SQL
SELECT *
  FROM tutorial.billboard_top_100_year_end
 WHERE year = 2013
   AND ("group" ILIKE '%macklemore%' OR "group" ILIKE '%timberlake%')
```
- `AND`와 `OR` 동시에 쓰기 : 구분은 `()`로 지어주면 된다.
    - 2013년 + 그룹에 macklemore만 있거나, timberlake만 있거나, 둘 다 있을 때 매칭됨
- 주의) 이런 식의 사용은 안됨
```SQL
-- 이런 식으로 쓸 수 없음
WHERE year = 2013
  AND "group" ILIKE ( '%macklemore%' OR '%timberlake%' )
```
- `WHERE` 문에 해당하는 조건식은 
  - `(col_name) (operator) (conditions)`의 형태이다. 
  - 즉 하나의 operator를 쓰고자 한다면, 반드시(?) col_name이 따라붙는 형태라는 것이다. 

- 예제 ) `BETWEEN`까지 쓰는 경우
```SQL
SELECT *
  FROM tutorial.billboard_top_100_year_end
 WHERE song_name ILIKE '%california%'
   AND (year BETWEEN 1970 AND 1979 OR year BETWEEN 1990 AND 1999)
```
- 똑같다. `BETWEEN A AND B` 의 용법 때문에 헷갈릴 수 있는데, 이것 때문에 추가로 소괄호를 칠 필요는 없다. 


#### NOT
```SQL
SELECT *
  FROM tutorial.billboard_top_100_year_end
 WHERE year = 2013
   AND year_rank NOT BETWEEN 2 AND 3
```
- 비교 연산자에는 사용하지 못한다 : 실제 사용이 불가능하기도 하고, `NOT >` = `<=` 이잖음? 반대 연산이 어렵지 않기 때문임
- `NOT LIKE`형태로 쓰면 해당 매칭만을 제외할 수 있고, `IS NOT NULL`의 형태로 써서 결측치를 제거할 수도 있다. 
- 예시에서 보이듯, `NOT (operator)` 형식으로 들어간다. 

### ORDER BY
```SQL
SELECT *
  FROM tutorial.billboard_top_100_year_end
 WHERE year = 2013
 ORDER BY artist
```
- 기본 : `오름차순` - 숫자 1부터 시작해서 알파벳 a->z 순으로 배열됨
  - 내림차순을 원한다면, `artist DESC` 형태로 붙이자.
- 이전에도 배웠듯, SQL이 작동되는 순서는 FROM부터 아래로 쭉 훑은 뒤 마지막에 SELECT가 실행됨

#### 여러 Column 정렬하기
```SQL
SELECT *
  FROM tutorial.billboard_top_100_year_end
 WHERE year_rank <= 3
 ORDER BY year DESC, year_rank
```
1. `column year`을 내림차순으로 정렬함
2. `column year_rank`을 오름차순으로 정렬함
- 즉 **ORDER BY 뒤에 나오는 COLUMN들의 순서를 따라 실행됨**
- ORDER BY 뒤에 각 COLUMN의 순서만을 전달해 실행할 수도 있는데, 모든 SQL에서 가능한 내용은 아니므로 패스함

### SQL 주석
```SQL
-- 요것은 단일 주석이고
/* 요것은 
여러 줄
주석입니다.*/ 
```
- 모든 주석(Comment)이 그렇듯 코드 실행에 영향을 주지 않는다.
- 다른 툴처럼 `드래그 + CTRL + /`도 가능

### Challenge
(4번 문제) : BASIC에서 나오지 않은 내용들 다룸

1. 날짜 범위 지정하기
```SQL
WHERE (COLUMN) BETWEEN DATE('year1-month1-day1') AND DATE('year2-month2-day2') + 1
```
- A부터 B까지라면 `B + 1`을 해줘야 한다.
  - 뇌피셜이긴 한데, 원하는건 B일 23:59:59까지인데 아마 B일 00:00:00까지만 체크되기 때문이지 않을까? 

2. 다른 테이블의 정보 동시에 보기 : `JOIN`
```SQL
SELECT *
  FROM (table_a) a
  JOIN (table_b) b
    ON a.column1 = b.column2
 (이하 WHERE 어쩌구저쩌구)
```

- SQL Challenge가 제대로 작동 안하네 ㅎㅎ;