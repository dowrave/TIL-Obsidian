```python
class ReviewAlias(models.Model):
    review = models.ForeignKey(
        Review,
        on_delete = models.CASCADE, # 리뷰가 삭제되면 별칭도 삭제
        related_name='aliases' # Review 모델에서 .aliases로 접근 가능
    )
```
> `Review` 모델에는 `aliases`라는 필드가 없음에도 검색 쿼리 등에서 `aliases`라는 필드를 이용할 수 있게 된다고 한다. 무슨 원리일까?

1. **정방향 참조** : 위 모델에서는 `review`라는 외래키 필드를 갖고 있기 때문에 `.review`로 매우 쉽게 접근할 수 있다. 이런 식으로 **갖고 있는 필드를 이용해 접근**하는 걸 정방향 참조라고 한다.

2. **역방향 참조**
	- `Review` 모델에서는 `ReviewAlias`라는 필드를 직접 갖고 있지 않다.
	- 하지만 "이 객체에 연결된 모든 외래키 객체들을 가져오고 싶다"라는 요구는 매우 흔하다.
	- `Django ORM`은 `1 -> N`으로의 접근을 가능하게 해준다. 이걸 `역방향 참조, 역참조`라고 한다.

## Django 내부에서 일어나는 일

`review_object.aliases`에 접근하는 순간 일어나는 일이다.

1. `Manager` 객체 생성
- `ReviewAlias` 모델에 대한 특별한 `Manager` 객체를 동적으로 생성한다. 이는 "`나를 호출한 review_object`와 연결된 것만 찾아야 한다"는 조건을 내부적으로 갖고 있다.

2. SQL 쿼리 변환
- `review_object.aliases.all()`이라는 코드는 아래와 유사한 SQL 쿼리로 변환된다.
```sql
SELECT * FROM reviews_reviewalias WHERE review_id = [review_object의 id]
```
> `review_alias` 테이블에서 `review_id`가 현재 `Review` 객체의 `id`와 일치하는 모든 레코드를 가져오는 쿼리를 실행한다.

3. 추상화 : `Django`는 위 과정을 거쳐 `review.aliases`로 다대다 필드가 원래부터 존재하는 것처럼 보이게 만든다. 

> 즉 **해당 테이블에 필드가 존재하는 것처럼 추상화되어 있지만, 실질적으로는 별도의 쿼리가 실행되는 개념**임. 복잡하게 생각할 것 없다. 

4. 검색 기능 등에서
```python
filter_backends = [SearchFilter] 
search_fields = [
	'title', 
	'aliases__name'
] 
```

위 처럼 사용하게 된다. 이는
1. `serach_fields`에 `aliases__name`이 포함된 걸 보면 "`Review`의 `aliases`라는 역참조 관계를 통해 `ReviewAlias`로 넘어가서 `name` 필드를 검색해야겠다"라고 인식한다.
2. 내부적으로 **`LEFT JOIN`을 사용, 두 테이블을 연결해 `title`이나 `aliases.name`에 검색어가 포함된 결과를 찾는 SQL 쿼리를 생성**한다. 



## 참고 : related_name
- `Django`는 역참조를 위해 기본적으로 `[모델이름_소문자]_set`이라는 이름을 자동으로 만든다. 위처럼 설정하지 않으면 `review_object.reviewalias_set.all()`처럼 접근해야 하는데 이를 `review_object.aliases.all()`로 접근할 수 있게 만든 것.