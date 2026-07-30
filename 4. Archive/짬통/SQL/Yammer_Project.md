# Yammer Project
[실습 예제](https://mode.com/sql-tutorial/sql-business-analytics-training/)
- `Yammer` :  협업 툴임
  - 분석 팀의 목표 : 데이터를 이용한 더 나은 생산품 & 사업적 결정
- 시각화는 `Mode`에서 제공하는 툴을 이용함
- 실무와 효율에 관한 이야기들도 있으니 살펴보면 유용할 것이다

## 1. 사용자 참여`User Engagement`의 감소 조사
### 1. 매 주 참여(`Engagement`)하는 유저의 수 추이 파악
```sql
SELECT DATE_TRUNC('week', occurred_at) AS week,
       COUNT(DISTINCT user_id) AS num_of_engaged_users
  FROM tutorial.yammer_events 
 WHERE event_type = 'engagement'
 GROUP BY 1
 ORDER BY 1 desc
```
![visualization](users_per_week.PNG)
* **중요 : 더 깊게 들어가기 전에, 유저 감소의 가능한 원인을 생각해보자(가설을 세우라는 것!)**
    - 앞으로 할 일은 이 가설을 검증해보는 것이 될 것이다
    - **가설을 잘 세우는 건 중요**함 : **데이터를 살펴보는 시간을 줄일 수 있기 때문**이다. 
    - 물론 **정답은 없음** : 브레인스토밍으로 가능한 경우의 수들을 뽑아내는 것
* 또한 위 차트가 보여주는 정보와 보여주지 않는 정보를 명확히 알 필요도 있다.
```
1. 왜 유독 8월에 떨어졌는가 -> 국가나 지역별로 검증할 필요도 있을 것 같음
2. 신제품의 출시 여부도 영향이 있을 것이다 : 8월이 신제품 비수기일 수도 있고?
그 외엔 테이블에 대해 알고 있는 게 많이 없어서 잘 몰?루겠는데..
//
튜토리얼에서 생각한 것들은 아래와 같다
1. 휴일(증명 : 특정 국가의 참여만이 낮아졌다면 뒷받침할 근거가 됨)
2. 어플리케이션 손상 가능성(예를 들면 회원 가입이 막혔다면 유입이 줄어드니까 성장이 낮아질 수 있음)
3. 추적 코드 손상 가능성(새로운 데이터가 생기지 않는다면 ㅇㅋ)
4. 봇에 의한 비정상적인 트래픽
    - 봇의 패턴을 분석해야 하므로 결정하기가 어려워짐
5. 우리 사이트에 대한 트래픽 차단
6. 마케팅 이벤트
7. 데이터가 후져서
8. 크롤러의 변경
```
- 가능성이 많기 때문에 효율적으로 움직이는 게 중요하다. 시간 낭비 방지 대책으로는
    1. 경험 : 산업에서 몇 번 겪어보면 가장 빈번한 문제가 무엇인지 알 수 있음
    2. 의사소통 : 어떤 기간에 무슨 일이 있었는지 타인에게 물어보는 건 정보를 얻을 수 있는 가장 쉬운 방법임
    3. 속도 : 일부 가설은 더 쉽게 볼 수 있다 : 해봤거나, 데이터나 쿼리가 쉬운 경우임
        - 이런 테스트는 먼저 해보는 게 좋음
    4. 의존성 : 어떤 시나리오를 테스트 했을 때 이해하기 쉬워지는 다른 시나리오가 있다. 순서대로 할 것.

### 2. 가설 검증을 위한 데이터 탐색
- 이 예제에서는 4개의 테이블이 있다.
1. `Users`
- `user_id`, `created_at`, `state`, `activated_at`, `company_id`, `language`

2. `Events`
- `user_id`, `occurred_at`, `location`, `device`
- `event_type`
  - `signup_flow` : 유저의 인증 과정에서 참조한 것들
  - `engagement` : 가입 후 일반적인 상품과 관련된 것들 
- `event_name`
  - `create_user`
  - `enter_email`
  - `enter_info`
  - `complete_signup`
  - `home_page` : 홈페이지를 유저가 로드함
  - `like_message` : 좋아요
  - `login`
  - `search_autocomplete` : 자동완성 리스트에서 유저가 검색 결과를 선택함
  - `search_run` : 유저가 탐색 쿼리를 돌린 뒤 결과를 받음
  - `search_click_result_X` : 검색 결과 `X = 1~10` 중 `X`를 선택함
  - `send_message` : 메시지 게시
  - `view_inbox`
3. `Email Events`
- `user_id`, `occurred_at`
- `action`
  - `sent_weekly_digest`
  - `email_open`
  - `email_clickthrough` : 이메일 내의 링크 클릭
4. `Rollup Periods`
- 연속된 **시계열을 만들기 위한 테이블**. 
- `INTERVAL` 함수가 있지만 시계열에서 `rolling time`을 만들기 위해 사용되는 방법으로 쉬워서 자주 사용됨.

---------------------------
### 3. 위 가설들로 직접 탐색해볼 것
1. 국가 별 `Engagement` 추이
1) 모든 국가를 보는 게 좋은 방법은 아닌 것 같아서, 유저 수가 많은 상위 10개 국가에 대해 조사
    - 근데 유저의 국적을 추적할 방법이 없음(사용 언어 뿐)
    - 그나마 기업의 국적을 추적하는 방법을 생각했는데, 다국적 기업일 수도 있음(실제로 회사는 인도인데 사용 언어는 독일어, 한국어, 영어, 아랍어 등 다양하게 나옴) 
2) 지난 1년간 `engagement`에 관여한 아이디가 가장 많은 국적을 조사
![국가별](top10_by_nation.PNG)
```sql
SELECT sub2.location,
       sub2.week,
       sub2.engagement_counts,
       SUM(sub2.engagement_counts) OVER
       (PARTITION BY sub2.week ORDER BY sub2.week) AS summation_of_week
  FROM (
SELECT e.location AS location,
       DATE_TRUNC('week', occurred_at) AS WEEK,
       COUNT(DISTINCT e.user_id) AS engagement_counts
  FROM (
        SELECT location,
              COUNT(DISTINCT user_id) AS users_in_nations
          FROM tutorial.yammer_events
         WHERE occurred_at <= NOW() - INTERVAL '1 years'
         GROUP BY 1
         ORDER BY 2 DESC
         LIMIT 10
       ) sub
  JOIN tutorial.yammer_events e 
    ON sub.location = e.location 
 WHERE e.event_type = 'engagement'
 GROUP BY 1, 2  
      ) sub2
 ORDER BY 2 DESC
 ```

2. 유입이 줄었는가?
```sql
SELECT DATE_TRUNC('week', created_at) as week,
       COUNT(CASE WHEN state = 'pending' THEN 1 ELSE NULL END) as pended,
       COUNT(CASE WHEN state = 'active' THEN 1 ELSE NULL END) as activated
  FROM tutorial.yammer_users
 WHERE created_at >= date('2014-05-01')
 GROUP BY 1 
 ORDER BY 1
 ```
- 회원 가입이 막혔는지를 보는 쿼리.
- ![회원 가입](회원가입.PNG)

미국을 더 깊게 파보면
```sql
SELECT sub.week,
      COUNT(sub.user_id) as counts_total_users,
      COUNT(CASE WHEN sub.state = 'active' THEN 1 ELSE NULL END) AS active,
      COUNT(CASE WHEN sub.state = 'pending' THEN 1 ELSE NULL END) AS pending
  FROM( 
        SELECT DISTINCT DATE_TRUNC('week', e.occurred_at) AS week,
              e.user_id,
              u.state
          FROM tutorial.yammer_events e
          RIGHT JOIN tutorial.yammer_users u
            ON e.user_id = u.user_id
          AND u.created_at >= date('2014-05-01')
          AND location = 'United States'
      ) sub
  group by 1
  order by 1
```
![미국 가입자 수 추이](us_sign_up.PNG)
- 가입했는데 승인이 안 된 `pending`의 증가세가 있지만, 이게 중요한 요소인가는 모르겠음.
- 한편 서브쿼리에 `engagement` 조건을 넣으면 (당연하게도) `pending = 0`이 뜸.
------------------------
### 4. 이 튜토리얼의 답지를 따라가보자
#### 1. 성장 : 신규 유저 유입 조회
```sql
SELECT DATE_TRUNC('day',created_at) AS day,
       COUNT(*) AS all_users,
       COUNT(CASE WHEN activated_at IS NOT NULL THEN u.user_id ELSE NULL END) AS activated_users
  FROM tutorial.yammer_users u
 WHERE created_at >= '2014-06-01'
   AND created_at < '2014-09-01'
 GROUP BY 1
 ORDER BY 1
 ```
![ans_1](ans_1.PNG)
- 일 단위로 가입자 수를 나타낸 그래프이다.
- 요일 단위로 추적했을 때 평일은 높고 주말은 낮은 추이를 보인다.
- 유저 수 감소에 중요한 요소는 아님을 알 수 있음 : 비슷, 혹은 증가하는 추이를 보이기 때문임

#### 2. 기존 유저가 감소했는가 조회
```sql
SELECT DATE_TRUNC('week',z.occurred_at) AS "week",
       AVG(z.age_at_event) AS "Average age during week",
       COUNT(DISTINCT CASE WHEN z.user_age > 70 THEN z.user_id ELSE NULL END) AS "10+ weeks",
       COUNT(DISTINCT CASE WHEN z.user_age < 70 AND z.user_age >= 63 THEN z.user_id ELSE NULL END) AS "9 weeks",
       COUNT(DISTINCT CASE WHEN z.user_age < 63 AND z.user_age >= 56 THEN z.user_id ELSE NULL END) AS "8 weeks",
       COUNT(DISTINCT CASE WHEN z.user_age < 56 AND z.user_age >= 49 THEN z.user_id ELSE NULL END) AS "7 weeks",
       COUNT(DISTINCT CASE WHEN z.user_age < 49 AND z.user_age >= 42 THEN z.user_id ELSE NULL END) AS "6 weeks",
       COUNT(DISTINCT CASE WHEN z.user_age < 42 AND z.user_age >= 35 THEN z.user_id ELSE NULL END) AS "5 weeks",
       COUNT(DISTINCT CASE WHEN z.user_age < 35 AND z.user_age >= 28 THEN z.user_id ELSE NULL END) AS "4 weeks",
       COUNT(DISTINCT CASE WHEN z.user_age < 28 AND z.user_age >= 21 THEN z.user_id ELSE NULL END) AS "3 weeks",
       COUNT(DISTINCT CASE WHEN z.user_age < 21 AND z.user_age >= 14 THEN z.user_id ELSE NULL END) AS "2 weeks",
       COUNT(DISTINCT CASE WHEN z.user_age < 14 AND z.user_age >= 7 THEN z.user_id ELSE NULL END) AS "1 week",
       COUNT(DISTINCT CASE WHEN z.user_age < 7 THEN z.user_id ELSE NULL END) AS "Less than a week"
  FROM (
        SELECT e.occurred_at,
               u.user_id,
               DATE_TRUNC('week',u.activated_at) AS activation_week,
               EXTRACT('day' FROM e.occurred_at - u.activated_at) AS age_at_event,
               EXTRACT('day' FROM '2014-09-01'::TIMESTAMP - u.activated_at) AS user_age
          FROM tutorial.yammer_users u
          JOIN tutorial.yammer_events e
            ON e.user_id = u.user_id
           AND e.event_type = 'engagement'
           AND e.event_name = 'login'
           AND e.occurred_at >= '2014-05-01'
           AND e.occurred_at < '2014-09-01'
         WHERE u.activated_at IS NOT NULL
       ) z

 GROUP BY 1
 ORDER BY 1
LIMIT 100
```
![account_age](accounts_age.PNG)
- 서브쿼리에서는 3가지 특성을 새로 만들었다.
  1. 계정을 활성화한 주
  2. 계정 활성화한 후 주문까지 걸린 기간(두 timestamp 간의 날짜 차이를 `day`로 나타냄)
  3. 계정의 나이(활성화 시점이 생일)
- 메인 쿼리에서는 서브쿼리의 정보를 활용해
    1. 주문한 계정들의 평균 나이
    2. 주문과 계정 나이에 대한 디테일
       - `DISTINCT CASE WHEN ~~` 문은 참고할 만하다.
- 얻을 수 있는 정보 : 10주 이상된 계정들의 `engagement` 수가 감소하고 있음.
- 따라서 **일시적인 사건들(일회성 급증, 순위 변경, 차단 등)에 의한 감소가 아님**을 알 수 있다. 
#### 3. 다양한 장치들에 대한 분석
```sql
SELECT DATE_TRUNC('week', occurred_at) AS week,
       COUNT(DISTINCT e.user_id) AS weekly_active_users,
       COUNT(DISTINCT CASE WHEN e.device IN ('macbook pro','lenovo thinkpad','macbook air','dell inspiron notebook',
          'asus chromebook','dell inspiron desktop','acer aspire notebook','hp pavilion desktop','acer aspire desktop','mac mini')
          THEN e.user_id ELSE NULL END) AS computer,
       COUNT(DISTINCT CASE WHEN e.device IN ('iphone 5','samsung galaxy s4','nexus 5','iphone 5s','iphone 4s','nokia lumia 635',
       'htc one','samsung galaxy note','amazon fire phone') THEN e.user_id ELSE NULL END) AS phone,
        COUNT(DISTINCT CASE WHEN e.device IN ('ipad air','nexus 7','ipad mini','nexus 10','kindle fire','windows surface',
        'samsumg galaxy tablet') THEN e.user_id ELSE NULL END) AS tablet
  FROM tutorial.yammer_events e
 WHERE e.event_type = 'engagement'
   AND e.event_name = 'login'
 GROUP BY 1
 ORDER BY 1
LIMIT 100
```
![device_categories](device_category.PNG)
- `events` 테이블에 있는 장치들을 분류한 뒤 해당 값들을 `week`에 대해 집계한 결과다.
- 전반적으로 감소하는 추세지만, 특히 `phone`의 감소 추세를 볼 수 있다. 
- `engagement`에 대한 조사이므로, `phone`을 이용한 트래픽에 문제가 있음을 알 수 있다 : 이 시점에서 모바일 어플리케이션에 어떤 문제가 있는가를 주변인에게 물어볼 수 있을 것
- 또한 튕겨나간 사람들을 다시 유입할 요소를 생각할 수 있다 : 유저를 다시 유입하기 위해 안내 메일을 보내는 것도 방법이다. 

#### 4. 이메일 테이블 분석
```sql
SELECT DATE_TRUNC('week', occurred_at) AS week,
       COUNT(CASE WHEN e.action = 'sent_weekly_digest' THEN e.user_id ELSE NULL END) AS weekly_emails,
       COUNT(CASE WHEN e.action = 'sent_reengagement_email' THEN e.user_id ELSE NULL END) AS reengagement_emails,
       COUNT(CASE WHEN e.action = 'email_open' THEN e.user_id ELSE NULL END) AS email_opens,
       COUNT(CASE WHEN e.action = 'email_clickthrough' THEN e.user_id ELSE NULL END) AS email_clickthroughs
  FROM tutorial.yammer_emails e
 GROUP BY 1
 ORDER BY 1
```
![이메일](email_actions.PNG)
- `clickthrough`이 낮아졌음 : 해당 항목은 이메일의 링크를 클릭했느냐 여부였음 -> 해당 항목에 문제가 생겼음을 알 수 있다.

#### 5. 이메일 열기 & Clickthrough 분석
```sql
SELECT week,
       weekly_opens/CASE WHEN weekly_emails = 0 THEN 1 ELSE weekly_emails END::FLOAT AS weekly_open_rate,
       weekly_ctr/CASE WHEN weekly_opens = 0 THEN 1 ELSE weekly_opens END::FLOAT AS weekly_ctr,
       retain_opens/CASE WHEN retain_emails = 0 THEN 1 ELSE retain_emails END::FLOAT AS retain_open_rate,
       retain_ctr/CASE WHEN retain_opens = 0 THEN 1 ELSE retain_opens END::FLOAT AS retain_ctr
  FROM (
SELECT DATE_TRUNC('week',e1.occurred_at) AS week,
       COUNT(CASE WHEN e1.action = 'sent_weekly_digest' THEN e1.user_id ELSE NULL END) AS weekly_emails,
       COUNT(CASE WHEN e1.action = 'sent_weekly_digest' THEN e2.user_id ELSE NULL END) AS weekly_opens,
       COUNT(CASE WHEN e1.action = 'sent_weekly_digest' THEN e3.user_id ELSE NULL END) AS weekly_ctr,
       COUNT(CASE WHEN e1.action = 'sent_reengagement_email' THEN e1.user_id ELSE NULL END) AS retain_emails,
       COUNT(CASE WHEN e1.action = 'sent_reengagement_email' THEN e2.user_id ELSE NULL END) AS retain_opens,
       COUNT(CASE WHEN e1.action = 'sent_reengagement_email' THEN e3.user_id ELSE NULL END) AS retain_ctr
  FROM tutorial.yammer_emails e1
  LEFT JOIN tutorial.yammer_emails e2
    ON e2.occurred_at >= e1.occurred_at
   AND e2.occurred_at < e1.occurred_at + INTERVAL '5 MINUTE'
   AND e2.user_id = e1.user_id
   AND e2.action = 'email_open'
  LEFT JOIN tutorial.yammer_emails e3
    ON e3.occurred_at >= e2.occurred_at
   AND e3.occurred_at < e2.occurred_at + INTERVAL '5 MINUTE'
   AND e3.user_id = e2.user_id
   AND e3.action = 'email_clickthrough'
 WHERE e1.occurred_at >= '2014-06-01'
   AND e1.occurred_at < '2014-09-01'
   AND e1.action IN ('sent_weekly_digest','sent_reengagement_email')
 GROUP BY 1
       ) a
 ORDER BY 1
```
![열기_ct](open_ct_rate.PNG)
쿼리 분석  
1. 서브쿼리
- 이메일 액션은 4가지가 있음
  1. `clickthrough` : 이메일의 링크를 눌렀는가
  2. `open` : 이메일을 열었는가
  3. `sent_weekly_digest` : 매 주 보내는 광고메일 보냄
  4. `sent_reengagement_email` : 서비스 재이용 권유 메일 보냄
- 즉 `e2`는 이메일을 받고 5분 이내에 열었을 때 해당이고
- `e3`은 이메일을 열고 5분 내에 링크를 눌렀는가에 대한 정보임
- 서브쿼리는 매 주에 대해 각 메일 항목(`digest`, `reengagement`)에 대해 `이메일이 보내졌는가`, `이메일을 열었는가`, `이메일의 링크로 들어갔는가` 3가지, 즉 6가지 column을 만들었다.
2. 메인쿼리
- 각 메일의 종류에 대해 보내진 메일을 열었는가, 연 메일의 링크를 들어갔는가에 대한 비율을 뽑았음
- `open / CASE WHEN emails = 0 THEN 1 ELSE emails END` 문이 이해가 안 갔는데, `(다음 과정) / (이전 과정)` 식으로 연관성이 있기 때문에 위처럼 쿼리를 짠 거임(메일이 가지 않았는데 여는 게 가능하지 않기 때문임)

- 위 사항을 통해 `weekly_ctr`, 즉 다이제스트 메일의 링크와 관련된 문제가 있음을 알 수 있다. 유저 참여가 감소한 기간 동안 해당 지표도 같이 하락했기 때문.

- 이거 하면서 든 생각은 **5분의 인터벌은 너무 짧지 않냐**는 건데, 이게 회사의 업무 관련한 db라는 걸 생각하면 합당한 가정일지도 모르겠다. 회사 생활을 해봤어야 알죠


#### 맺음말
- 위 조사를 통해 모바일 참여와 다이제스트 메일에 문제가 있음을 알 수 있다.
- 분석가가 문제가 있을 수 있는 부분을 파악한 뒤 관련 부서에 알려 그들이 어떤 부분에 집중해야 하는지를 빠른 시간에 파악하도록 할 수 있다. 

-----------------------------

# 2. 검색 기능 이해
- `Yammer`에는 어떤 페이지든 헤더에 `Search Box`가 있음. 검색은 사람, 그룹, 대화에 대해 이루어짐.
  - 타이핑 중에도 항목에 따라 자동완성 항목들을 볼 수 있다. 이 자동완성 항목들은 분류되어 있음.
  - `View All Results`를 누르면 결과 페이지로 넘어감
  - 결과 페이지에서는 또 특정 그룹이나 날짜 별로 결과를 조회할 수 있는 기능이 있음
  
### 문제
- 검색에 쓰이는 시간이 잘 활용되는가
- 검색 작업을 해야 하는지 여부와 검색 작업을 한다면 어떻게 수정해야 하는가

### 방향 잡기
- 데이터를 보기 전, 유저가 검색과 어떻게 상호작용하는지 여러 가설을 세워볼 것
  - 검색의 목적?
  - 목적을 만족했는지는 어떻게 알 수 있을까?
  - 검색 경험을 어떻게 정량화할 수 있을까?
```
* 저번 예제와 다르게 이번에는 내 가정만 세우고 진행한 뒤 실습 정답에서 세운 과정과 비교하겠음
1. 검색의 목적
- 어떤 사람이나 그룹, 대화
- 사내 커뮤니티니까, 어떤 프로젝트 주제에 대한 이야기가 오갔을 수도 있겠다
- 어쨌든 회사와 관련된 뭔가를 찾고자 검색을 이용할 것임

2. 목적을 만족했는가 파악하기
- 일단 재검색을 배제하고 생각함 : 기준을 세우기 애매할 것 같음. 테이블을 보고 생각이 바뀔 수도 있겠지만.
- 가장 직접적으로 볼 수 있는 지표는 run 대비 search_click_X의 비율을 보는 것일 듯 : 결과가 만족스럽지 않다면 검색 결과에서 살펴보지도 않을 것 같기 때문임

3. 경험 정량화하기
- run -> search_click_X 가 실행되지 않은 케이스는 검색 목적을 만족하지 못했을 것
- autocomplete와 run을 합친 숫자 전체가 검색한 수가 될 듯 : autocomplete의 비율이 높을수록 
```
### 데이터
- `Users`, `Events`
- 이벤트 테이블에서 참고할 만한 것
    - `search_autocomplete` : 자동완성된 결과물을 유저가 클릭했을 때
    - `search_run` : 검색을 실행했고 결과 페이지를 봤을 때
    - `search_click_X` : 검색 결과 1~10 중 유저가 클릭한 숫자

### 추천 가설들
1. 유저들의 검색 경험이 일반적으로 좋은가 나쁜가
2. 검색할 가치가 있는가
3. 검색할 가치가 있다면 어느 부분을 개선해야 하는가

- 검색 상태를 설명하는 간단한 프레젠테이션을 작성하고 결과를 그래픽으로 표시하라. 수행할 조치가 있다면 권장할 준비가 되어 있어야 한다. 관련성이 있다고 판단되는 항목을 테스트하기 충분한 정보가 없다면 주의사항을 논하라

- 추천 기능이 기존 검색보다 개선되었는지를 이해하는 방법을 결정하라.

---------------------
## 실습 내용

- 사실 뭘 하느냐보다도 쿼리를 어떻게 짜느냐가 더 중요함. 계획이 있어도 쿼리를 못 짜면 말짱 도루묵이기 때문임
- 서브 쿼리를 여러 번 써야 할 거 같아서, 각 쿼리를 정리하면서 진행한다.

1. 유저 별로 연속된 행동 조사하기
```sql
-- 1) USER_ID 별로 액션 조회하기
SELECT u.user_id, 
       e.occurred_at,
       e.event_name
  FROM tutorial.yammer_events e
  JOIN tutorial.yammer_users u 
    ON e.user_id = u.user_id 
 WHERE event_name ILIKE 'search_%'
 ORDER BY 1, 2

-- 여기서 LEAD()함수를 넣고 싶지만, WINDOW FUNCTION은 WHERE 조건을 사용할 수 없다.
-- 이를 쓰기 위해선 위 쿼리로 테이블을 뽑아낸 뒤 상위 쿼리에서 이용하는 방법 밖에 없는 듯?

 -- 2) (1)을 서브쿼리로 해서 다음 시간의 행동을 같은 열에 집어넣기
    -- 참고) 위 상태에서 LEAD()함수를 사용한다고 하면 정상적으로 적용이 안된다.
    -- LEAD() 함수에는 WHERE 조건을 넣을 수 없기 때문이다. WHERE 조건이 적용되지 않은 보이지 않는 event_name의 시간이 불러와짐
SELECT sub.user_id as id,
       sub.occurred_at as time,
       CASE WHEN sub.user_id = LEAD(sub.user_id) OVER (ORDER BY 1, 2) THEN LEAD(sub.occurred_at) OVER (ORDER BY 1, 2) 
            ELSE NULL END AS next_action_time, -- 유저 아이디가 다음 칸과 다른 경우는 다음 시간을 빈 칸으로 두었음
      -- CASE WHEN sub.user_id = LEAD(sub.user_id) OVER (ORDER BY 1, 2) THEN LEAD(sub.occurred_at) OVER (ORDER BY 1, 2) - sub.occurred_at 
      --       ELSE NULL END AS time_difference,
       CASE WHEN LEAD(sub.occurred_at) OVER (ORDER BY 1, 2) - sub.occurred_at < INTERVAL '1 min'
             AND sub.user_id = LEAD(sub.user_id) OVER (ORDER BY 1, 2) 
             AND sub.event_name = 'search_run' AND LEAD(sub.event_name) OVER (ORDER BY 1, 2) ILIKE 'search_click_result_%' THEN NULL
            WHEN LEAD(sub.occurred_at) OVER (ORDER BY 1, 2) - sub.occurred_at < INTERVAL '1 min'
             AND sub.user_id = LEAD(sub.user_id) OVER (ORDER BY 1, 2) THEN 're_searched'
            ELSE NULL END AS re_searched,

        sub.event_name
  FROM (
     SELECT u.user_id, 
           e.occurred_at,
           e.event_name
      FROM tutorial.yammer_events e
      JOIN tutorial.yammer_users u 
        ON e.user_id = u.user_id 
     WHERE event_name ILIKE 'search_%'
    ORDER BY 1, 2 -- 이거 건드리면 결과 달라짐. 유지!
     ) sub
 ORDER BY 1, 2
/* 코멘트) 
CASE WHEN 조건문은 위부터 순차적으로 동작함(위에서 조건을 만족한 ROW는 밑 조건식에서 대상이 되지 않음)
그래서 search_run -> click_result 가 즉각적이었다면 이는 검색을 다시 한 게 아님(검색하고 결과를 클릭한 거니까) 
저 상황을 제외한 나머지 모든 조건은 검색 결과가 만족스럽지 못해 다시 검색한 것으로 간주했음
*/

 -- 3. 위 결과를 바탕으로 재검색 비율 뽑아내기
SELECT COUNT(CASE WHEN sub2.re_searched = 1 THEN 1 ELSE 0 END) AS total_searched_counts,
      COUNT(CASE WHEN sub2.re_searched = 1 THEN 1 ELSE NULL END) AS re_searched_counts,
      COUNT(CASE WHEN sub2.re_searched = 1 THEN 1 ELSE NULL END) / COUNT(CASE WHEN sub2.re_searched = 1 THEN 1 ELSE 0 END) :: float AS re_searched_rate
  FROM (
      SELECT sub.user_id as id,
       sub.occurred_at as time,
       CASE WHEN sub.user_id = LEAD(sub.user_id) OVER (ORDER BY 1, 2) THEN LEAD(sub.occurred_at) OVER (ORDER BY 1, 2) 
            ELSE NULL END AS next_action_time, -- 유저 아이디가 다음 칸과 다른 경우는 다음 시간을 빈 칸으로 두었음
      -- CASE WHEN sub.user_id = LEAD(sub.user_id) OVER (ORDER BY 1, 2) THEN LEAD(sub.occurred_at) OVER (ORDER BY 1, 2) - sub.occurred_at 
      --       ELSE NULL END AS time_difference,
       CASE WHEN LEAD(sub.occurred_at) OVER (ORDER BY 1, 2) - sub.occurred_at < INTERVAL '1 min'
             AND sub.user_id = LEAD(sub.user_id) OVER (ORDER BY 1, 2) 
             AND sub.event_name = 'search_run' AND LEAD(sub.event_name) OVER (ORDER BY 1, 2) ILIKE 'search_click_result_%' THEN 0
            WHEN LEAD(sub.occurred_at) OVER (ORDER BY 1, 2) - sub.occurred_at < INTERVAL '1 min'
             AND sub.user_id = LEAD(sub.user_id) OVER (ORDER BY 1, 2) THEN 1
            ELSE 0 END AS re_searched,
      -- CASE WHEN sub.event_name = 'search_run' AND LEAD(sub.event_name) OVER (ORDER BY 1, 2) ILIKE 'search_click_result_%' 
      --       THEN 'not_re_searched' ELSE NULL END AS not_re_searched,
        sub.event_name
            FROM (
               SELECT u.user_id, 
                     e.occurred_at,
                     e.event_name
                FROM tutorial.yammer_events e
                JOIN tutorial.yammer_users u 
                  ON e.user_id = u.user_id 
               WHERE event_name ILIKE 'search_%'
              ORDER BY 1, 2 -- 이거 건드리면 결과 달라지네 ㄷㄷ
               ) sub
           ORDER BY 1, 2
  ) sub2

  -- 코멘트 : 전체 값 중 특정 값이 차지하는 비율을 알려면 COUNT(WHEN ~~ THEN value ELSE NULL) / COUNT (WHEN ~ THEN value1 ELSE val2) 로 넣으면 된다. COUNT함수는 NULL 값을 세지 않는 특성이 있기 때문이다.
```
- 마지막 쿼리 결과 전체 검색 횟수가 40611회, 재검색 횟수가 25470회로 `1분 이내로 재검색한 비율`이 62.72%가 나왔다.
- 기기나 국가 등 조건을 다양하게 줄 수 있을 것 같은데 일단 보류함
- 이거 한다고 2시간 반 걸림
- 여러 시행착오가 있다 : 주석으로 달아놓았다

----------------------------------
## 튜토리얼에서 제공하는 Answer

### 1. 가설 설정하기
- **문제를 간단하고 정확하게** 잡는 게 나중에 시간 절약에 도움이 된다. 
- 검색은 사람들이 원하는 것을 쉽게 찾도록 하여 시간을 아끼고 속도를 향상시켜준다.
  
살펴볼 만한 여러 가능성들
```
1. 누가 사용하는가
2. 검색 빈도
  - 많이 검색되는 단어가 가치가 높을 가능성이 높음
  - 단 짧은 시간에 반복된다면 원하는 바를 찾지 못해 검색어를 다듬고 있을 확률이 높다.
3. 반복되는 단어들 
  - 단어들의 유사성을 비교하는 것도 방법.
  - 난이도가 올라가니 일단은 배제
4. Clickthrough(ct)
  - 하나의 검색 결과에서 많은 링크를 들어간다면 결과가 만족스럽지 못할 확률이 높다.
    - 그렇다고 하나만 클릭한 게 검색 결과가 좋다는 얘기도 아니다 : 검색어를 수정해서 재검색한다면, 이전의 검색 결과는 좋지 않았다는 뜻이 된다.
  - 그러나 클릭해서 들어간(Clickthrough, ct) 행위는 검색 결과가 좋았는지를 판별하는 좋은 지표가 된다. 
5. 자동완성 ct
  - 검색 결과 클릭과 별도로 측정되어야 한다
```

### 2. 검색 상태
- 세션(어떤 정보를 찾기 위한 일련의 탐색 과정) 별로 검색을 이해하기 위해, 이 솔루션은 어떤 두 이벤트가 `10분` 내에 사용자의 액션이 있었다면 하나의 세션으로 간주한다.
- 즉 `10분` 동안 사용자의 액션이 없었다면 별도의 세션으로 간주한다. 
- 즉 10분 내에 정보를 탐색하는 횟수가 검색 알고리즘의 성능을 나타내는 지표가 될 수 있다는 것이다.

1. 기간에 따른 자동완성(autocomplete)과 검색(run) 기능 수행 숫자
```sql
SELECT DATE_TRUNC('week',z.session_start) AS week,
       COUNT(*) AS sessions,
       COUNT(CASE WHEN z.autocompletes > 0 THEN z.session ELSE NULL END) AS with_autocompletes,
       COUNT(CASE WHEN z.runs > 0 THEN z.session ELSE NULL END) AS with_runs
  FROM (
SELECT x.session_start,
       x.session,
       x.user_id,
       COUNT(CASE WHEN x.event_name = 'search_autocomplete' THEN x.user_id ELSE NULL END) AS autocompletes,
       COUNT(CASE WHEN x.event_name = 'search_run' THEN x.user_id ELSE NULL END) AS runs,
       COUNT(CASE WHEN x.event_name LIKE 'search_click_%' THEN x.user_id ELSE NULL END) AS clicks
  FROM (
SELECT e.*,
       session.session,
       session.session_start
  FROM tutorial.yammer_events e
  LEFT JOIN (
       SELECT user_id,
              session,
              MIN(occurred_at) AS session_start,
              MAX(occurred_at) AS session_end
         FROM (
              SELECT bounds.*,
              		    CASE WHEN last_event >= INTERVAL '10 MINUTE' THEN id
              		         WHEN last_event IS NULL THEN id
              		         ELSE LAG(id,1) OVER (PARTITION BY user_id ORDER BY occurred_at) END AS session
                FROM (
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
                     ) bounds
               WHERE last_event >= INTERVAL '10 MINUTE'
                  OR next_event >= INTERVAL '10 MINUTE'
               	 OR last_event IS NULL
              	 	 OR next_event IS NULL   
              ) final
        GROUP BY 1,2
       ) session
    ON e.user_id = session.user_id
   AND e.occurred_at >= session.session_start
   AND e.occurred_at <= session.session_end
 WHERE e.event_type = 'engagement'
       ) x
 GROUP BY 1,2,3
       ) z
 GROUP BY 1
 ORDER BY 1
 ```
![run_ac](run_ac.PNG)
- (느낀 점) : 일단 서브쿼리 빈도는 걱정할 필요는 없겠다. 예제는 5번 썼는데 ㅎㅎ
- 일단 세션을 어떻게 정의했는가부터 살펴보자.
  - 가장 안쪽 쿼리에서 다음 이벤트와 이전 이벤트를 정의했음
    - 독립적인 ROW_NUMBER()를 `ID`로 지목하기도 했음
  - `이전 이벤트가 10분 이후` OR `이전 이벤트가 없음` 이면 `세션 = ID`이고, 이외에는 이전 `ID`를 가져오는 게 세션 개념임 -> 따라서 세션은 연속적인 숫자가 아님
- 그 위에서는 유저 아이디와 세션을 `GROUP BY`한 뒤, 세션 내의 `occurred_at`의 최솟값을 세션 시작점, 최댓값을 세션 종료 점으로 설정함
- 위 테이블을 `yammer_events` 테이블을 `FROM`이라고 했을 때 `LEFT JOIN`함
  - `ON` 조건 : event의 사건 발생 시점이 세션의 시작점보다 크고 끝점보다 작음
- 위 테이블을 다시 서브쿼리로 넣어 세션 시작 시간, 세션 ID, 유저 ID 및 `search_autocomplete, search_run, search_click_%`에 대한 `count` 함수를 넣음
- 마지막으로, 다시 위 테이블을 서브쿼리로 넣고
  - 매 주에 대해
    - 세션의 총 숫자
    - autocomplete > 0 인 세션들의 수
    - runs > 0 인 세션들의 수
  - 를 집계함.

세션의 25%를 autocomplete가 차지한다. run은 8% 정도만을 차지한다. 
```sql
SELECT DATE_TRUNC('week',z.session_start) AS week, 
       COUNT(CASE WHEN z.autocompletes > 0 THEN z.session ELSE NULL END)/COUNT(*)::FLOAT AS with_autocompletes,
       COUNT(CASE WHEN z.runs > 0 THEN z.session ELSE NULL END)/COUNT(*)::FLOAT AS with_runs
-- 이하 동일 : 가장 바깥 부분만 가져옴
```
![pct](run_ac_pct.PNG)
- 쿼리를 보면 알겠지만 얘는 `COUNT(*)`도 잘 작동한다. ;;
- 결과 ) `AUTOCOMPLETE`의 사용 비중이 높기 때문에 해당 기능을 개선할 필요가 있을 것이다.

1. `Autocomplete`은 세션 당 2회 이상 실행된다.
```sql
SELECT autocompletes,
       COUNT(*) AS sessions
  FROM (
SELECT x.session_start,
       x.session,
       x.user_id,
       COUNT(CASE WHEN x.event_name = 'search_autocomplete' THEN x.user_id ELSE NULL END) AS autocompletes,
       COUNT(CASE WHEN x.event_name = 'search_run' THEN x.user_id ELSE NULL END) AS runs,
       COUNT(CASE WHEN x.event_name LIKE 'search_click_%' THEN x.user_id ELSE NULL END) AS clicks
  FROM (
SELECT e.*,
       session.session,
       session.session_start
  FROM tutorial.yammer_events e
  LEFT JOIN (
       SELECT user_id,
              session,
              MIN(occurred_at) AS session_start,
              MAX(occurred_at) AS session_end
         FROM (
              SELECT bounds.*,
              		    CASE WHEN last_event >= INTERVAL '10 MINUTE' THEN id
              		         WHEN last_event IS NULL THEN id
              		         ELSE LAG(id,1) OVER (PARTITION BY user_id ORDER BY occurred_at) END AS session
                FROM (
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
                     ) bounds
               WHERE last_event >= INTERVAL '10 MINUTE'
                  OR next_event >= INTERVAL '10 MINUTE'
               	 OR last_event IS NULL
              	 	 OR next_event IS NULL   
              ) final
        GROUP BY 1,2
       ) session
    ON e.user_id = session.user_id
   AND e.occurred_at >= session.session_start
   AND e.occurred_at <= session.session_end
 WHERE e.event_type = 'engagement'
       ) x
 GROUP BY 1,2,3
       ) z
 WHERE autocompletes > 0
 GROUP BY 1
 ORDER BY 1
 ```
 ![이미지](ac_in_sess.PNG)
![이미지2](sess_runs.PNG)
- 자동완성, 일반 검색 모두 여러 번 실행됨
- 일반 검색의 경우 여러 번 검색하는 빈도가 더 높음 
  - 이는 검색 결과가 매우 좋지 않거나
  - 항상 검색을 이용하는 매우 적은 그룹의 유저들이 있음을 의미함
  
3. 일반 검색의 성능 더 파고 들기

```sql
SELECT clicks,
       COUNT(*) AS sessions
  FROM (
        -- ...(세션 추려내는 쿼리 : 맨 위 참조)
       ) z
 WHERE runs > 0
 GROUP BY 1
 ORDER BY 1
```
![이미지](click_per_search.PNG)
- 일반 검색이 있는 세션 중 어떤 결과도 클릭하지 않은 세션이 매우 많음 : 이는 일반 검색의 성능이 매우 좋지 않음을 시사함
```sql
SELECT runs,
       AVG(clicks)::FLOAT AS average_clicks
  FROM() -- ...
 WHERE runs > 0
 GROUP BY 1
 ORDER BY 1
```
![이미지](avg_click_per_search_counts.PNG)
- 심지어 일반 검색을 더 많이 했더라도, 더 많은 클릭으로 이어지진 않음

4. 검색 순위와 클릭 빈도
```sql
SELECT TRIM('search_click_result_' FROM event_name)::INT AS search_result,
       COUNT(*) AS clicks
  FROM tutorial.yammer_events
 WHERE event_name LIKE 'search_click_%'
 GROUP BY 1
 ORDER BY 1
```
![이미지](click_search_order.PNG)
- 좋은 검색 결과는 앞쪽 3개에 클릭 수가 몰려 있어야 함
- 비교적 균등하게 분포되었기 때문에 이 정렬 순서 또한 수정될 필요가 있음

5. 일반 검색 재사용률
```sql
SELECT searches,
       COUNT(*) AS users
  FROM (
SELECT user_id,
       COUNT(*) AS searches 
  FROM (
SELECT x.session_start, -- 세션 시작 시간
       x.session, -- 세션 고유 번호
       x.user_id, -- 유저 고유 아이디
       x.first_search, -- 유저 별 첫 탐색 시간
       COUNT(CASE WHEN x.event_name = 'search_run' THEN x.user_id ELSE NULL END) AS runs -- FROM 조건에 1개월 이내에 다시 실행한 세션을 탐색함
       -- 그 중에 RUN이 있으면 ID를 살림
  FROM (
SELECT e.*,
       first.first_search, -- 첫 탐색 날짜
       session.session, -- 세션 고유 번호
       session.session_start -- 세션 시작 시간
  FROM tutorial.yammer_events e
  JOIN (
       SELECT user_id,
              MIN(occurred_at) AS first_search
         FROM tutorial.yammer_events
        WHERE event_name = 'search_run'
        GROUP BY 1
       ) first
    ON first.user_id = e.user_id
   AND first.first_search <= '2014-08-01'
  LEFT JOIN (
      -- 세션 테이블까지는 동일함
       ) session
    ON session.user_id = e.user_id
   AND session.session_start <= e.occurred_at
   AND session.session_end >= e.occurred_at
   AND session.session_start <= first.first_search + INTERVAL '30 DAY'
 WHERE e.event_type = 'engagement'
       ) x -- 각 세션 번호와 세션의 시작점, 첫 검색일 등..
 GROUP BY 1,2,3,4
       ) z
 WHERE z.runs > 0
 GROUP BY 1
       ) z
 GROUP BY 1
 ORDER BY 1
LIMIT 100
```
![이미지](search_again_in_month.PNG)

- 그 결과, 일반 검색의 재사용률(1개월 이내)은 현저히 낮다.
- 1개월 이내에 `RUN` 쿼리를 몇 번 재사용했는가에 대한 그래프임

- `AUTOCOMPLETE`과 비교
```sql
-- 쿼리 : 위 쿼리에서 'run -> autocomplete'으로 바꿔주기만 하면 됨
```
![이미지](ac_again.PNG)
- 한편 `AUTOCOMPLETE`의 1개월 이내 재사용 빈도는 높음

맺음)
자동 완성 대비 일반 검색에 문제가 있고, 그 중 하나는 검색 순위에 있다. 검색 순위 알고리즘을 변경하는 것이 도움이 될 수 있다. 물론 이 예제는 수많은 답안 중 하나이고, 중요한 건 전체 검색 결과를 개선하는 데 초점을 맞춰야 한다는 것이다.

----------

# 3. A/B 테스트
## A/B 테스트란?
- 실사용자를 대상으로 대조군 / 실험군으로 나눠 UI나 알고리즘의 효과를 비교하는 방법론
  - 오바마 캠프의 사이트 UI에 대한 A/B 테스트 실험 결과 기부 전환율을 49%, 이메일 수집률을 161% 증가시켰다.
- 사용자를 분리하는 방법에는 3가지가 있다.
  1. 노출 빈도 분산 방식
    - 페이지 렌더링 시 비율에 따라 A/B를 다르게 노출 시킴
    - `사용자 분산과의 차이` : 노출 빈도 분산 방식은 한 유저가 A와 B를 모두 맞닥뜨릴 수 있음
    - 가장 통계적 유의성이 높지만, UI/UX 분야에서는 사용자에게 혼란을 줄 수 있다.
    - 알고리즘 테스트에 적합
  2. 사용자 분산 방식
    - 사용자를 A/B로 나눠 고정적으로 다른 변수를 노출 시킴
    - UI/UX 테스트에 적합하지만 특정 Heavy User에 의해 결과값이 왜곡될 수 있다.
  3. 시간 분할 방식
    - 초/분 단위로 시간대를 분산해 A, B를 노출시킴
    - 보안 / 설계 상 문제로 1, 2가 어려울 때 상대적으로 쉽게 활용할 수 있다.
- A/B 테스트의 결과 신뢰 방법
  1. AA 테스트
  - 분산된 트래픽에 대해 동일한 Variation(AA의 의미)을 보여주고 차이가 없을 때 AB 테스트를 수행한다는 뜻
  2. P-value 분석
  - p-value = `유의 확률`
  - H0이 맞다고 가정할 때 얻은 결과보다 극단적인 결과가 관측될 확률.
  - p-value가 특정 값(`0.05`, `0.01`을 보통 설정)보다 작을 경우 H0을 기각하는 것이 관례인데, 위키피디아에 따르면 기준을 `0.005`로 바꿔야 한다는 얘기도 있다고 함

## 케이스 설명
- 메시지를 타이핑하는 `Publisher` 기능에 핵심
- 6월 1일부터 30일까지 A/B 테스트를 수행
- `Control Group` : 옛날 버전 / `Treatment Group` : 최근 버전 2가지로 사용자들을 분산시킴.
- 7월 1일에 `A/B` 테스트의 결과를 살펴봄.
  - `Treatment Group`이 50% 높게 나옴
- `t-test(t검정)`
  - 모집단의 평균, 표준편차를 모를 때 표본에서 추정된 분산이나 표준편차를 활용하여 검정함
  - 통계는 기술통계(`Descriptive Statistics`)와 추리통계(`Inferential Statistics`)로 나뉜다. 
    - 기술통계는 수집된 데이터의 특성을 요약, 제시
    - 추리통계는 다시 모집단의 분포를 가정하지 않는 `비모수적 통계검정`과 모집단의 특성을 가정하는 `모수적 통계검정`으로 세분화된다.
      - `t-검정`은 모수적 통계 검정임
  - 가정) 모집단의 분포는 정규분포를 따른다, 등분산성의 가정이 충족된다, 종속변수는 양적변수다.
  - 가설) 
    - H0 : 두 집단(모-표본) 간 평균 차이가 없다
    - H1 : 두 집단 간 평균이 유의미한 차이가 있다
  - 종류)
    - 1. 단일표본 T검정 : 추출한 표본의 평균을 연구자가 이론 OR 경험적 근거에 의해 설정한 특정한 수와 비교하여 검정한다.
    - 2. 두 독립표본 T검정 : 모집단에서 2개의 표본을 추출해 평균을 비교하고, 이를 통해 모집단 간 평균에 차이가 있는지를 확인한다.
      - 이 때 두 표본이 독립이라면 `두 독립표본 T검정`, 종속이라면 `두 종속표본 T검정`이다.
        - 독립표본 : 서로 다른 두 지역의 학업성취도, 우을증에 대한 두 약의 효과 비교
        - 종속표본 : 특정 치료, 약물 투여를 기준으로 한 사전/사후 검사

## 접근하기
- 왜 저런 결과가 나타났는가를 가설을 세워보라 : 보통 50%의 상승은 겁나 드문 일이다.
- 총 4개의 테이블을 사용한다
  - `Users`
  - `Events`
  - `Experiments`
    - `experiment` : 실험의 이름 : 실험 동안 제품의 무엇이 바뀌었는가를 볼 수 있음
    - `expermient group` : `test_group`은 새로운 버전, `control_group`은 옛날 버전을 사용했음.
  - `Normal Distribution`
    - `score` : `Z-score` - 값이 0 이상인 테이블에만 들어가 있다. 
    - `value` : `z-score`보다 작은 값의 영역

## 결과 검증하기
1. 이 테스트가 정상적으로 수행되었는가? `lift`와 `p-value` 값이 올바르게 측정 되었는가? 위 코드부터 시작하는 게 좋다.
2. 다른 지표도 살펴보자
3. 데이터가 올바른가도 살펴봐야 한다 : 결과가 기록되는 방식, 사용자를 실험/대조군으로 취급하는 방식 등 -> 뭔가 부정확하다면 문제를 수정할 단계를 정할 수 있다.
4. 당신의 결론으로 인한 추천을 하나 해볼 것 : 새로운 퍼블리셔가 모두에게 공개되어야 하는가? 재실험이 필요한가? 만약 그렇다면 뭐가 달라져야 하는가? 아예 폐기해야 하는가?
---------------
## 직접 진행
1. 일단 통계치를 뽑는 것부터 시작하자.

------------------
## 답지