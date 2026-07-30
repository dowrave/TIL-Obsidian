- 이전 : [[5-1. FastAPI 튜토리얼]]
## FAST API의 CRUD 작성법
> - `Create` : `POST`
> - `Read` : `GET`
> - `Update` : `PUT`
> - `Delete` : `DELETE`

### 1.  `Path` 파라미터 이용
| Type   | Request Header                                | Request Body | Response Boddy            |
| ------ | --------------------------------------------- | ------------ | ------------------------- |
| Create | `POST /users/name/{name}/nickname/{nickname}` | {}           | `{ "status" : "success"}` |
| Read   | `GET /users/name/{name}`                      | {}           | `{"nickname" : "world"}`  |
| Update | `PUT /users/name/{name}/nickname/{nickname}`  | {}           | `{"status" : "success"}`  |
| Delete | `DELETE /users/name/{name}`                   | {}           | `{"status" : "success"}`                          |

#### API 구현
- `crud_path.py`
```python
# crud_path.py  
from fastapi import FastAPI, HTTPException  
  
# Create a FastAPI instance  
app = FastAPI()  
  
# User database  
USER_DB = {}  
  
# Fail response  
NAME_NOT_FOUND = HTTPException(status_code=400, detail="Name not found.")  
  
  
@app.post("/users/name/{name}/nickname/{nickname}")  
def create_user(name: str, nickname: str):  
	USER_DB[name] = nickname  
	return {"status": "success"}  
	  
  
@app.get("/users/name/{name}")  
def read_user(name: str):  
	if name not in USER_DB:  
		raise NAME_NOT_FOUND  
	return {"nickname": USER_DB[name]}  
  
  
@app.put("/users/name/{name}/nickname/{nickname}")  
def update_user(name: str, nickname: str):  
	if name not in USER_DB:  
		raise NAME_NOT_FOUND  
	USER_DB[name] = nickname  
	return {"status": "success"}  
  
  
@app.delete("/users/name/{name}")  
def delete_user(name: str):  
	if name not in USER_DB:  
		raise NAME_NOT_FOUND  
	del USER_DB[name]  
	return {"status": "success"}
```

- 실행
```sh
uvicorn crud_path:app --reload
```
- [http://localhost:8000/docs](http://localhost:8000/docs) 에서 각 HTTP 메서드의 파라미터를 보자 
![[Pasted image 20230102155255.png]]

### 2. `Query` 파라미터 이용

| TYPE   | Reques Header   | Request Body                              | Response Body            |
| ------ | --------------- | ----------------------------------------- | ------------------------ |
| Create | `POST /users`   | `{"name":"hello" , "nickname" : "world"`  | `{"status": "success"}`  |
| Read   | `GET /users`    | `{"name" : "hello"}`                      | `{"nickname" : "world"}` |
| Update | `PUT /users`    | `{"name" : "hello", "nickname" : world2`} | `{"status" : "success"}` |
| Delete | `DELETE /users` | `{"name" : "hello"}`                      | `{"status" : "success"}` |

#### API 구현
- `curd_query.py`
```python
# crud_query.py  
from fastapi import FastAPI, HTTPException  
  
# Create a FastAPI instance  
app = FastAPI()  
  
# User database  
USER_DB = {}  
  
# Fail response  
NAME_NOT_FOUND = HTTPException(status_code=400, detail="Name not found.")  
  
  
@app.post("/users")  
def create_user(name: str, nickname: str):  
	USER_DB[name] = nickname  
	return {"status": "success"}  
  
  
@app.get("/users")  
def read_user(name: str):  
	if name not in USER_DB:  
		raise NAME_NOT_FOUND  
	return {"nickname": USER_DB[name]}  
  
  
@app.put("/users")  
def update_user(name: str, nickname: str):  
	if name not in USER_DB:  
		raise NAME_NOT_FOUND  
	USER_DB[name] = nickname  
	return {"status": "success"}  
  
  
@app.delete("/users")  
def delete_user(name: str):  
	if name not in USER_DB:  
		raise NAME_NOT_FOUND   
	del USER_DB[name]  
	return {"status": "success"}
```

- 실행
```sh
uvicorn crud_path:app --reload
```
- [http://localhost:8000/docs](http://localhost:8000/docs) 마찬가지
![[Pasted image 20230102155842.png]]

- 위의 `path`와 달리 여기는 `path parameter`가 포함되어 있지 않다.

### API 테스트
- `Swagger UI`(`localhost:8000/docs`)에 접속해서 아래 시나리오의 작동을 확인하자 (각 항목 클릭 -> `Try it out`  클릭 -> name, nickname 넣으면 됨)
- 두 파일 모두에 대해 해볼 것 

1. CREATE
	- `name : hello`  + `nickname : world`
	- 결과 : `Server Response Body : { "status" : "success" }`

2. GET
	- `name : hello` 
		- 결과 `{"nickname" : "world"}`
	- `name : hello2`
		- 결과 `{"detail" : "Name not found."}` 및 `400 에러` 확인

3. UPDATE
	- `name : hello, nickname : world`
		- `status : success` 확인
		- 다시 GET으로 접근해서 `name : hello`를 넣었을 때 `nickname : world`를 반환하는지 확인
	- `name:hello, nickname : world2`
		- `status: success` 확인
		- GET으로 접근, `name:hello` 접근 시 `nickname: world2`인지 확인
	- `name:hello2, nickname:world2` 
		- `detail : name not found` 리턴 & `400 에러` 확인

4. DELETE
	- `name : hello`
		- `status : success` 리턴 확인
		- GET 접근, `name : hello` 입력 시 `detail : name not found` 확인
	- `name : hello2`
		- `detail : name not found` & `400 에러` 확인

> 실습 중 맞은 상황
> `crud_path.py`
> (둘다 어떤 방식을 쓰든 공통인 것 같음)
>> 1. 경로는 `/`로 시작해야 함
>> 2. 다시 시작되는 상황이면 PUT으로 넣은 데이터 날아감

### Path vs Query
- `Path`는 경로에 변수 값이 저장되어 함수에 전달됨
- `Query`는 경로에 변수 값이 저장되지 않음 

- 다음 : [[5-3. Pydantic CRUD]]