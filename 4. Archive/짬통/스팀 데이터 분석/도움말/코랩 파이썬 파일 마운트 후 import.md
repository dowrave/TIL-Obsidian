일반적으로 어떤 파이썬 파일에서 작업할 때 `import xxx`만 입력해줘도 현재 작업중인 폴더와 같은 폴더에 있는 `xxx.py`를 import할 수 있음.
근데 코랩은 현재 작업중인 파일의 위치를 정확히 모른다. 파일을 들어가서 켜지 않잖음? 그런데 어떤 파이썬 파일을 mount한 다음, 이용하고 싶을 때는 이런 방식을 쓴다.

```python
import sys
sys.path.append('마운트한 파일을 포함한폴더')

import xxx
```


- 참고) 아래랑은 다른 개념임
```python
import os
os.getcwd('원하는 작업 경로')
```
- `import`랑 `cmd`에서의 작업 방식이 다르다? 이런 느낌이다.

- 대신 이렇게는 되겠지
```python
import os
os.getcwd('마운트 파일 포함한 폴더')

import sys
sys.path.append('.') # 위에서 작업 경로를 바꿔줬으니, 여기선 현재 경로가 됨

import xxx
```
