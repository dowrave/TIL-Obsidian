- 여기서 `다른 s3 버킷 스토리지`라 함은 백엔드의 **미디어 파일과 스태틱 파일이 저장**되는 곳이 되겠다.

### 문제 상황
- 프론트엔드에서 `s3` 스토리지 버킷의 데이터를 불러올 때, 이미지든 `mp3` 파일이든 모두 가져오지 못하고 있음
	- 여기서 `이미지`는 주로 `img`에 들어간 `src`값으로, `http://s3버킷주소`로 표현되고 있음
	- `mp3`파일은 백엔드로 요청을 보내서, 백엔드에서 스토리지 버킷의 데이터를 프론트엔드로 옮겨야 함

- Claude Opus에게 물어보면 대충..
	- `모든 퍼블릭 리소스 차단`을 켜면, `http://`로 데이터를 가져올 수 없다고 함
	- 대신 프론트엔드에서 임시적인 접근 권한을 얻도록 구성해서, 스토리지 버킷의 데이터를 가져올 수 있다고 한다.

### 해결 1) 이미지 불러오기
- 근데 그냥 이게 문제였음 
	- `버킷 스토리지`에서, **모든** 퍼블릭 액세스를 차단해야 하는가?
		- 2개의 항목만 액세스 차단을 활성화 했다
		- **_새_ ACL(액세스 제어 목록)을 통해 부여된 버킷 및 객체에 대한 퍼블릭 액세스 차단** 활성화
		- **_새_ 퍼블릭 버킷 또는 액세스 지점 정책을 통해 부여된 버킷 및 객체에 대한 퍼블릭 액세스 차단** 활성화
	- 해당 버킷 스토리지의 버킷 정책을 수정한다 : **프론트엔드, 백엔드, 로컬 에서의 s3:GetObject 권한을 활성화**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowAccessFromSpecificIPs",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::your-bucket-name/*",
      "Condition": {
        "IpAddress": {
          "aws:SourceIp": [
            "프론트엔드_IP_주소_또는_CIDR",
            "백엔드_IP_주소_또는_CIDR",
            "로컬_IP_주소_또는_CIDR"
          ]
        }
      }
    }
  ]
}
```

### 진행 중 ) mp3 파일(리스트) 불러오기
- `boto3`에서 액세스키랑 시크릿 액세스키를 제대로 못 얻는다는데, 난 그걸 넣는 로직도 구현한 적이 없다. 관련 디버깅이 필요함.
- 재생 목록 가져오는 것에서 에러가 나고 있음.

> 디버깅 중
> - `settings.py`의 내용 수정
	- `SongView`에서 파일을 못 읽는 게 아니다. 즉, `serializer`에서 에러가 날 가능성이 높음
	- 지금 과정이 데이터를 가져오는 과정이 아님에도 시리얼라이저에서 얻은 데이터의 개수가 0개인데, 이는 `settings.py`에서 `AWS_ACCESS_KEY`와 `AWS_SECRET_ACCESS_KEY`로 변수명을 설정했기 때문이다. 
	- `boto3` : `settings.py`에서 `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY_ID`로 변수 명을 인식한다.
> - 다음 오류 ) `NotImplementedError: This backend doesn't support absolute paths.`
> 	- 로컬에서는 `song.audio_file.path`를 썼지만, 지금은 `s3` 버킷의 `url`에 접근해야 하므로 아래처럼 수정해야 함
```python
	file_url = song.audio_file.url # AWS의 경우 다른 인스턴스인 s3에 있는 값이므로
	response = requests.get(file_url)
	file_content = response.content
```
>	- 이외에도 `with open()`문과 관련한 오류가 있었는데, 결론은 `StreamingHttpResponse`를 쓰면 `with open()`문을 쓰지 말고, 파일 이름을 그냥 지정해두면 됨.