- 쿠버네티스는 보안 정보를 관리하기 위한 `Secret`을 별도로 제공한다. 
- `ConfigMap`과 차이점 : Secret은 데이터가 `base64`로 저장된다.

> Secret 자체는 암호화되지 않는다.
> `etcd`에 접근이 가능하다면 누구나 저장된 `Secret`을 확인할 수 있다. `Valut`와 같은 외부 솔루션으로 보안을 강화할 수 있다.

### Secret 만들기
`username.txt`
```
admin
```
`password.txt`
```
1q2w3e4r
```

- Secret 생성 및 확인
```sh
kubectl create secret generic db-user-pass --from-file=./username.txt --from-file=./password.txt

# secret 상세 조회
kubectl describe secret/db-user-pass

# -o yaml로 상세 조회
kubectl get secret/db-user-pass -o yaml

# 저장된 데이터 base64 decode
echo {.txt 내용} | base64 -- decode
```

- Secret을 환경 변수로 연결
`alpine-env.yml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: alpine-env
spec:
  containers:
    - name: alpine
      image: alpine
      command: ["sleep"]
      args: ["100000"]
      env:
        - name: DB_USERNAME
          valueFrom:
            secretKeyRef:
              name: db-user-pass
              key: username.txt
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-user-pass
              key: password.txt
```

- 환경변수 확인
```sh
kubectl apply -f alpine-env.yml

kubectl exec -it alpine-env -- env
```
- 환경변수만을 바꿔준다면 `apply`가 정상적으로 작동하지 않을 수 있다 : 이때는 `kubectl delete pod/alpine-env`를 통해 `pod`을 지운 뒤 재실행하자(`alpine-env`는 `pod`으로 구동되니까 가능한 거임!)

##### 마무리
`Secret`은 완전히 믿고 쓸 수 있는 상황은 아니어서, 정말 암호화가 필요하다면 별도의 솔루션을 고려하자.
