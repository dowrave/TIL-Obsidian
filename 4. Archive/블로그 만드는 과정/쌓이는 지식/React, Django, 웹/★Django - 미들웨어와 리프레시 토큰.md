- [[★복합 - 리프레시 토큰 안전하게 보관하기]]에서는 프론트엔드에서 `App.tsx`에 새로고침 시 리프레시 토큰이 유효하다면 백엔드의 뷰로 접근하여 닉네임과 아이디를 받아오는 로직을 썼었다.
- `Django`에는 `MiddleWare`라는 개념이 있는데, `View`를 실행하기 전의 `request`나 실행한 후의 `response`에 대한 처리를 수행할 수 있다. 이를 이용하면 굳이 별도의 URL로 접근할 필요 없이 모든 요청/응답에서 로그인 상태를 유지하기 위한 로직을 구현할 수 있다고 생각했고, 여기엔 이를 정리해뒀다.


---
- 로그인 로직 
	- 프론트 : 아이디 + 비밀번호 백엔드에 전달
	- 백엔드 : 아이디 + 비밀번호 유효 시 프론트엔드에 `http only 쿠키` 형식으로 **리프레시 토큰과 액세스 토큰을 반환** 

- 이는 이전에 진행했던 과정이고, 여기서는 로그인된 상태(= 쿠키에 토큰이 포함된 상태)라면 리프레시 토큰이 유효한 경우, 요청이 들어온 경우 액세스 토큰을 자동으로 재발급하는 과정을 정리한다.
- 추가로 헤더에 유저의 아이디와 닉네임도 같이 반환한다.

## 백엔드
- 내 경우  `backend`가 전체 프로젝트 폴더이며, `backend/backend`에 `settings.py`를 비롯한 기본적인 파일들이 있다. 여기에 `middleware.py`를 작성했음.
```python
# backend/backend/middleware.py
from django.contrib.auth import get_user_model
from rest_framework_simplejwt.authentication import JWTAuthentication
from rest_framework_simplejwt.exceptions import InvalidToken, AuthenticationFailed
from rest_framework import status
from rest_framework.response import Response
from rest_framework_simplejwt.tokens import RefreshToken, AccessToken
from rest_framework_simplejwt.exceptions import InvalidToken, TokenError

User = get_user_model()

# 미들웨어는 HTTP 요청, 응답 처리 과정에서 사용됨
# 미들웨어에서 에러가 발생하는 경우, 뷰 단위까지 가지 않고 바로 Response가 프론트엔드로 반환됨
# 

class CookieTokenMiddleware:
    """
    쿠키에 저장된 토큰을 요청의 헤더에 추가함
    """
    def __init__(self, get_response):
        self.get_response = get_response

    def __call__(self, request):
        
        access_token = request.COOKIES.get('access_token')
        refresh_token = request.COOKIES.get('refresh_token')

        if access_token and refresh_token:
            request.META['HTTP_AUTHORIZATION'] = f'Bearer {access_token}'
            print("액세스 토큰 : 쿠키 -> 헤더 이동 성공")
        
        response = self.get_response(request)

        return response

class TokenRefreshMiddleware:
    """
    로그인된 사용자의 경우, 리프레시 토큰 유효 시
    액세스 토큰이 자동 재발급됨
    """
    def __init__(self, get_response):
        self.get_response = get_response

    def process_request(self, request):
        """뷰 처리 전 백엔드 뷰로 들어가는 과정에서의 전처리 함수"""

        refresh_token = request.COOKIES.get('refresh_token')
        access_token = request.COOKIES.get('access_token')

        if refresh_token and access_token:
            try:
                RefreshToken(refresh_token).check_blacklist() 
            except TokenError:
                raise AuthenticationFailed("리프레쉬 토큰이 만료되었습니다. 재로그인 필요.")
            
            # 리프레시 유효 / 액세스 만료 시, 액세스를 재발급함

            try:
                access_token = AccessToken(access_token) # 문자열 -> 객체 변환 및 액세스 토큰 검증
                user_id = access_token.payload['user_id']

            except TokenError as e:
                # 액세스 토큰 만료 시 진입
                try:
                    # 일단 이게 오류 없이 작동하므로 이렇게 두겠음
                    refresh_token_obj = RefreshToken(refresh_token)
                    user = User.objects.get(id = refresh_token_obj.payload['user_id'])
                    refresh = RefreshToken.for_user(user) 
                    
                    # 토큰 재발급
                    access_token = refresh.access_token
                    user_id = access_token.payload['user_id'] 

                except TokenError as e:
                    print(f"토큰 오류 : {e}")
                    raise AuthenticationFailed("액세스 토큰 재발급에 실패하였습니다.")

            # 토큰이 유효할 경우 유저 정보 반환
            user = User.objects.get(id = user_id)

            request.user = user.id
            print('미들웨어 : ', request.user)
            request.username = user.username
            request.nickname = user.nickname 

            # 재발급된 액세스 토큰은 일단 request 내에 넣어둔다
            request.new_access_token = access_token
            print("middleware : 액세스 토큰 (재)발급 완료")

        else:
            return None

    def process_response(self, request, response):
        """뷰 처리후 프론트엔드로 반환하는 과정에서의 후처리 함수"""

        if response is not None:
            if hasattr(request, 'new_access_token'):
                response.set_cookie('access_token', request.new_access_token, httponly = True)
                if isinstance(response, Response): # Response 객체에서만 작동
                    response['userId'] = request.username
                    response['nickname'] = request.nickname
                    
        return response
        
    
    def __call__(self, request):
        try:
            self.process_request(request) # 뷰 호출 전 실행 코드 
            
            response = self.get_response(request) # view이거나 다른 미들웨어 호출
    
            response = self.process_response(request, response) # 뷰 호출 후 실행 코드
            return response
        
        except AuthenticationFailed as e:
            return Response(str(e), status = status.HTTP_401_UNAUTHORIZED)
```

- 미들웨어를 설정하기 위해, `settings.py`에도 미들웨어를 추가한다.
```python
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'corsheaders.middleware.CorsMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'backend.middleware.TokenRefreshMiddleware', # 얘와
    'backend.middleware.CookieTokenMiddleware', # 얘를 추가했다.
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]
```
> 여기서 `request`가 들어오는 상황에서는 위부터 아래로 진행되고, `response`인 경우에는 아래부터 위로 진행된다고 한다. 따라서 **순서가 중요**함. 
> - 예전에 CorsMiddleware를 설정할 때도 순서가 중요했던 적이 있었다.
### 미들웨어
- 요청을 처리하기 전이나 응답을 처리한 후, HTTP와 관련된 설정을 진행한다.
```python
class middleware:
	def __init__(self, get_response):
		self.get_response = get_response

	def __call__(self, request):
			self.process_request(request) # 뷰 호출 전 실행 코드 
            
            response = self.get_response(request) # view이거나 다른 미들웨어 호출
    
            response = self.process_response(request, response) # 뷰 호출 후 실행 코드
```
- 각 미들웨어는 크게 위의 구성을 갖는다.
- `self.get_response`는 이전이나 다음 미들웨어, 혹은 뷰를 호출한다. 
- 미들웨어에서는
	- `process_request(request)`로 `request`에 추가적인 처리를 해주고
	- `process_response(request, response)`로 `response`에 추가적인 처리를 해준다.
