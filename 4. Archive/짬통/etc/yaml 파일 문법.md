1. 들여쓰기는 **2칸** or 4칸
2. 데이터는 `key: value`로 정의
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: echo
  labels:
    type: app
```
- `key:` 입력 후 **반드시 띄어 써야 함**

3. 배열은 `-`로 표시
```yaml
person:
  skills:
    - docker
    - kubernetes
```

4. 주석은 `#`
5. 참/거짓은 `true`, `false` 외에도 `yes`, `no`를 지원함
6. `"1.2"`는 문자, `1.2`는 숫자로 인식함
7. 줄바꿈(`newline`) :
	- `|` : 마지막 줄바꿈 포함 
	- `|-`  : 마지막 줄바꿈 제외
	- `>` : 중간에 들어간 빈 줄 제외
```yaml
newlines_sample: |
  number one line
  second line
  last line
```

그 외 주의할 점
- `:`가 들어간 문자열은 `""`를 반드시 쳐줘야 함(그 외에는 안쳐줘도 무방)

참고할 사이트
[json2yaml](https://www.json2yaml.com) : json -> yaml 문법 변환 사이트  
[yamllint](http://www.yamllint.com) : yaml 문법 체크 & 해석 결과