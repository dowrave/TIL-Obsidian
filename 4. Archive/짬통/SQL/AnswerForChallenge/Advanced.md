## 1. 2015년 동안 west 지역에서 어떤 달의 포스터 주문율이 가장 높았는가?
```sql
SELECT EXTRACT('year' FROM o.occurred_at) as year,
       EXTRACT('month' FROM o.occurred_at) as month,
       COUNT(poster_qty) as monthly_poster
  FROM sqlchallenge1.orders o
  JOIN (
        SELECT a.id, a.name, sub.sales_rep_id, sub.region_name
          FROM sqlchallenge1.accounts a
          JOIN (
                SELECT s.id AS sales_rep_id, 
                      r.name AS region_name
                FROM sqlchallenge1.sales_reps s
                JOIN sqlchallenge1.region r
                  ON s.region_id = r.id 
                AND r.name = 'West'
                ) sub 
            ON a.sales_rep_id = sub.sales_rep_id
        ) final
    ON final.id = o.account_id
 WHERE o.occurred_at >= DATE('2015-01-01') 
   AND o.occurred_at < DATE('2016-01-01')
 GROUP BY 1, 2
 ORDER BY 3 DESC
```
- 퍼센트로 쓰려면 서브 쿼리로 한 번 더 빼면 될 것 같음

## 2. 1번째 주문부터 주문을 받았을 때, 어떤 판매원이 총 판매액 10만 달러에 가장 먼저 도달했는가?
```sql
SELECT sub.id, sub.this_sold, sub.total_sold, sub.time
  FROM (
        SELECT a.sales_rep_id AS id,
               o.total_amt_usd AS this_sold,
               SUM(o.total_amt_usd) OVER 
               (PARTITION BY a.sales_rep_id ORDER BY o.occurred_at)
               AS total_sold,
               o.occurred_at AS time
          FROM sqlchallenge1.orders o 
          JOIN sqlchallenge1.accounts a
            ON o.account_id = a.id 
        ) sub
WHERE sub.total_sold >= 100000
ORDER BY sub.time
```
- 쿼리 결과는 `321640`이 `2014-04-06`에 가장 먼저 도달했는데 보기에 없다.
  - 보기(`321900`, `321700`, `321650`, `321680`) 중에서는 `321680`이 `2014-10-05`에 제일 먼저 도달함
  - `321700` : `2015-10-15`
  - `321650` : `2016-07-08`
  - `321900` : `2016-08-05`

## 3. 2건 이상 판매한 판매원 중 1번째 주문과 2번째 주문의 텀이 가장 긴 직원의 이름?
```sql

SELECT final_sub.name,
       final_sub.time, 
       final_sub.next_time,
       final_sub.next_time - final_sub.time as difference
FROM( 
       SELECT sub2.name,
             sub2.time,
             LEAD(time, 1) OVER (PARTITION BY sub2.name ORDER BY sub2.time) as next_time
        FROM (
              SELECT sub.sales_reps_name AS name,
                     COUNT(o.occurred_at) OVER
                     (PARTITION BY sub.sales_reps_name ORDER BY o.occurred_at)
                     AS counts,
                     o.occurred_at AS time
                FROM sqlchallenge1.orders o
                JOIN (
                      SELECT a.id as account_id,
                             s.name as sales_reps_name
                        FROM sqlchallenge1.accounts a 
                        JOIN sqlchallenge1.sales_reps s
                          ON a.sales_rep_id = s.id 
                      ) sub
                   ON o.account_id = sub.account_id 
           ) sub2
       WHERE sub2.counts <= 2
         AND sub2.name IN ('Julia Behrman', 'Samuel Racine', 'Eugena Esser', 'Nelle Meaux')
    ) final_sub
WHERE final_sub.next_time IS NOT NULL

```
- `LAG`, `LEAD` 쓰는 문제 + 서브 쿼리 문제
- `Timestamp` 자료형 간 연산은 그냥 자연스럽게 할 수 있음 : 별도의 함수(`INTERVAL`)가 필요 없다.
- 집계값을 새로운 변수로 쓰기 위해 서브 쿼리가 들어가는 데 그 빈도가 무려 3번이다. 이거 맞냐? 개선 여지가 있을 것 같은데
`Eugena Esser - 30일 6시간 25분 2초 / Julia Behrman - 28일 10시간 17분 49초 / Nelle Meaux - 10일 22시간 40분 49초 / Samuel Racine - 2분 50초`

## 4. 1만 달러 이상 팔기 전에 최소 9개 이상의 주문을 받은 판매원의 수?
```SQL
SELECT *
        -- COUNT(*)
  FROM (
  SELECT sub.sales_rep_id,
         COUNT(o.occurred_at) OVER
         (PARTITION BY sub.sales_rep_id ORDER BY occurred_at)
         AS counts,
         SUM(total_amt_usd) OVER
         (PARTITION BY sub.sales_rep_id ORDER BY occurred_at)
         AS usd_dollars
    FROM sqlchallenge1.orders o 
    JOIN (
          SELECT a.id as account_id,
                 s.id as sales_rep_id
            FROM sqlchallenge1.sales_reps s 
            JOIN sqlchallenge1.accounts a 
              ON s.id = a.sales_rep_id
          ) sub
      ON o.account_id = sub.account_id
      ) sub2
 WHERE sub2.counts >= 9 
   AND sub2.usd_dollars < 10000
```
- 위에서 헤매니까 의외로 쉬워짐

## 5. 주어진 기간 중 이틀(two-calender-day) 동안의 구매액이 가장 많은 구간은?
```SQL
SELECT sub.group,
       SUM(sub.total_amt_usd)
  FROM (
        SELECT occurred_at, 
               CASE WHEN occurred_at >= DATE('2016-10-26') AND occurred_at < DATE('2016-10-28') THEN '2016-10-26'
                    WHEN occurred_at >=  DATE('2016-12-20') AND occurred_at < DATE('2016-12-22') THEN '2016-12-20'
                    WHEN occurred_at >=  DATE('2016-12-26') AND occurred_at < DATE('2016-12-28') THEN '2016-12-26'
                    ELSE NULL END AS group,
               total_amt_usd,
               SUM(total_amt_usd) OVER
               (ORDER BY occurred_at)
               as acc_sum
          FROM sqlchallenge1.orders
        ) sub
 WHERE sub.group IS NOT NULL
 GROUP BY 1
 ORDER BY 2 DESC
```
- `2016-12-26 : 338567.67 / 2016-12-20 : 183276.29 / 2016-10-26 : 150110.43`
- 왜 뒤로 갈수록 쉽고 앞이 어렵냐