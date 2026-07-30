AWS RDS를 사용하여 MySQL 데이터베이스를 클라우드로 마이그레이션하는 과정은 다음과 같습니다:

### 1. 로컬 MySQL 데이터베이스 백업:
   - `mysqldump` 명령어를 사용하여 로컬 MySQL 데이터베이스를 백업합니다.
   - 터미널에서 다음 명령어를 실행하여 백업 파일을 생성합니다:
     ```
     mysqldump -u username -p database_name > backup.sql
     ```
   - `username`은 MySQL 사용자 이름으로, `database_name`은 백업할 데이터베이스 이름으로 바꿔주세요.
   - 백업 파일(`backup.sql`)이 생성됩니다.

### 2. RDS 인스턴스 생성:
   - AWS Management Console에 로그인하고 RDS 서비스로 이동합니다.
   - "Create database"를 클릭하여 새로운 RDS 인스턴스를 생성합니다.
   - "Engine type"에서 "MySQL"을 선택합니다.
   - "Templates"에서 "Free tier"를 선택하여 프리 티어 사용량 한도 내에서 인스턴스를 생성합니다.
   - "DB instance identifier", "Master username", "Master password"를 입력합니다.
   - 나머지 설정은 기본값으로 유지하고 인스턴스를 생성합니다.

### 3. 보안 그룹 설정:
   - RDS 인스턴스가 생성되면 "Connectivity & security" 탭으로 이동합니다.
   - "Security group rules"에서 "Inbound" 규칙을 편집합니다.
   - "Type"을 "MYSQL/Aurora"로 선택하고, "Source"를 "Anywhere"로 설정합니다. (보안을 위해 특정 IP 주소로 제한할 수도 있습니다.)
   - 변경 사항을 저장합니다.

### 4. 데이터 복원:
   - RDS 인스턴스의 엔드포인트를 확인합니다. ("Connectivity & security" 탭에서 확인 가능)
   - 터미널에서 다음 명령어를 사용하여 백업 파일을 RDS 인스턴스로 복원합니다:
     ```
     mysql -h endpoint -u username -p database_name < backup.sql
     ```
   - `endpoint`는 RDS 인스턴스의 엔드포인트로, `username`은 RDS 인스턴스 생성 시 지정한 마스터 사용자 이름으로, `database_name`은 생성한 데이터베이스 이름으로 바꿔주세요.
   - 메시지가 나타나면 RDS 인스턴스 생성 시 지정한 마스터 비밀번호를 입력합니다.

> 여기서 발생한 에러 : `ERROR 2003 (HY000): Can't connect to MySQL server on 'mydatabase.cleuoeckeix6.ap-northeast-2.rds.amazonaws.com:3306' (10060)`
> - [AWS RDS 접속 에러 HY000](https://dream-and-develop.tistory.com/416)
> 	- AWS 인스턴스의 인/아웃 바운드 네트워크 규칙이 MySQL 클라이언트가 실행되는 현재 호스트에 도달 못해 발생함
> 	- RDS 보안 그룹의 Public IP 주소를 등록, RDS MySQL 인스턴스에 액세스 권한을 준다.
> 	- IPv4, IPv6 각각에 `유형 - 모든 트래픽`을 허용해준다.

> 에러 원인 : `mydatabase - 수정 - 연결 - 추가 구성 - 퍼블릭 액세스 불가능`이면 VPC 외부에서의 접근이 불가능하다
> 	- 따라서 **백업을 시도하겠다면 `퍼블릭 액세스를 가능`으로 일시적으로 바꿨다가, 백업을 하고 나서 다시 `불가능`으로 변경**하면 됨 
> 	- `불가능`이 기본적으로 바람직함 : 외부에서의 인스턴스 접근은 가능한 최소화하는 게 좋기 때문이다.

> 따라서 작업은 아래처럼 가져갔다.
> 0. `RDS 콘솔`에서 `수정 - 퍼블릭 액세스 가능`으로 변경
> 1. `mysql -h [RDS엔드포인트] -u [RDS마스터이름] -p`로 RDS의 mysql에 접근
> 2. `CREATE DATABASE blog`, `CREATE DATABASE steam`으로 2개의 DB를 생성
> 3. `exit`로 나온 다음, `mysql -h [RDS엔드포인트] -u [RDS마스터이름] -p [db이름] < [db백업파일.sql]`로 백업 파일을 데이터베이스에 등록
> 4. `RDS 콘솔 - 데이터베이스 - 수정 - 퍼블릭 액세스 불가능`으로 다시 변경
### 5. 데이터 확인:
   - RDS 인스턴스에 연결하여 데이터가 올바르게 복원되었는지 확인합니다.
   - MySQL 클라이언트 도구를 사용하거나 터미널에서 다음 명령어를 실행하여 연결할 수 있습니다:
     ```
     mysql -h endpoint -u username -p
     ```
   - 연결 후 데이터베이스와 테이블을 확인하고 데이터가 정상적으로 존재하는지 확인합니다.


