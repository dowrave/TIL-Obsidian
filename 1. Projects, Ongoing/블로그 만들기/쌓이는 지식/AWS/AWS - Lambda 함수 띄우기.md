- 기존 `steamspypi`를 이용한 데이터 수집 과정을 AWS Lambda에 띄우고, RDS에 저장하는 과정까지를 구현한다.

## 과정

### 이미지 생성 및 레포지토리에 넣기
- 우선 갖고 있는 함수를 `lambda_handler`의 형식으로 넣어야 함
```python
# Lambda 실행 시 main_func은 1회만 실행되어야 한다
def lambda_handler(event, context):
    test = False
    main_func(test, TO_CSV = False, TO_SQL = True)
    return {
        'statusCode' : 200,
        'body': json.dumps("람다 함수 실행 성공적")
    }

if __name__ == "__main__":
    lambda_handler(None, None)
```

> - 내 경우는 `boto3`을 이용해 `파라미터 스토어`와 직접 연결하므로, `requirements.txt`에 `boto3`을 추가했음 -`urllib3`버전을 기존 `2.0.2` -> `1.25.4`로 낮췄는데, 충돌 가능성은 있다

- 이미지를 빌드함
```sh
docker build -t {이미지 이름} . 
```

- 이미지를 태그함
```sh
docker tag (이미지 이름):latest <ECR 프라이빗 리포지토리 이름>:latest
```
> 여기서 ECR 레포지토리는 `private`으로 설정해두는 게 맞는 듯? 안 그러면 나중에 `lambda` 콘솔에서 이미지를 연결하려 할 때 나타나는 게 없었다.

- ECR에 로그인함
```sh
aws ecr get-login-password --region {region} | docker login --username AWS --password-stdin {userid}.dkr.ecr.{region}.amazonaws.com
```
> 만약 안되면`broken pipe` -  `|`을 기준으로 각 명령어를 따로따로 실행해보자

- 이미지를 푸시함
```sh
docker push <registry_url>/<repository_name>:<tag>

# 예시
docker push {12NumberUserId}.dkr.ecr.{Region}.amazonaws.com/{RepositoryName}:{Tag}
```
> ECR에는 `이미지 태그`가 제일 앞으로 오므로 태그에 적절한 별명을 넣어주자

### 환경 변수 설정

- Lambda 함수의 "구성" 탭에서 환경 변수를 설정합니다.
- `MYSQL_STEAM_DB`, `MYSQL_STEAM_USER`, `MYSQL_STEAM_PASSWORD` 등 필요한 환경 변수를 추가합니다.
- 민감한 정보는 AWS Systems Manager Parameter Store를 사용하여 암호화된 형태로 저장하고 참조할 수 있습니다.
- [[AWS - Systems Manager Parameter Store로 환경 변수 세팅하기]]

### boto3을 사용할 경우) 엔드포인트 설정
- `Lambda` 함수를 VPC에 연결 시, 해당 함수는 인터넷에 직접 접근할 수 없고, VPC 내부 리소스와만 통신할 수 있게 된다.
	- AWS Management Console에서 Lambda 서비스로 이동합니다.
	- 문제가 발생한 Lambda 함수를 선택합니다.
	- Lambda 함수의 상세 정보 페이지에서 "구성(Configuration)" 탭을 클릭합니다.
	- 왼쪽 메뉴에서 "VPC"를 선택합니다.
	- Lambda 함수가 VPC에 연결되어 있는지 확인합니다. VPC가 설정되어 있다면, 사용된 서브넷과 보안 그룹을 확인합니다.
	- AWS Management Console에서 VPC 서비스로 이동합니다.
- `엔드포인트(Endpoints)` 섹션으로 이동합니다.
	- "Create Endpoint"를 클릭하고, AWS 서비스 목록에서 "Systems Manager Service"를 선택합니다.
		- `ssm`으로 조회하면 나타남
	- Lambda 함수가 연결된 VPC, 서브넷, 보안 그룹을 선택하고 엔드포인트를 생성합니다.
		- 일단 `public 라우팅 테이블`에 연결된 것들만 지정함
	- (굳이 필요 없을지도?) 생성된 엔드포인트의 DNS 이름을 확인합니다. 
	- Lambda 함수의 VPC 보안 그룹 설정에서, 아웃바운드 규칙을 확인합니다. **SSM 엔드포인트의 DNS 이름**과 **포트(443)에 대한 HTTPS 액세스를 허용**하는 규칙이 있어야 합니다.
		- 내 경우에는 2개의 Public 서브넷을 설정했다.  Lambda 함수에 설정한 `보안 그룹` - 아웃바운드 규칙에서 총 3개의 규칙을 추가했다.
			 1. `IPv4, HTTPS, 443` 및 `대상 : 각각의 서브넷의 IPv4 값`을 넣음
			 2. 외부 인터넷 액세스를 위해 `IPv4, HTTPS, 443` 및 `대상 : 0.0.0.0/0`을 넣음
### VPC 및 보안 그룹 설정
- Lambda 함수가 RDS에 접근할 수 있도록 VPC와 보안 그룹을 설정합니다.
- Lambda 함수의 "구성" 탭에서 "VPC"를 선택하고, RDS 인스턴스와 동일한 VPC와 서브넷을 지정합니다.
- 필요한 경우 새로운 보안 그룹을 생성하고, RDS 인스턴스의 인바운드 규칙에 해당 보안 그룹을 추가합니다.
###  실행 역할 권한 설정
- Lambda 함수가 CloudWatch 로그에 쓰기 권한을 갖도록 실행 역할(Execution Role)에 적절한 권한을 부여합니다.
- IAM 콘솔에서 Lambda 함수의 실행 역할을 찾아 "CloudWatchLogsFullAccess" 정책을 연결합니다.


### 트리거 설정 (선택사항)
- 주기적으로 데이터를 수집해야 한다면 EventBridge(CloudWatch Events)를 사용하여 Lambda 함수를 트리거할 수 있습니다.
- EventBridge 콘솔에서 새 규칙을 생성하고, 일정(cron 표현식)과 대상(Lambda 함수)을 지정합니다.
###  테스트 및 모니터링
- Lambda 함수를 테스트 이벤트와 함께 실행해 보고, CloudWatch 로그를 통해 실행 결과를 확인합니다.
- 필요한 경우 코드를 수정하고 다시 업로드합니다.
- CloudWatch 경보를 설정하여 Lambda 함수의 오류율이나 실행 시간 등을 모니터링할 수 있습니다.


## 시행착오 중

> 발생한 문제 : 이미지를 빌드하고, 테스트로 실험해봤는데 `botocore.exceptions.ConnectTimeoutError: Connect timeout on endpoint URL: "https://vpce-03c77e1686443b046-5l9j639c.ssm.ap-northeast-2.vpce.amazonaws.com/"` 에러가 발생함.



1. 아래처럼 설정하고 다시 빌드해봄 -> 이슈 동일 
```PYTHON
ssm_client = boto3.client('ssm', endpoint_url='https://vpce-<endpoint-id>.ssm.<region>.vpce.amazonaws.com')
```

2. 설정한 엔드포인트 자체는 정상적으로 보임
	- 서비스 이름: `com.amazonaws.ap-northeast-2.ssm`
	    - 이는 AWS Systems Manager(SSM) 서비스를 나타냅니다.
	- 엔드포인트 유형: `Interface`
	    - 인터페이스 엔드포인트는 프라이빗 IP 주소를 통해 액세스되며, VPC 내부에서 서비스에 대한 연결을 제공합니다.
	- 프라이빗 DNS 이름 활성화됨: `예`
	    - VPC 내에서 서비스에 액세스할 때 프라이빗 DNS 이름을 사용할 수 있습니다.
	- 프라이빗 DNS 이름: `ssm.ap-northeast-2.amazonaws.com`
	    - 이는 VPC 내에서 SSM 서비스에 액세스하기 위해 사용할 수 있는 DNS 이름입니다.
	- `DNS 이름에는 엔드포인트 ID(vpce-03c77e1686443b046)와 가용 영역 정보(ap-northeast-2a, ap-northeast-2b)가 포함`되어 있습니다.

3. `Lambda 함수`와 연결된 서브넷이 있는 Public 라우팅 테이블에 `라우팅 설정` 추가
	- `13.209.1.0/24` // `eni-093fd72252731e704`

5. `VPC > 네트워크 ACL`에 모든 서브넷이 있는 ACL에 아래 규칙 추가
	- 인바운드 : `101, HTTPS(443), TCP(6), 443, 172.31.0.0/16`
	- 아웃바운드 : `101, HTTPS(443), TCP(6), 443, 0.0.0.0/0`
