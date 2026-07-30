# Yammer 데이터 분석 
[데이터리안 무료 강의](https://www.inflearn.com/course/%EB%8D%B0%EC%9D%B4%ED%84%B0-%EB%B6%84%EC%84%9D-sql-%EC%8B%A4%EC%A0%84/unit/72141?tab=curriculum)
- 이 강의를 왜 듣는가?
  - 데이터를 딱 받았을 때 어떻게 접근해야 할 지가 좀 막막한 점이 있었다.
  - SQL의 숙련도 이슈도 물론 있다.
  - 일단은 정확도에 집중한 다음 속도에 집중해야 하는데, 자꾸 2가지를 한꺼번에 잡으려고 하니까 산만해지는 점이 있다.
  - 따라서, 전문가는 어떻게 접근했는지 알아보고자 듣게 되었다.


## 여는 강의

- 따릉이 등의 공공데이터 분석도 의미가 있는데, 회사의 데이터를 다룰 기회가 많이 없음
- `Yammer` : 페북과 유사한 사내 메신저
- 현업에서 푸는 문제와 유사하고, 지금 풀고 있는 문제를 얘기할 수 있는 기회라 생각하여 강의를 기획함

### 케이스 3가지
1. 활성 유저의 감소 원인
2. 검색 기능 분석
3. A/B 테스트 결과 검증 

### 프로젝트 5단계
1. 준비 : 사전 자료 스스로 학습
    - 사전 자료를 스스로 학습하는 게 전체 학습량의 90% 이상임
    - 나오는 전문 용어를 직접 찾아보는 것도 필요
    - 베스트는 이 사전 자료만으로 공부를 마쳤다!라고 판단이 드는 거고
    - 강의 내용은 이해되지 않은 내용을 확인하거나, 강사가 제안하는 새로운 관점을 보는 정도로 파악하면 좋다.
2. 문제상황 : 프로젝트의 문제상황 파악
3. 분석 : 분석 해설
4. 주요 SQL 해설 : 분석에 사용한 주요 SQL 해설
5. 프로젝트 요약

## 1. 활성 유저의 감소 원인
- 시작하기 전에, 회사에 대해 알아두자.
### 1. 문제 파악
- `User Engagement Dashboard`
  - `DAU : Daily Active User / Weekly AU / Monthly AU` 
  - 특정 기간 동안 활성 유저를 파악함  

(문서 중) `Yammer`의 `Engagement` 정의 : `Server Call`을 발생시키는 모든 행위, `events` 테이블에 `Engagement`로 정의되어 있다.
- 해당 지표가 감소하고 있으니, 어떤 일이 있었는지 밝히자. (해결책도 같이 제시하면 좋을 것이다.)
- 실제로도 **매일 확인하는 지표(매출 등)는 사장이나 매니저가 계속 보는 요소**이다. 즉 많이 발생하는 케이스임.  

(문서 중) 데이터에 접근하기 전, **가능한 원인들을 쭉 적어보고 테스트할 순서를 만들어라.** 
- 여러 원인이 있을 수 있는데, 어떤 순서로 테스트할 지 생각해보고, 그 이유도 생각하라.
- 차트가 보여주는 것과 보여주지 않는 것도 생각하라.  
- **생각한 모든 것을 테스트해볼 순 없음.** 중요하다고 생각하는 것과 그 순서 및 이유를 생각해보자.

(강의 내용) `Engagement`의 정의는 회사마다 다를 수 있다. 로그인을 했을 때, 어떤 기능을 썼을 때, 결제를 했을 때 등 `Active`에 대한 정의가 다르다는 것.

### 2. 테이블 분석
(강의 내용) 4개의 테이블이 주어지는데, 3개의 테이블만 사용할 것.  
(강의 내용) 이 예제에는 테이블의 이름 및 각 Column에 대한 **설명**이 있지만, 이것도 사람이 하는 일이기 때문에 **여러분이 다니는 회사에는 없을 수도 있다**.   

- 테이블 설명은 내가 해본 거니까 생략함  


(문서 내용) 차트의 `View Query Results`를 누르면 쿼리 결과 테이블이 나오고 이는 복사해서 이용할 수 있음  

(문서 내용)
1. 처음에 세운 가설이 추가적인 질문을 하게 만드는가?
2. 만약 그렇다면 그게 무엇이고 어떻게 테스트할지?
3. 데이터만 갖고 대답할 수 없는 가설이라면, 어떻게 가설을 테스트할 수 있는가?(회사에 정말 다닌다고 생각하면 어떤 것들을 할 수 있을까?)
4. 어떤게 `Engagement Dip`에 가장 큰 영향을 줬는가?
5. 회사는 어떻게 대응해야 할까?  
(강의 내용) 이 질문들에 어떻게 대답할지 고민해보면 좋을 것 같다. 문서의 `Answers`가 마냥 정답은 아니다.

### 3. 쿼리하는 법
- 일단 문서에는 차트와 해당 차트를 만든 쿼리가 같이 있다. 쿼리 쓰는 법은 아니까 패스
- 직접 이런저런 쿼리를 작성하고 데이터를 뽑아보자.

### 4. 강사의 요약 내용
[분석 내용](https://docs.google.com/spreadsheets/d/1JccUfsUFzX_Mc80LqDRUpFEzqGtVVzhhW7tu8pk1RD0/edit#gid=231328655)
- 스프레드시트 깔끔하다..!
- 일단 어떤 부분을 분석했는가는 문서 내용과 동일하기 때문에 별도로 필기하진 않겠음
- 근데 쿼리 내용이 다르다는 걸 본 기억이 있어서 그건 검토해봄  
- 더 나아간 내용
  1. 8월 4일 이후의 WAU 감소가 정말 문제인가? 매 년, 매 분기 반복되는 패턴일 수 있다.
  2. 데이터를 분해하자
    - 전체를 부분으로 나누면 전체를 볼 때는 알 수 없었던 변동 원인을 관찰할 수 있음
    - 분석이 잘 안되면 `feature`를 몇 개 추가해서 데이터를 더 깊게 살펴볼 수 있다.
    - `WAU`, `유저 코호트`, `이메일 인게이지먼트` 등을 더 깊게 파고들 수 있다.
      - 고객 유입 채널(광고), 고객 정보(성, 연령, 사용기기, 거주 지역 등), 국가, 코호트 범위 등

### 5. 쿼리 내용
(강의 내용)  
- 어떻게 원하는 데이터 추출을 했는가를 추적
- 강사님이 보고 치는 게 아니라 직접 어떤 걸 조회할지 생각하면서 친 내용임
1. WAU 추적
```sql
SELECT DATE_TRUNC('week', occurred_at) AS week,
       COUNT(DISTINCT user_id) as weekly_active_user
  FROM tutorial.yammer_events e 
 WHERE occurred_at BETWEEN '2014-04-28 00:00:00' AND '2014-08-25 23:59:59'-- time_window
   AND event_type = 'engagement'
 GROUP BY week
 ORDER BY week
```
- `Mode`는 `PostgreSQL`을 씀 : `MySQL`과는 일부 용법이 다르다
  - 예를 들면 `DATE_TRUNC()`은 `MySQL`에는 없나봄?
- 문서에 있는 `WAU` 차트의 경우, `time window`를 주지 않았는데, 상용 테이블에서 이러면 큰일난다 : 데이터 조회 비용이 어마어마해짐
  - 따라서 **조회할 기간을 정해놓는 건 가장 먼저 할 일**이다.
- `GROUP BY`, `ORDER BY`에 쓰이는 `COLUMN`은 명시를 하는 것이 좋다 : 복잡한 쿼리일수록 어떤 기준으로 묶고 나열했는지 직관적으로 파악할 수 있도록!

2. 날짜와 가입자 수(ACTIVE, PENDING 포함)
```sql
SELECT DATE_TRUNC('day', created_at) AS signup_date
     , COUNT(user_id) AS signup_users
     , COUNT(CASE WHEN activated_at IS NOT NULL THEN user_id ELSE NULL END) as activated_users-- 활성화 유저 수
     
  FROM tutorial.yammer_users
 WHERE created_at BETWEEN '2014-06-01 00:00:00' AND '2014-08-31 23:59:59'
 GROUP BY signup_date
```
- 쿼리를 치며 유념할 것은, `스케일`에 관한 것이다.
  - 예를 들어 수십 명 단위를 생각하고 쿼리를 짰는데 결과가 수백 명이 나왔다면, 데이터가 잘못됐을 확률보다 쿼리가 잘못됐을 확률이 더 높다.
- **결과물을 예상하면서 쿼리를 짜야 한다.**  

3. 이메일 관련 쿼리 2가지 (ANSWER의 4번 하단, 5번 하단)
1) 단순 이메일 종류에 따른 숫자 카운트
```SQL
SELECT DATE_TRUNC('week', occurred_at) AS WEEK
     , action
     , COUNT(DISTINCT user_id) AS cnt_user
  FROM tutorial.yammer_emails em
 WHERE occurred_at BETWEEN '2014-04-28 00:00:00' AND '2014-08-25 23:59:59'
 GROUP BY week, action
```
- 문서의 내용은 쿼리 결과 표를 `날짜 - 종류1 카운트 - 종류2 카운트 - ...` 식으로 보고 싶어서 저렇게 짠 거임(피벗팅)
  - 피벗팅이 더 직관적이긴 하다. 대신 코드가 조금 더 복잡해지는 점은 있다.
- 근데 코드를 위처럼 짜고 그 결과를 복사해서 스프레드시트에 넣은 뒤 피벗테이블 기능을 이용할 수 있음!
  - 피벗테이블 기능이 이해가 안간다면 꼭 SQL로 할 필요는 없다!
- 실제로 위처럼 쿼리를 짠 다음 시각화 기능을 이용해도 비슷하게 나옴


2) 이메일이 보내진 뒤 5분 이내에 메일을 클릭했는가 아닌가
```sql
SELECT DATE_TRUNC('week', em1.occurred_at) as week
    , COUNT(CASE WHEN em1.action = 'sent_weekly_digest' THEN em1.user_id ELSE NULL END) as sent_digest_emails
    , COUNT(CASE WHEN em2.action = 'email_open' THEN em2.user_id ELSE NULL END) as sent_digest_opened
    , COUNT(CASE WHEN em3.action = 'email_clickthrough' THEN em3.user_id ELSE NULL END) as sent_digest_clickthrough
    -- 강의의 쿼리는 아래와 같다 : 아래 3개 주석도 위와 똑같이 작동함
    -- , COUNT(CASE WHEN em1.action = 'sent_weekly_digest' THEN em1.user_id ELSE NULL END) as lecture_digest_emails
    -- , COUNT(CASE WHEN em1.action = 'sent_weekly_digest' THEN em2.user_id ELSE NULL END) as lecture_digest_opened
    -- , COUNT(CASE WHEN em1.action = 'sent_weekly_digest' THEN em3.user_id ELSE NULL END) as lecture_digest_clickthrough

    -- 퍼센트로 표시 : float로 데이터를 꼭 바꿔줘야 한다
    , COUNT(CASE WHEN em2.action = 'email_open' THEN em2.user_id ELSE NULL END) / COUNT(CASE WHEN em1.action = 'sent_weekly_digest' THEN em1.user_id ELSE NULL END) :: float || '%' as open_pct
    , COUNT(CASE WHEN em3.action = 'email_clickthrough' THEN em3.user_id ELSE NULL END) / COUNT(CASE WHEN em1.action = 'sent_weekly_digest' THEN em1.user_id ELSE NULL END) :: float || '%'as ct_pct
FROM tutorial.yammer_emails em1
    LEFT JOIN tutorial.yammer_emails em2
      ON em1.user_id = em2.user_id 
     AND em2.occurred_at BETWEEN em1.occurred_at AND em1.occurred_at + INTERVAL '5 min'
     AND em2.action = 'email_open'
    LEFT JOIN tutorial.yammer_emails em3
      ON em1.user_id = em3.user_id 
     AND em3.occurred_at BETWEEN em2.occurred_at AND em2.occurred_at + INTERVAL '5 min'
     AND em3.action = 'email_clickthrough'
WHERE em1.occurred_at BETWEEN DATE('2014-07-01') AND DATE('2014-08-31')
  AND em1.action = 'sent_weekly_digest'
GROUP BY week

```
- 문서 내용은 엄청 복잡함(서브쿼리에 서브쿼리에 서브쿼리)
- 근데 `SELF JOIN`과 `ON` 조건을 이용하면 꽤 많이 추릴 수 있음
- 특히 `LEFT JOIN`과 `Condition`을 어떻게 활용하는지를 집중적으로 보자 : 원본의 매우 복잡한 쿼리를 단순화하는 좋은 쿼리라고 생각함
- `SELECT`문의 위, 아래 두 방법 모두 잘 작동함(왜인지 모를 이슈로 아래 쿼리가 작동 안했음)
- 강의에서 구글 스프레드시트를 활용할 것을 권장하고 있음
  - SQL로 피벗테이블을 하려면 해당되는 테이블 하나하나를 CASE WHEN 문에 집어넣어줘야 함
  - 스프레드시트로 피벗테이블을 하겠다면 SQL로 `GROUP BY`까지만 하고 그 결과를 옮기면 더 쉽고 빠르게 결과를 볼 수 있음.
- 집계 값들을 연산하고자 할 때(특히 백분율) -> `float`로 값을 **꼭 !** 변환해야 한다. 변환 안하면 그냥 0으로 나옴.

## 2. 검색 기능 문제
- 여기서도 똑같이 쿼리를 어떻게 짜는지에 대해서만 짚고 넘어간다. 나머지 내용은 문서를 따라가고 있기 때문.
  
### 세션 만드는 방법
- `Yammer`의 세션은 일련의 연속된 행동(서버와의 상호작용)으로 이루어졌으며, 이전 행동과 다음 행동의 시간 차이가 `10분` 이상이라면 두 행동은 별도의 세션이다.
- 이것도 사실상 문서의 쿼리를 그대로 설명하는 내용이라 따로 필기할 필요는 없겠다.
```sql
SELECT sess.*
  FROM 
(
SELECT e.user_id, 
       e.occurred_at, 
       e.event_name, 
       x.session, 
       x.session_start, 
       x.session_end
  FROM tutorial.yammer_events e 
  LEFT JOIN (
            SELECT session.user_id,
                  session.session,
                  MIN(occurred_at) as session_start,
                  MAX(occurred_at) as session_end
              FROM ( 
                  SELECT final.*,
                  -- 세션 정의
                  CASE WHEN final.last_event >= INTERVAL '10 min' THEN final.id 
                        WHEN final.last_event IS NULL THEN final.id 
                        ELSE LAG(id) OVER (PARTITION BY user_id ORDER BY occurred_at) END AS session
                      FROM (
                          SELECT bound.*
                            FROM (
                              SELECT e.user_id
                                    , e.occurred_at
                                    , occurred_at - LAG(e.occurred_at, 1) OVER (PARTITION BY e.user_id ORDER BY e.occurred_at) as last_event
                                    , LEAD(e.occurred_at, 1) OVER (PARTITION BY e.user_id ORDER BY e.occurred_at) - occurred_at as next_event
                                    , e.event_name
                                    , ROW_NUMBER() OVER () AS id
                                    
                                FROM tutorial.yammer_events e 
                              WHERE e.event_type = 'engagement'
                              ORDER BY user_id, occurred_at
                                  ) bound
                  WHERE last_event IS NULL 
                      OR next_event IS NULL
                      OR last_event >= INTERVAL '10 min'
                      OR next_event >= INTERVAL '10 min'
                ) final
            ) session
              GROUP BY session.user_id, session.session
          ) x
          ON e.user_id = x.user_id 
         AND e.occurred_at >= x.session_start 
         AND e.occurred_at <= x.session_end
 ORDER BY user_id, occurred_at
) sess
```
- 위 쿼리가 세션으로 각 행동들을 묶은 쿼리이고, 여기서부터 분석을 시작하면 됨

- ex1) `autocomplete`과 `run`이 있는 세션의 수
```sql
SELECT DATE_TRUNC('week', occurred_at) AS week,
       event_name,
       COUNT(session) as session
  FROM ~~sess~~
WHERE event_name IN 'search_autocomplete, search_run'
GROUP BY week, event_name
ORDER BY session DESC
```

- ex2) 기간에 따른 1회 세션 당 `autocomplete`와 `run`의 평균 실행 횟수
```sql
SELECT week,
       SUM(autocompletes) / COUNT(*) as avg_ac,
       SUM(runs) / COUNT(*) as avg_runs
  FROM 
(
SELECT DATE_TRUNC('week', occurred_at) as week
     , session
     , COUNT(CASE WHEN event_name = 'search_autocomplete' THEN session ELSE NULL END ) as autocompletes
     , COUNT(CASE WHEN event_name = 'search_run' THEN session ELSE NULL END ) as runs
  FROM (~~sess~~)
WHERE event_name ILIKE 'search_%'
GROUP BY week, session
) y
GROUP BY week
```
## 3. A/B 테스트

- 여기는 통계에 대한 전문 내용도 나와서 감히 진행할 수 없었던 부분이니 강의를 풀로 보자.
- A/B 테스트에 대한 내용은 옵시디언에 정리했다.  
- (강의 내용) : A/B 테스트, t-test, p-value 등은 통계학 용어. 여기서는 깊게 들어가진 않음
- 통계학에 관한 내용은 알아둬야 한다는 거!
  - 특히 `구간 추정`에 관한 내용은 꼭.
- `대조군Control Group` vs `실험군Treatment Group`
  
### 문제
- A/B 테스트 결과, 실험군의 메시지 포스팅이 50%나 높게 나왔음
- 근데 50%는 말이 안되는 수치임
- 따라서 실험이 올바르게 수행되었는가를 살펴볼 필요가 있음
- 튜토리얼 문서의 차트를 보면 6월 동안 실험군은 4.07회, 대조군은 2.669회
  - `A/B 테스트`의 전문가가 되겠다면 `rate_difference, rate_lift, stdev, t_stat, p_value`에 대한 이해가 필요함
  - 아니라면 집중적으로 다뤄볼 지표는 아니니 패스함

(강의 내용)
- 통계학 책에서 분포, 구간 추정 등을 뚫어내면 그 뒤에 `p-value`나 `t-test` 등이 나옴
- 알아두면 좋으나 요 강의에서는 패스

결론은 A/B 테스트가 유효한 결과인지 실험을 다시할 필요가 있는지?

(문서 : 대충 가정 세우고 테스트 어떻게 할 건지 생각하라 - 이런 식으로 50%나 바뀌는 경우는 잘 일어나지 않는다.)

(강의 내용)
- 데이터를 볼 때 주의할 내용 : 실무에서 볼 데이터는 여기 예제와는 완전히 다름
  - 이렇게 **실험군과 대조군이 완전히 차이나는 테스트는 강사가 실무에서 본 적 없다.**
- 따라서 실제 문제는 훨씬 복잡함 
  - EX) 실험군과 대조군의 차이가 크지 않다면?
    - 두 실험군이 통계적으로 유의미한 차이를 가진 집단인가? -> 통계학적 지식이 들어감
    - 차이가 적을수록 표본이 훨씬 많이 필요함
      - EX) 연예인과 내 외모 비교(큰 차이 -> 소수의 데이터로 파악 가능) vs 내 친구와 내 외모 비교(**적은 차이 - 표본이 많이 필요해짐**)
      - 표본이 많이 필요한 경우, 분석가는 혼자서 결정할 수 없음 : 상품을 담당하는 사람과 반드시 커뮤니케이션이 필요함.
      - 실험이 커질수록 위험해질 가능성도 커짐
      - 심지어 그 결과가 실질적인 차이가 없다로 이어질 수도 있음 : 별 차이가 없는데 갈아 엎어야 함?
    - **A/B 테스트는 많은 사람이 얽히기 때문에 커뮤니케이션 능력도 중요**함. 통계 공부가 전부가 아님

### 테이블 분석
- `Experiment` 
  - `user_id`
  - `occurred_at`
  - `experiment` : 어떤 실험에 속했는지
  - `experiment_group` : 어떤 그룹에 속했는지
  - `location`
  - `device`

- 참고) **한 프로덕트에 대해** A/B 테스트는 하나만 돌아가지 않음 : **여러 A/B 테스트가 동시에 수행되고 있음** 
  - 따라서 실험 내에서 유저가 어떤 그룹에 속했는지`experiment_group`
  - 쿼리로 친다면 `experiment`로 `group by`가 되겠지?
  - 한 실험에서 유저는 테스트 기간 동안 여러 그룹에 있으면 안 됨
    - 실제로는 왔다갔다 하는 A/B테스트도 있음 (이전에 어떤 링크에서 본 모든 유저가 다른 화면을 볼 수 있는 걸 얘기하는 듯)
      - 예를 들면 유튜브 추천 알고리즘 : 직접적으로 유저가 파악하기 힘든 프로덕트에 대해서는 꼭 각 유저에 대한 일관성을 유지할 필요는 없다.
    - 이 실험은 **UI**에 대한 내용인데, **화면의 일관성을 유지할 필요**가 있어서 위 얘기가 나옴

### (문서) Validating Result
- 문서에 있는 차트 쿼리가 잘 짜여졌는지를 살펴보자. 50%의 차이가 났다는 건 쿼리에 문제가 있을 가능성이 있다.
- `Check Other Metrics to make sure that this outsized result is not isolated to this one metric` -> 포스팅 횟수에서 50%의 차이가 났지만, **다른 지표**(매출이라든가..)**에 대해서도 살펴보라**는 뜻
  - **면접에서도 많이 나오는 질문** : A/B 그룹에서 별 차이가 없고, 통계적으로도 이 지표에 대해서 별 차이가 없다는 게 증명되었을 때 분석가로서 어떻게 추가적인 접근을 할 수 있는가? 
    - EX) 포스팅 횟수에 대해선 좋은 성장을 보였을 수 있지만, 로그인이라든지 매출이라든지 **코어로 생각하는 지표**를 살펴볼 필요도 있다. 
- 데이터가 옳은가를 살펴보라 : 테스트 결과가 옳은 방식으로 기록되었는지, 유저의 실험군 / 대조군 분포가 올바르게 이루어졌는지 등. 만약 뭔가 잘못됐다면 문제를 고칠 방법들을 결정하라
- 결론에 따른 추천을 해보라 : 새로운 퍼블리셔가 공개되어도 괜찮은가? 실험을 다시 해야 하는가? 만약 그렇다면 뭐가 달라져야 하는가?
(강의 내용)  
- 분석 파트는 통계적인 내용이 많이 없음 : 앞의 두 프로젝트보다 훨씬 쉬움
- 따라서 직접 가설을 세워보고 접근하자
[데이터리안 강의 요약](https://docs.google.com/spreadsheets/d/1NCGVlsQ3CN79oWu5P1tT-I2TuvfRmOTgwwAerysm_vI/edit#gid=285114659)
  - 더 나아가는 내용에 좋은 글들이 있다. 1번은 위에 적은 거고, 2번은 페이스북의 A/B테스트와 윤리에 관한 이야기이다.
``` 
-- 직접가설
1. 실험군 / 대조군으로 분류된 유저의 숫자는 큰 상관이 없을까? 물론 얼추 기억나는 통계학에 의하면 표본집단이 모집단의 분포를 따라간다면 큰 문제는 없는 것으로 알고 있다.
2. 숫자가 상관 없다면, 표본의 분포가 모분포를 따라가는가도 생각해볼 수 있을 것이다 : 근데 여기 관여하는 요인을 추리기는 쉽지 않을 듯
```

### Preparation and Prioritizing
(문서 내용)  
- 데이터를 보기 전, 유저의 행동에 미친 특징이 뭐고, 그 이유는 뭘지 가설을 세우는 건 중요하다
- **가설을 세우지 않고 출발하면, 어설프게 이것저것 하다가 끝나는 경우가 많다.**(겁나 많이 겪어봄)
  - 따라서 분석의 목표지점을 명확하게 정하고 시작하는 것이 중요하다. 왔다갔다 하지 않게끔.
- 여러 예시가 있다.
1. `Posting Rate`는 `Publisher`의 성공을 측정하는 좋은 지표`Metrics`가 아닐 수 있다 : `Publisher`는 포스팅을 하기 위한 수단일 뿐, 사용자들이 `Publisher`의 가치를 뽑아내고 있는 게 아닐 수 있다.
  - 예를 들면 `게시` 버튼을 엄청 크게 만드는 게 `Yammer`에게 도움이 될까?
  - 따라서 **다른 지표들과 함께 살펴보는 게 중요**하다.
2. 테스트가 잘못되었을 가능성 : 수학 계산(`Arithmetic`)이 잘못됐을 가능성. `1+2/3 = 5/3 , (1+2)/3 = 1`
3. 사용자를 그룹에 잘못 뿌렸을 가능성 : `random`이 유지되지 않을 가능성이 있다.
  - 실험군에 여자만, 대조군에 남자만 있다면? `random`이 아니죠?
4. (가장 어려움) : 독립변수가 아닌데 종속변수에 영향을 주는 뇨속이거나, 
```
X : 독립변수 - 실험자가 변인을 파악하기 위해 설정한 변수
Y : 종속변수
X->Y 를 파악하기 위해 실험자는 이외의 변수들은 변인이 아니라고 봄
Confounding Factor : 종속변수에 영향을 주면서 독립변수가 아닌 변수들

ex) 
커피를 많이 마시면 폐암에 걸린다는 결론이 나왔다
근데 담배를 피는 사람들은 커피도 많이 마실 수 있다는 게 밝혀졌다고 하자
담배가 폐암에 영향을 준다는 통념을 생각한다면,
담배는 커피 + 폐암 모두에 영향을 주는 Confounding Factor가 된다.
이 경우
(커피 -> 폐암)이 아니라
(커피 -> 담배 -> 폐암)이 된다.
```
### 분석 1
- 강의 내용은 문서를 따라간다 : 그러나 여기는 내가 통계 용어에 막혀 답안을 보지 않았기 때문에 쭉 듣는다

메시지의 숫자는 실험의 성공을 결정짓는 유일한 요소가 아니므로 다른 지표를 활용한다 : 특히, `Yammer`에서 유저가 뽑아내는 가치에 대한 지표에 관심이 있는데, `Yammer`는 `Login Frequency`를 `Core Value`로 사용한다. **적어도 이 회사에서 `Login Frequency`를 높여주는 기능은 좋은 기능이라고 할 수 있겠다.**

우선, 유저 당 평균 로그인 횟수가 올랐다. 이는 유저들이 메시지를 더 많이 보낼 뿐만 아니라, `Yammer`에 더 많이 가입했다는 의미이기도 하다.

추가로, 6월 동안의 계정 당 고유한 로그인 날짜 수 파악도 필요하다.(하루에 4번 로그인한 것과 1개월에 걸쳐 4번 로그인한 건 분명 다르니까) 만약 로그인 날짜는 그대로인데 로그인 횟수만 올랐다면 로그인 버그도 생각해볼 수 있다(문서 내용은 이런데, 강사는 이게 약간 오바라고 생각한다)  
반대로 2개의 지표가 같이 올랐다면(계정 당 평균 로그인 수와, 계정 당 로그인한 평균 날짜 수) 유의미한 지표

강사는 **구글 스프레드 시트를 자주 활용**한다 : 쿼리 후 `Mode`의 결과를 쉽게 옮길 수 있기도 하고.. **정리하기도 매우 좋아보인다!**


### 분석 2 : 강의에서는 패스
2번째 항목은 A/B 테스트에 적용하는 수학적인 방법에 대한 이야기가 초반에 나온다 (단/양측 테스트, 표본 크기, 표본 분포 가정 등)  
이 테스트에는 방법론적 오류가 있다 - **신규 사용자와 기존 사용자를 동일한 그룹에 묶고 테스트 기간 동안 게시한 메시지 수를 측정**한다 - 즉 **1월에 가입한 사용자와 6월 29일에 가입한 사용자가 같은 그룹에 묶일 수 있다.**  
**신규 사용자와 기존 사용자를 별도로 고려하는 것이 더 합리적**이며, 이렇게 하면 크기 비교가 더 적절해지고 `Novelty Effect`를 적용할 수도 있다. (`Novelty : 새로운 / 신기한` 아래 내용이 `Novelty Effect`)
`Yammer`에 익숙한 사용자는 새로운 기능이라는 이유로 테스트를 해볼 수 있고, 신규 사용자는 새로운 기능을 보더라도 자신에겐 새로운 기능이 아님 : 내가 처음 본 게 그 새로운 기능이기 때문임 -> 그래서 `참신하다는 이유로` 클릭할 상황은 없을 것임

### 분석 3
- 기존 / 신규 유저로 나눴더니 문제가 하나 나왔다 : 6월에 가입한 사람이 모두 `Control Group`에 속해 있었다.(`Treatment Group`이 없다)
- X축은 가입 시기, Y축은 유저 수, Groupby는 실험/대조군
- 이 경우는 `분석 1`에서 수행한 각 그룹 별 포스트 수에 영향을 미치는 요소임 : 물론 까보기 전에는 최근에 가입했다는 이유로 포스트 수가 적을 거라는 예단을 하는 건 금물이다 : 신나서 막 포스팅했을지는 까보기 전에는 모르는 것임
- 중요한 건 기존 유저와 새로 가입한 유저의 특질 차이를 예상할 수 있으므로, 이들을 구분해야 한다는 것이다.
- 따라서 실험 기간인 6월에 가입한 사람들을 `Control Group`에서 제외하고 다시 비교했을 때, `Control Group`의 평균 메시지 전송 수가 `2.66 -> 2.91`로 상승함.
- `The test results narrow considerably` : 실험 결과의 폭(두 그룹의 차이)이 좁아졌다  


### 분석 4.
- 하나의 문제를 찾았음. 다른 가능성들을 생각하는 것도 좋을 것임
- 왜냐면 **1개의 문제만이 있는 경우는 잘 없음** (`하나의 문제만으로 어떤 현상을 모두 설명할 수 있을 것이다!` 라는 함정에, 데이터 분석가들은 자주 빠진다.)
- 다른 분류(`Cohort`)들도 가능 : `기존 vs 신규 유저`, `기기별 사용`, `유저 타입 별 사용`
--------------------------

- 큰 맥락에서 결론이 변한 것은 아님 
- 로그인 횟수에 대해 차트를 만드는 것도 좋겠죠?
- 이 레포트의 상황(실험 기간 동안의 가입자들이 죄다 한쪽 그룹으로 몰리는 경우)은 강사도 겪은 적이 있음. 생각보다 자주 나오는 상황이라고 한다. 

### 쿼리 리뷰
- 계속 1개의 쿼리를 사용했기 때문에 이를 응용해서 이용하게끔 정리해보자
- 실험 기간 동안 유저 당 평균 메시지 건수 정리
- 직접 써보고 강의 내용을 따라가보자
```sql
-- 목표 : 실험 / 대조군의 각 유저 당 포스팅한 게시글 수
SELECT 
      -- DATE_TRUNC('day', sub.occurred_at) as day
      sub.experiment_group as group
    -- , COUNT(sub.event_name)
    , COUNT(CASE WHEN sub.event_name = 'send_message' THEN user_id ELSE NULL END) AS message_counts
    , COUNT(DISTINCT user_id) as users
    , COUNT(CASE WHEN sub.event_name = 'send_message' THEN user_id ELSE NULL END) / COUNT(DISTINCT user_id) :: float AS message_per_account
  FROM (
SELECT ex.user_id
     , e.occurred_at
     , ex.experiment_group 
     , e.event_name
  FROM tutorial.yammer_experiments ex
  JOIN tutorial.yammer_events e -- LEFT를 넣어야 맞지 않나 싶음
    ON e.occurred_at BETWEEN date('2014-06-01') AND date('2014-07-01') - INTERVAL '1 sec'
   AND ex.user_id = e.user_id -- 레전드네 ㄹㅇ
       ) sub
 GROUP BY "group"

```
- `ON` 조건에 `ID` 안 쓰고 외않되? 이러고 있던게 레전드;
- 직접 쓰면서 느끼는 건데, 집계 함수를 적용하기 전의 테이블을 내가 보면서 쿼리를 짜는 게 맞음 
  - 안 그러면 ㅈㄴ 헷갈림 : 같은 쿼리도 여러 번 돌리게 됨
  - 그리고 가능한 서브쿼리 테이블을 살려놓는 게 좋다 : 여러 `METRIC`에 대해 위에서 집계 함수를 적용하려면 원본 테이블을 가능한 유지하는 것이 좋을 것임
- `COUNT(DISTINCT)` 겁나 오래 걸림

#### (강의 내용)
- 집계함수가 없다면 굳이 서브쿼리를 쓸 필요는 없다 : `JOIN`을 활용하는 방법이 있음.

```SQL
SELECT u.user_id,
    --  , u.activated_at
    --  , ex.experiment
     , ex.experiment_group
     , ex.occurred_at -- 계정이 어떤 그룹에 속했는지가 정해진 시점
     , COUNT(e.user_id) AS cnt_send_messages 
  FROM tutorial.yammer_experiments ex
       JOIN tutorial.yammer_users u 
         ON ex.user_id = u.user_id
       LEFT JOIN tutorial.yammer_events e
         ON ex.user_id = e.user_id
         AND e.occurred_at BETWEEN ex.occurred_at AND '2014-06-30 23:59:59' 
         AND e.event_name = 'send_message'
 WHERE ex.experiment = 'publisher_update'
 GROUP BY u.user_id, u.activated_at, ex.experiment, ex.experiment_group, ex.occurred_at
```
- 이벤트가 `send_message`가 아니라면 JOIN된 e테이블에 해당하는 항들은 NULL이 뜰 것이다 ->`JOIN`된 e.user_id의 수는 `send_message`의 수와 동일하다.
- 위 쿼리는 결과적으로 각 유저가 몇 번의 메시지를 보냈는지를 정리하게 됨
- 서브쿼리가 많아질 때 `a, b, c`식의 바이어스를 준다면 그 서브쿼리가 어떤 역할을 하는 지 알 수 없게 됨
  - 따라서 **서브쿼리 바이어스를 그 역할을 설명하는 이름**으로 정해주든가, **바이어스 옆에 주석**이라도 달아두는 게 중요하다.

위 쿼리를 sub라고 한다면..
```sql
SELECT experiment_group
     , COUNT(user_id) AS users -- DISTINCT 필요 없음 : 서브쿼리에서 ex.user_id는 1번만 등장하기도 하고, GROUP BY로 묶었기 때문이기도 하다.
     , AVG(cnt_send_message) as 'avg' -- 그룹별 
     , SUM(cnt_send_message) as total
  FROM () sub
 GROUP BY experiment_group
```
- 중복되는 조건(주로 ON이나 WHERE문에서 걸러지는)은 굳이 `SELECT`에서 다시 조회할 필요는 없다.
- **쿼리는 최대한 간결하게 짜는게 좋다.**
  - 쿼리를 짜면서 이런저런 생각이 많이 드는 게 사실인데, 가장 깔끔하게 쿼리를 짜놔야 유지관리가 쉽기 때문이다.
  - 따라서 쿼리를 짠 뒤 필요 없는 부분을 쳐내는 작업을 하는 게 좋다.

### 마무리
- `Mode`가 초보자가 접근하기 좋지만, `굳이 저렇게 짜야 했나` 싶은 부분도 있다. 테이블 피봇팅이라든지, 서브쿼리라든지
- 따라서 직접 쿼리를 짜면서 "이런 식으로 하는 것도 좋겠다"라고 비판하면서 쿼리를 짜보시라는 것!