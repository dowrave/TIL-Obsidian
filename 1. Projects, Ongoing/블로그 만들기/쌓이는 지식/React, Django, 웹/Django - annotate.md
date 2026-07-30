```python
daily_activity = Post.objects.filter(category = category_obj) \
								.filter(created_at__year = year) \
								.annotate(day=ExtractDay('created_at')) \
```
> 여기서 `annotate`의 역할은, `created_at`에서 뽑아낸 `day`라는 값을 `day`라는 필드에 지정하는 것이다.

- 즉, `annotate`는 **새로운 필드를 일시적으로 만드는 것**임. 
- 일시적이라는 점에서는 서브쿼리와 비슷한데, `group by`의 개념과 더 유사하다고 함.