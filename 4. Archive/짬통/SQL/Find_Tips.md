# SQL 부딪히면서 배우기

## 1. 여러 행에 걸쳐 중복된 값이 나타날 때, DISTINCT에 대해
```SQL
SELECT DISTINCT user_id, device

```
- 예를 들면 어떤 유저가 어떤 제품을 주문하려 하는 과정에서 로그가 기록된다고 하자.
- **DISTINCT는 모든 COLUMN에 대해 적용된다.**
  1. `(COL1), DISTINCT (COL2)` 식으로 사용할 수 없다는 게 있고,
  2. `DISTINCT COL1, COL2, COL3, ...`인 경우, 튜플로 `COL1, COL2, COL3, ...`을 생각할 수 있을 것이다. 이 때 `DISTINCT`는 **튜플이 완전히 똑같지 않다면 제거하지 않는다.**

## 2. WINDOW FUNCTION에 대해
```SQL
SELECT user_id,
      event_type,
      event_name,
      occurred_at,
      occurred_at - LAG(occurred_at,1) OVER (PARTITION BY user_id ORDER BY occurred_at) AS last_event,
      LEAD(occurred_at,1) OVER (PARTITION BY user_id ORDER BY occurred_at) - occurred_at AS next_event,
      ROW_NUMBER() OVER () AS id
 FROM tutorial.yammer_events e
WHERE e.event_type = 'engagement'
ORDER BY user_id,occurred_at
```
- 애초에 생각했던 게 위 상황에서 `LAG()`, `LEAD()`을 쓸 수 없다 였음
  - 왜냐면 `WHERE` 조건을 넣을 수 없기도 하고
  - `GROUP BY`로 `user_id`가 묶이지 않아서 못 쓴다고 생각했음.
- 근데 위처럼 해도 쿼리가 잘 작동함
- 왜냐면 `ORDER BY`에서 `user_id`, `occurred_at` 순서로 묶인 데이터들과 `PARTITION BY user_id ORDER BY occurred_at` 순서로 묶인 데이터들의 순서가 같기 때문임
- `GROUP BY`는 생각할 필요가 없나 봄!(일단은?)
- 추가) `WHERE`조건에는 `ENGAGEMENT` 및 회원가입과 관련된 분류가 하나 더 있는데, 이것도 `ORDER BY`에 의해 묶이는 걸 생각하면 상관 없다. 회원가입과 유저 활동을 동시에 할 수 없기 때문임

