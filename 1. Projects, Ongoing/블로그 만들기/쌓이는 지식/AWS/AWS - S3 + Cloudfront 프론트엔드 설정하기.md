- 2024년 5월 1일 작업 기준이므로 달라질 수 있음
- `Claude - Opus`는 짱이야 
	- Sonnet을 썼을 때는 겁나 헤맸음
---
### S3 버킷 생성:
- AWS Management Console에 로그인하고 S3 서비스로 이동합니다.
- "Create Bucket"을 클릭하여 새로운 버킷을 생성합니다.
- 버킷 이름을 입력하고 리전을 선택합니다.
- 버킷의 **퍼블릭 액세스를 허용**하도록 설정합니다.
- 생성한 버킷을 선택하고 "Properties" 탭으로 이동합니다.
- "Static website hosting"을 활성화하고 인덱스 문서와 에러 문서를 지정합니다.
### 빌드 결과물 업로드:
- 로컬 머신에서 S3 버킷으로 빌드 결과물을 업로드합니다.
- AWS CLI를 사용하거나 GUI 도구를 사용할 수 있습니다.
- 예를 들어, AWS CLI를 사용한다면 다음 명령어를 실행합니다:
	`aws s3 sync ./dist s3://your-bucket-name`
        
### CloudFront 배포 생성:
- AWS Management Console에서 CloudFront 서비스로 이동합니다.
- "Create Distribution"을 클릭하여 새로운 CloudFront 배포를 생성합니다.
- "Web" 옵션을 선택합니다.
- "Origin Domain Name"에서 이전에 생성한 S3 버킷을 선택합니다.
- "Viewer Protocol Policy"를 "Redirect HTTP to HTTPS"로 설정합니다.
- 필요한 경우 캐시 동작, 지리적 제한 등의 추가 설정을 구성합니다.
- 배포를 생성합니다.

### S3 버킷 권한 설정
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "PublicReadGetObject",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::[bucket-name]/*"
        }
    ]
}
```

---

- 아래 2개는 아직 도메인을 안 달아놔서 진행 X
###  CNAME 설정 (선택사항):
- 사용자 지정 도메인을 사용하려면 도메인 등록 기관에서 CNAME 레코드를 설정합니다.
- CNAME 레코드는 사용자 지정 도메인을 CloudFront 배포의 도메인 이름과 연결합니다.
### HTTPS 설정 (선택사항):
- 사용자 지정 도메인을 사용하는 경우 HTTPS를 설정할 수 있습니다.
- AWS Certificate Manager를 사용하여 SSL/TLS 인증서를 발급하고 CloudFront 배포에 연결합니다.