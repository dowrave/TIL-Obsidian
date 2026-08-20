
- CORS 이슈는 [[11. axios를 이용한 HTTP 통신#크로스 오리진 문제 해결 방법#CORS|CORS]]에서 다룬 적이 있으니 내용에 대한 자세한 언급은 생략한다.
---

### 1. CORS 설치
```sh
pip install django-cors-headers
```

### 2. settings.py
- `settings.py`
```python
INSTALLED_APPS = [
	...
	, 'corsheaders', # CORS (크로스오리진 설정)
]

MIDDLEWARE = [
    'corsheaders.middleware.CorsMiddleware',
]

# 크로스 오리진 설정
CORS_ALLOWED_ORIGINS = [
    '프론트엔드 서버 주소'
]
```