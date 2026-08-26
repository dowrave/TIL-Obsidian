- 내 경우, 기본 설정인 `sqlite3`을 쓰다가 `MySQL`로 옮기게 되었다. 이 과정을 적음.

- 이용 기능
	- `Django`에서 지원하는 `dumpdata`와 `loaddata`

- `sqlite3`의 데이터를 빼서, `MySQL`에 데이터를 넣는다. **각 과정은 해당 데이터베이스인 상태에서 진행한다.**
- MySQL에 데이터베이스가 만들어져 있다는 가정 하에 진행하고, 실행 환경은 **윈도우**이다.

---
## 기본 베이스
1. `SQLite3`인 상태에서 아래의 명령어를 실행한다. **`Git Bash`나 `Linux Shell` 등에서 실행하는 것이 좋은데**, `Window PowerShell`은 인코딩이 `utf-16`으로 나오기 때문.
```sh
python manage.py dumpdata > datadump.py
```

2. `settings.py`에서 데이터베이스를 `sqlite3`에서 `MySQL`의 설정으로 바꾼다.
```python
# DATABASES = {
#    'default': {
#        'ENGINE': 'django.db.backends.sqlite3',
#        'NAME': BASE_DIR / 'db.sqlite3',
#    }
# }

# MySQL 버전

# DATABASES = {
#     'default' : {
#         'ENGINE' : 'django.db.backends.mysql',
#         'NAME': secrets['DATABASE_NAME'],
#         'USER': secrets['DATABASE_USER'],
#         'PASSWORD': secrets['DATABASE_PASSWORD'],
#         'HOST': secrets['DATABASE_HOST'],
#         'PORT': secrets['DATABASE_PORT'],
#     }
# }
```

3. 현재 새로운 DB에는 테이블이 없는 상태이다. 테이블을 만들어준다.
```sh
python manage.py makemigrations
python manage.py migrate
```

4. `MySQL`로 바꾼 뒤, 아래의 명령어를 실행한다.
```sh
python manage.py loaddata datadump.json
```

---
## 발생한 이슈

1. `loaddata` 과정에서 인코딩 관련 오류 발생
	- `powershell`에서 `python manage.py dumpdata > datadump.json` 명령어를 실행하면 기본 `utf-16`으로 인코딩된다. 일반적으로 `utf-8`을 쓰니 위의 명령어는 `Linux Shell`이나 `Git Bash`을 이용해 실행하자.
	- 이를 파워쉘에서 실행한 게 위쪽의 과정에 넣은 명령어이다. 

2. `loaddata` 과정에서
```
django.db.utils.IntegrityError: Problem installing fixture 'C:\Users\HyeonTae Lee\Desktop\mysite_django_react\backend\datadump.json': Could not load contenttypes.ContentType(pk=14): (1062, "Duplicate entry 'token_blacklist-blacklistedtoken' for key 'django_content_type.django_content_type_app_label_model_76bd3d3b_uniq'")
```
위와 같은 `IntegrityError` 발생 


- `migrate` 과정에서 Django는 앱의 모델들에 대한 테이블을 생성하고, 각 모델에 대응한는 `ContentType` 엔트리를 `django_content_type` 테이블에 추가한다.
- 이 때, 덤프한 데이터를 덮어씌우려는 과정에서 이미 생성된 `ContentType` 엔트리와 중복될 수 있다.
- 따라서, `datadump.json`에서 `contenttype`을 추가하는 모든 항목을 직접 제거한 다음, `python manage.py loaddata datadump.json`을 수행했고, 정상적으로 `Installed 566 object(s) from 1 fixture(s)`이 출력되었다.

3. 그럼에도 `sqlite3`에서 잘 작동하던 기능이 `mysql`에서 잘 작동하지 않는 경우가 있다.
	- 내 경우는 아카이브를 조회하는 기능에서 해당 문제가 발생했음 - 더 뜯어보니, **`category`를 받아서 DB에서 조회할 때 객체를 `sqlite` 때와 달리 반환하지 못하는 문제가 있음**
	- [[Django - sqlite와 MySQL의 문자열 처리 차이(by ChatGPT)]] 

4. 또, 이런 문제도 있었다.
```python
	if year:
		q_objects &= Q(created_at__year = year)
		if month:
			q_objects &= Q(created_at__month = month)
			if day:
				q_objects &= Q(created_at__day = day)
```
> - `sqlite3`에선 위 코드로도 잘 작동했다(조회하는 값이 한국 시간 기준)
> - 근데 `MySQL`에서는 잘 작동하지 않았음 : 단순히 '날짜 기준이 달라졌다'가 아니라, 아예 기간에 대한 조회가 작동하지 않았다.
- [[Django - sqlite와 MySQL의 시간대 처리 차이(by ChatGPT)]] 참조.

- 따라서 위 코드는 이렇게 수정되었다. 더 좋은 방법은 모르겠음
```python
	# 기간별 아카이브 조회 - MySQL
	year_rawsql = RawSQL(
		"YEAR(CONVERT_TZ(created_at, '+00:00', '+09:00'))", []
	)
	month_rawsql = RawSQL(
		"MONTH(CONVERT_TZ(created_at, '+00:00', '+09:00'))", []
	)
	day_rawsql = RawSQL(
		"DAY(CONVERT_TZ(created_at, '+00:00', '+09:00'))", []
	)

	if year:
		q_objects &= Q(id__in=Post.objects.annotate(year = year_rawsql)
					   .filter(year = year)
					   .values('id'))
		if month:
			q_objects &= Q(id__in=Post.objects.annotate(month = month_rawsql)
						.filter(month = month)
						.values('id'))
			if day:
				q_objects &= Q(id__in=Post.objects.annotate(day = day_rawsql)
							.filter(day = day)
							.values('id'))
```
