- 기본 라이브러리가 아님
## 1. 인코딩 자동 감지
- 파일의 인코딩을 자동으로 감지하는 기능이 있다
```python
import chardet

with open('file.csv', 'rb') as f:
	result = chardet.detect(f.read())
# result는 dict 형태로 주어짐
# 예시 :  {'encoding': 'EUC-KR', 'confidence': 0.99, 'language': 'Korean'}
```

- 이를 이용해 **판다스에서 `pd.read_csv(encoding = )`에 줄 값을 자동으로 구할 수 있음** (`result[encoding]` 형태로 인자를 주면 됨)

