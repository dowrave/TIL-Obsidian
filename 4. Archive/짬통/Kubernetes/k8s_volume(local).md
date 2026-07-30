- 도커에서 컨테이너 업데이트 시 내부 데이터가 다 날아갔듯, **k8s에서도 Pod을 제거하면 컨테이너 내부 데이터가 모두 날아간다.**
- 따라서 별도의 저장소에 데이터를 저장하고 새로 컨테이너를 만들 때 이전 데이터를 가져와야 한다.
- 쿠버네티스에선 `Volume`을 이용해 컨테이너의 디렉토리를 외부와 연결하고 다양한 플러그인을 지원하여 대부분의 스토리지를 별도 설정 없이 쓸 수 있다.

- 실무에서는 `awsElasticBlockStore(aws)`, `azureDisk(azure)`, `gcePersistentDisk(Google Cloud)`와 같은 Volume을 쓰지만, 실제 클라우드를 써야 하므로 여기선 **로컬 저장소를 이용**하는 법을 다룬다.

> PV/PVC
> 데이터 저장이 필요한 경우 흔히 `PV:Persistent Volume`, `PVC:Persistent Volume Claim`을 사용한다. 

### Volume 만들기

#### empty_dir
- Pod 안에 속한 컨테이너 간 디렉토리를 공유하는 방법.
![[empty-dir.webp]]
- 보통 `사이드카sidecar`라는 패턴에서 사용한다.
	- 특정 컨테이너에서 수집되는 로그 파일을 `별도의 컨테이너(사이드카)`가 수집할 수 있다.

- `empty-dir.yml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sidecar
spec:
  containers:
    - name: app
      image: busybox
      args:
        - /bin/sh
        - -c
        - >
          while true;
          do
            echo "$(date)\n" >> /var/log/example.log;
            sleep 1;
          done
      volumeMounts:
        - name: varlog
          mountPath: /var/log
    - name: sidecar
      image: busybox
      args: [/bin/sh, -c, "tail -f /var/log/example.log"]
      volumeMounts:
        - name: varlog
          mountPath: /var/log
  volumes:
    - name: varlog
      emptyDir: {}
```
- `app` 컨테이너는 `/var/log/example.log`에 로그 파일을 처리하게 만든다.
- `sidecar` 컨테이너는 해당 로그 파일을 처리하게 한다.
```sh
kubectl apply -f empty-dir.yml

# 로그 확인
kubectl logs -f sidecar -c sidecar
```
- 결과 : 1초마다 로그가 계속 찍히는 걸 확인할 수 있음(`app` 컨테이너의 로그 파일을 `sidecar` 컨테이너에서 처리함)

#### hostpath
- 호스트 디렉토리 -> 컨테이너 디렉토리에 연결하는 방법
- 호스트의 `/var/log` 디렉토리를 연결한다.
![[hostpath.webp]]
- `hostpath.yml`
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: host-log
spec:
  containers:
    - name: log
      image: busybox
      args: ["/bin/sh", "-c", "sleep infinity"]
      volumeMounts:
        - name: varlog
          mountPath: /host/var/log
  volumes:
    - name: varlog
      hostPath:
        path: /var/log
```

- 컨테이너에서 마운트 디렉토리 확인
```sh
$ kubectl apply -f hostpath.yml

$ kubectl exec -it host-log -- sh

$ ls -al /host/var/log
```
- 여기서 **Host = Node**이다.
- Host(Node) 내에 Pod이 있고, Pod 내에 Container가 있음. 일단은 이런 포장 개념으로 알고 있자.

