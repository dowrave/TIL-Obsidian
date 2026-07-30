
## 1. 외래키 출력 설정

```python
    category = serializers.CharField()
    section = serializers.CharField()
    subsection = serializers.CharField()
```
> 이 3개의 필드는 이용하려는 모델에서 외래키 관계이다.

1. 기본적으로 설정하지 않으면, 이 3개의 필드는 숫자로 나온다.
2. 위처럼 지정하면, 해당 모델에서 `__str__()`메서드로 정의된 것이 출력된다. 대부분 `name`으로 지정했기 때문에 `name`이 나옴.


## 2. 이용할 필드, 제외할 필드
```python
    class Meta:
        model = Post
        # fields = '__all__'
        exclude = ['id']
```
> 어떤 필드들을 사용할지는 class Meta에서 정한다.

1. 모든 필드를 사용하겠다면 `fields = '__all__'`을 쓰면 된다.
2. `update` 등 일부 필드만을 수정하는 경우, 굳이 모든 필드를 포함할 필요는 없다. 