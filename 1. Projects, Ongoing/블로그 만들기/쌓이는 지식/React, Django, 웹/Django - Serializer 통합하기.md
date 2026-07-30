- Post 모델에 대한 Serializer를 기존엔 List와 Detail에 따로 구현했는데, 이들을 통합했다.

## 통합 후
```python
class PostSerializer(serializers.ModelSerializer):
    class Meta:
        model = Post
        fields = '__all__' # 필수) 밑에서 동적으로 지정해줘도 여기서 지정해줘야 함

    def get_fields(self):
        fields = super().get_fields()

        # urls.py에 name으로 지정한 문자열 매칭을 확인한다.
        if 'PostList' in self.context['request'].resolver_match.url_name:
            # List 필드 선택
            fields = {
                'content': serializers.SerializerMethodField(),
                'subsection': serializers.CharField(source='get_subsection_as_string'),
                'created_at': serializers.DateTimeField(format='%Y/%m/%d %H:%M'),
            }
        elif 'PostDetail' in self.context['request'].resolver_match.url_name:
            fields = {
                'author': serializers.CharField(source='get_author_as_string'),
                'category': serializers.CharField(source='get_category_as_string'),
                'section': serializers.CharField(source='get_section_as_string'),
                'subsection': serializers.CharField(source='get_subsection_as_string'),
                'title': serializers.CharField(),
                'content': serializers.CharField(),
                'created_at': serializers.DateTimeField(format='%Y/%m/%d %H:%M'),
            }
        
        return fields
    
    def get_content(self, obj):
        if 'PostList' in self.context['request'].resolver_match.url_name:
            return obj.content[:100] # 리스트의 경우 내용 100글자로 제한
        return obj.content
```
- 이렇게 구성할 경우, `self.context['request']`를 넣어주기 위해 `views.py`에서 키값을 넣어줘야 한다.
```python
# ListView
        serializer = PostSerializer(posts, many = True, context={'request' : request})

# DetailView
        serializer = PostSerializer(post, context={'request' : request})
```

---

## 통합 전
```python
class PostListSerializer(serializers.ModelSerializer):

    # List에선 100글자까지만 보여준다
    content = serializers.SerializerMethodField() 

    # 없으면 각각 숫자로만 표시된다
    subsection = serializers.CharField(source='get_subsection_as_string')

    # 시간 가공
    created_at = serializers.DateTimeField(format = '%Y/%m/%d %H:%M')

    class Meta:
        model = Post
        fields = ['id', 'subsection', 'title', 'created_at', 'content']

    def get_content(self, obj):
        # 내용 중 100글자까지만 보여주겠다
        return obj.content[:100]
    
class PostDetailSerializer(serializers.ModelSerializer):

    # slug = serializers.CharField(source='get_slug_as_string')
    author = serializers.CharField(source='get_author_as_string')
    category = serializers.CharField(source='get_category_as_string')
    section = serializers.CharField(source='get_section_as_string')
    subsection = serializers.CharField(source='get_subsection_as_string')

    # 시간 가공
    created_at = serializers.DateTimeField(format = '%Y/%m/%d %H:%M')

    class Meta:
        model = Post
        fields = '__all__'
```