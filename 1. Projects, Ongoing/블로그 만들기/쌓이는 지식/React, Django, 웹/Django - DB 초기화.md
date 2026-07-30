- 애초에 테이블을 바꾸기 전에 잘 설계해야 한다!
- 그럼에도 뭔가 꼬여서 DB 자체를 초기화하고 싶은 경우는 이런 방법들이 있다.

## 1. DB를 처음부터 다시 만들기
```sh
python manage.py migrate {your_app_name} zero
python manage.py migrate
```
> 프로젝트 전체가 아니라 **특정 앱만 초기 상태로 되돌리는 방식**이다.

## 2. 모든 데이터를 삭제하고 다시 만들기
```sh
python manage.py flush
```
> **모든 데이터를 삭제**하는 것에 유의하자.


### 만약 위 방법으로도 안 먹힌다면
- 위 방법 + SQL을 완전히 초기화(기본 : `db.sqlite3`)하고 진행해야 함
- `sqlite3`을 쓴다면 해당 파일을 그냥 삭제하면 된다. 프로젝트의 루트 디렉토리에 있음.
- 만약 `migrations` 폴더도 지우고, `db.sqlite3`도 지웠는데 `No Changes Detected`가 발생한다면
```sh
python manage.py makemigrations {app_name}
```
> 을 한번 쳐보자.


