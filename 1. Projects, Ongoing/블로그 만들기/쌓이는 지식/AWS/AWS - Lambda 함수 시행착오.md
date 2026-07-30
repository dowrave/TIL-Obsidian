### 상황 1. ssm에 접속 못하는 상황
**무한 로딩 이슈**
- 환경 변수는 `Systems Manager - 파라미터 스토어`에 저장해둔 뒤 그때그때 가져오는 방식을 이용하려고 했다. 이를 위해서는 `boto3`이라는 라이브러리가 필요한데, 얘가 `Lambda` 함수에서 요청을 보낼 때 무한 로딩에 걸리는 문제가 있었다.
- 하도 만진게 많아서 어떻게 정리를 해야 할지 모르겠음. 
- 가장 유력한 원인으로 생각되는 건, `ssm` 엔드포인트의 인바운드 규칙을 `HTTPS`로 설정한 줄 알았는데 `HTTP`로 설정된 게 있어서 그걸 바꿔줬다.

### 상황 2. steamspy.com에서 데이터를 못 읽어오는 상황
```
- VPC 외부의 IP에 요청을 보냈으나 최초 1000개의 데이터도 수집하지 못하고 TimeoutError: [Errno 110] Connection timed out에러가 발생. urllib3.**exceptions**.MaxRetryError: HTTPSConnectionPool(host='steamspy.com', port=443): Max retries exceeded with url: /api.php?request=all&page=0 (Caused by NewConnectionError('<urllib3.connection.HTTPSConnection object at 0x7f8f411e0ac0>: Failed to establish a new connection: [Errno 110] Connection timed out' 에러 발생

- 람다 함수의 보안 그룹 중, 아웃바운드 규칙에는 HTTPS, TCP, 443, IPv4 `0.0.0.0/0 설정이 있음. 외부 사이트 접근을 위한 규칙으로 보임.`

- subnet은 2개가 연결되어 있으며, 둘 모두 public 라우팅 테이블과 연결된 서브넷임.

- 해당 라우팅 테이블의 라우팅 대상은 `0.0.0.0/0 : igw` , `13.209.1.0/24 : eni`, `172.31.0.0 : local`임. 
```