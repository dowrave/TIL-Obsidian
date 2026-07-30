- 직접 커스텀한 `MyUser`로 옮기는 과정이다.

## 1. User 테이블 정의
```python
class MyUser(AbstractBaseUser):

    username = models.CharField(
        max_length = 14,
        unique = True,
        validators = [RegexValidator(regex='^[a-z0-9]+$', 
                                     message="ID는 영소문자와 숫자만 가능합니다.")],
    )
    email = models.EmailField(
        verbose_name = 'email address', 
        max_length = 320,
        unique = True
    )
    nickname = models.CharField(
                            # min_length = 2,
                            max_length = 8, 
                            null = True, 
                            blank = True, 
                            unique = True,
                            validators = [RegexValidator(regex='^[a-zA-Z0-9가-힣]+$',
                                            message="닉네임은 한글, 영어 대소문자, 숫자로 2~8글자가 가능합니다.")])
    is_active = models.BooleanField(default = True)
    is_admin = models.BooleanField(default = False)

    objects = MyUserManager()

    USERNAME_FIELD = 'username'
    REQUIRED_FIELDS = ['email',] 
```
> - 어떤 필드가 구성되었는가만 보면 된다.
> - 일반적으로 db에 저장하는 `id`와, 사용자가 입력하는 아이디인 `username`은 별도의 것으로 친다.

## 2. 마이그레이션 스크립트 작성
- `MyUser`가 있는 앱의 `migrations/` 폴더에 있는 가장 마지막 `migration` 파일 다음의 파일을 작성한다. `{00xx}_{details_bla_bla}.py` 같은 규칙을 따르면 되는 듯.
```python
from django.db import migrations

def migrate_user_data(apps, schema_editor):
    User = apps.get_model('auth', 'User')
    MyUser = apps.get_model('authuser', 'MyUser')

	# 새롭게 작성하는 유저 테이블에 있는 필드는 다 써야 하는 듯.
    for user in User.objects.all():
        MyUser.objects.create(
            id=user.id,  
            username = user.username,
            email=user.email,
            nickname = user.username,
            is_active=user.is_active,
            is_admin=user.is_staff, 
            # 기타 필요한 필드를 여기에 추가
        )

class Migration(migrations.Migration):

    dependencies = [
        ('authuser', '0003_myuser_username_alter_myuser_id'),  # 이전 마이그레이션 파일명
    ]

    operations = [
        migrations.RunPython(migrate_user_data),
    ]
```


## 3. `python manage.py migrate`
