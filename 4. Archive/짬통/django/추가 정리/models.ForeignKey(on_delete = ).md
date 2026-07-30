```python
class X(models.Model):
	fields = models.Foreignkey(another_table, on_delete = ...)
```

- 위 예문에서 `on_delete`에 올 수 있는 옵션에 대한 설명이다. `참조무결성`이라는 게 있단다.
> **유저 A가 작성한 게시글 a가 있다**고 하자.  유저 A가 탈퇴했을 때 옵션에 따라 일어나는 일을 적는다.
## 1. models.CASCADE
- 다른 테이블에서 이 필드가 참조하는 레코드가 삭제되면, 이 테이블의 레코드도 같이 삭제된다.
> - 유저 A가 회원탈퇴를 하면 게시글 a도 같이 사라지게 된다.


## 2. models.PROTECT
- 이 필드가 참조하는 해당 테이블의 레코드를 삭제하지 못하게 protected error를 발생시킨다.
> - 유저 A가 회원탈퇴를 시도하면, protected error가 발생한다. 유저 A는 모든 게시글을 직접 지워야 탈퇴할 수 있다.

## 3. models.SET_NULL
- `null = True`일 때만 가능하며, 이 속성을 NULL로 바꾼다.
> - 유저 A가 회원탈퇴하면, 게시글 a의 작성자는 null이 된다.

## 4. models.SET_DEFAULT
- `default`값이 있을 때만 가능하며, 이 속성을 `default`로 바꾼다.
> - 유저 A가 회원탈퇴하면, 게시글 a의 작성자는 사전에 정의한 `default`값이 된다.

## 5. models.SET()
- 외래키 값이 삭제될 때, 그 값을 SET에 설정된 함수로 바꿔준다.

## 6. models.DO_NOTHING
- 말 그대로 삭제되더라도 아무 행동도 취하지 않는다. 
- `참조무결성`을 해칠 여지가 있다.

