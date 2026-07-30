- [[k8s_Deployment]] 실습 중, Deployment를 실행시켰을 때 문제 상황
```
NAME                             READY   STATUS             RESTARTS   AGE
pod/echo-deploy-6c8bb996-56mw5   0/1     ImagePullBackOff   0          9m23s
pod/echo-deploy-6c8bb996-nbzvp   0/1     ImagePullBackOff   0          9m23s
pod/echo-deploy-6c8bb996-tg2cn   0/1     ImagePullBackOff   0          9m23s
pod/echo-deploy-6c8bb996-x4qwq   0/1     ImagePullBackOff   0          9m23s
```

- 상태를 보려면 이 명령어를 이용한다.
```
kubectl describe pod/{pod_NAME}
```
-  여러 정보가 뜨는데 그 중 Events를 보면
```
Events:
  Type     Reason     Age                   From               Message
  ----     ------     ----                  ----               -------
  Normal   Scheduled  14m                   default-scheduler  Successfully assigned default/echo-deploy-6c8bb996-56mw5 to minikube
  Warning  Failed     9m1s                  kubelet            Failed to pull image "gchr.io/subicura/echo:v1": rpc error: code = Unknown desc = Error response from daemon: received unexpected HTTP status: 504 Gateway Time-out
  Normal   Pulling    7m14s (x4 over 14m)   kubelet            Pulling image "gchr.io/subicura/echo:v1"
  Warning  Failed     7m9s (x4 over 9m1s)   kubelet            Error: ErrImagePull
  Warning  Failed     7m9s (x3 over 8m40s)  kubelet            Failed to pull image "gchr.io/subicura/echo:v1": rpc error: code = Unknown desc = Error response from daemon: error unmarshalling content: invalid character '<' looking for beginning of value
  Warning  Failed     6m43s (x7 over 9m)    kubelet            Error: ImagePullBackOff
  Normal   BackOff    3m54s (x19 over 9m)   kubelet            Back-off pulling image "gchr.io/subicura/echo:v1"
```
`"gchr.io/subicura/echo:v1": rpc error: code = Unknown desc = Error response from daemon: received unexpected HTTP status: 504 Gateway Time-out"` 경로가 잘못된 듯?

#### 해결 
- `echo-deployment.yml` 파일의 `spec.template.spec.containers.image`의 경로가 `gchr.io/..`로 되어 있었음 -> `gchr.io`로 수정
- `yml`파일 작성이 정상적이었다면 Deployment 생성 속도는 쥰내 빠르다.