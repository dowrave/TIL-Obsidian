- `django`와 `react`를 활용해 간단한 `todolist-app`을 구현한다.
- 리액트의 경우, `react-vite-ts`를 이용함
- 백엔드와 프론트엔드 설치 순서는 크게 상관은 없고 프로젝트마다 다르다고 함
## 1. React-vite-ts 설치
1. `Node.js`, `npm` 설치
2. `vite` 프로젝트 생성 : `npx create-vite {app이름} --template react-ts`
3. 프로젝트 내부에서 `npm install`

## 2. Django 설정
1. 가상환경 생성 및 활성화 
```sh
python -m venv venv
venv/scripts/activate
```

2. django 설치 : `pip install django`
3. django 프로젝트 생성 : `django-admin startproject {프로젝트이름}`
4. django 앱 생성 및 설정 : `cd {프로젝트이름} startapp {앱이름}`

## 3. 통합 설정
### 1. Django : CORS(Cross Origin Resource Sharing) 활성화하기
1. 패키지 설치
```sh
pip install django-cors-headers
```

2. `settings.py`
```python
INSTALLED_APPS = [
	'corsheaders',
]

MIDDLEWARE = [ 
		 'corsheaders.middleware,CorsMiddleware',
	 ]

CORS_ALLOW_ORIGINS = [
	"프론트엔드 서버:포트"
]
```

### 2. React에서 Django API 호출
1. `axios` 설치
```sh
npm install axios
```

2. 컴포넌트에서 API 호출 예시
```tsx
import axios from 'axios';

useEffect(() => {
	axios.get('http://localhost:8000/api/data')
		.then(response => {
			console.log(response.data);
		})
		.catch(error => {
			console.error(error);
		});
}, []);
```

- 이외에도 이런 것들이 필요할 수 있다
```sh
pip install djangorestframework
```

- `settings.py`
```python
INSTALLED_APPS = [
	'rest_framework',
]
```