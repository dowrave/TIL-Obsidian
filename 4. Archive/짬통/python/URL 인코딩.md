### 왜 필요한가?
- 영어, 숫자를 제외한 나머지 문자를 16진수로 인코딩함
- GET으로 HTTP 요청 시 쿼리 파라미터가 붙는데, **URL은 ASCII 코드 값만 사용된다.** 한글이 포함된다면 URL은 그대로 인식할 수 없기 때문에 미리 인코딩하는 게 좋음.

### 코드 
#### 1. 간단하게 바로
```python
from urllib import parse
keyword = parse.quote('불곰') # 이 자체가 url encoding이 되는 거임


```
---
#### 2.
```PYTHON
from urllib import parse

url = parse.urlparse('https://brownbear:123@127.0.0.1:8080/path;params?name=불곰&params=123#id1')

print(url)
print(url.scheme)
```
- `url`은 `urllib.parse.ParseResult` 타입으로 반환되며, `url`은 `scheme` 메서드로 다른 메서드로 접근할 수 있는 `key, value` 쌍들을 반환한다.
- 이 중 **`query` 메서드는 `?` 뒤에 있는 `key=value` 값들을 가지고 있음**

#### 쿼리 스트링 분리
```python
query = parse.parse_qs(url.query) # {'name': ['불곰'], 'params': ['123']}
query = parse.parse_qsl(url.query) # [('name', '불곰'), ('params', '123')]
```
- `parse_qs`는 `dict` 형태로 쿼리를 분리하고, `parse_qsl`은 `list` 형태로 쿼리를 분리한다. 

#### url 인코딩
```python
# 분리된 value들을 url 인코딩한다
result = parse.urlencode(query,
						doseq = False)
```
