> IF 함수는 시각화에서 쓰이지 않는 경우가 없음! 여기선 기초적인 수준만 다룸

## 1. 계산된 필드로 조건 부여하기(이진)
```tableau
IF SUM([Sales]) >= 50000 THEN "HIGH" ELSE "LOW" END
IF SUM([Sales]) >= 50000 THEN "HIGH" END
IIF (SUM([Sales]) >= 50000, 'HIGH', 'LOW')
SUM([Sales]) >= 50000
```

- 방법 1. 50000 이상은 HIGH, 미만은 LOW
- 방법 2. 50000 이상은 HIGH, 미만은 Null
- 방법 3. 방법 1.과 동일하되 코드 길이를 줄일 수 있음
- 방법 4. 50000 이상은 True, 미만은 False **<-- 성능적인 측면에서 제일 우월함**
	1. T/F가 문자열보다 빠르게 작동함
	2. 코드의 길이를 줄일 수 있음
	- 이진 분류일 경우, 이 방법이 베스트임

---

- IF 함수 : `IF -  THEN - (ELSE -)  END`
- IIF 함수 : `IIF(조건식, True값, False값)`

- 참고 ) **조건식에는 이미 설정된 집합이나 T/F필드**가 올 수 있다

---

## 2. 다중 분류 조건 부여

```TABLEAU
IF SUM([Sales]) >= 50000 THEN "HIGH" 
ELSEIF SUM([Sales]) >= 25000 THEN "MID" 
ELSE "LOW" END
```
- `ELSEIF`를 이용하며, `ELSEIF`는 원하는 만큼 넣을 수 있다.

