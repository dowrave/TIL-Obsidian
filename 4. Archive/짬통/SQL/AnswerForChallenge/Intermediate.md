# Intermediate Challenge 
## 문제 1번
- 답 : MidWest - 9개로 가장 적음
```SQL
SELECT s.region_id,
       r.name,
       COUNT(*) as counts
  FROM sqlchallenge1.sales_reps s
  JOIN sqlchallenge1.region r
    ON s.region_id = r.id
 GROUP BY s.region_id, r.name -- 같은 조건이긴 하지만 더 명료하다
```

## 문제 2번 : 주어진 이름 3명 중 어떤 사람이 가장 2016년 9월에 가장 많은 주문을 한 계정을 갖고 있는가? 
- 답 :  account_id 1871 - sum 1679
```SQL
-- 3개의 테이블 정보를 사용해야 함 : Multiple JOIN을 알아야 할 거 같은데?
SELECT s.name,
       o.account_id,
       SUM(o.total) AS sum_total_for_account_id
  FROM sqlchallenge1.sales_reps s
    JOIN sqlchallenge1.accounts a 
      ON s.id = a.sales_rep_id
     AND s.name IN ('Tia Amato', 'Delilah Krum', 'Soraya Fulton')
    JOIN sqlchallenge1.orders o 
      ON a.id = o.account_id 
     AND o.occurred_at BETWEEN DATE('2016-09-01') AND DATE('2016-09-30') + 1
 GROUP BY s.name, o.account_id
 ORDER BY 3 DESC 


-- account_id - 2441 : total_for_account_id - 5552
-- 여기서 조회되는 account_id 1871도 sum 1679로 나옴
-- 이거 맞냐
-- SELECT account_id, 
--       SUM(total) as total_for_account_id
--   FROM sqlchallenge1.orders
-- WHERE occurred_at BETWEEN DATE('2016-09-01') AND DATE('2016-09-30') + 1
-- GROUP BY account_id
-- ORDER BY 2 DESC
```
-------------------------
## 문제 3번 : NE의 판매원에 의한 계정 중, 포스터를 사지 않은 계정이 있다. 어떤 회사인가?
- 답 : Exxon Mobil (아래 쿼리로 깔끔하게 나옴)
```SQL
SELECT DISTINCT a.name,
      o.poster_qty,
      r.name AS region_name
  FROM sqlchallenge1.region r-- NE는 id = 1이다.
    JOIN sqlchallenge1.sales_reps s 
      ON s.region_id = r.id 
    AND r.id = 1 
    JOIN sqlchallenge1.accounts a
      ON s.id = a.sales_rep_id 
    JOIN sqlchallenge1.orders o 
      ON o.account_id = a.id
    AND o.poster_qty = 0
WHERE a.name IN ('Under Armour', 
                  'Universal Health Services',
                  'Exxon Mobil',
                  'Goldman Sachs Group')
```                 
-------------------------------------
## 문제 4번 : 포스터를 1번도 주문하지 않은 account 수
 
- 답 : 난 2개가 조회되는데 정답에는 없다? 그래서 3개로 체크함 
```SQL
SELECT a.id,
      SUM(o.poster_qty) AS ordered_sum_poster,
      SUM(o.poster_amt_usd) AS used_sum_poster
  FROM sqlchallenge1.accounts a 
  JOIN sqlchallenge1.orders o 
    ON o.account_id = a.id 
GROUP BY a.id
ORDER BY 2
-- account_id 별로 poster_qty를 집계한 다음에 COUNT가 들어가야 함.
/*
SELECT *
  FROM sqlchallenge1.orders
*/
```

------------------------
## 문제 5번 : primary pocs의 가장 흔한 first name은?
- 답 : Jodee가 4개로 젤 많음
```SQL
SELECT primary_poc
  FROM sqlchallenge1.accounts
 WHERE primary_poc ILIKE 'huong%'
    OR primary_poc ILIKE 'jodee%'
    OR primary_poc ILIKE 'chadwick%'
    OR primary_poc ILIKE 'domonique%'
 ORDER BY primary_poc
 ```
- 원래라면 문자열을 분리하는 함수를 이용해서 first name만 뽑아낸 다음 agg function 적용하는 게 맞겠지만
- SUBSTRING이나 LEFT, CHARINDEX 등의 함수가 다 안먹힌다;
