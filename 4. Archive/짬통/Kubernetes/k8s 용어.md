![[Pasted image 20221221155644.jpg]]

### 클러스터
- 쿠버네티스의 여러 리소스를 관리하기 위한 집합체.

### 노드
- 쿠버네티스 리소스 중 가장 큰 개념. 
- **클러스터의 관리 대상**으로 등록된 **도커 호스트**로, **도커 컨테이너가 배치**된다.
- 쿠버네티스 **클러스터 전체를 관리하기 위한 마스터 노드 1개 이상이 반드시** 있어야 한다.(실제 환경에선 최소 3개 이상의 마스터 노드를 둔다.)
- 쿠버네티스는 노드의 리소스 사용 현황, 배치 전략 등을 근거로 컨테이너를 적절히 배치한다.

#### 마스터노드의 관리 컴포넌트
- `kube-apiserver` : 쿠버네티스 api 호출하는 컴포넌트로, `kubectl`로부터 리소스 조작 지시를 받는다.
- `etcd`  : 고가용성을 가준 분산 키-값 스토어, 클러스터의 백킹 스토어로 쓰임
- `kube-scheduler` : 노드 모니터링 & 컨테이너 배치할 노드 선택
- `kube-controller-manager` : 리소스 제어 컨트롤러

### 네임스페이스
- 쿠버네티스는 `클러스터 내부에 가상 클러스터`를 만들 수 있는데, 이를 `네임스페이스`라고 한다.
- 클러스터 구축 시 `default`, `docker`, `kube-public`, `kube-system`이라는 4개의 네임스페이스가 이미 있다.
```sh
kubectl get namespace
```
- 전체 클러스터에서 리소스의 구분 용도라고 생각해도 무방하다. 즉 특정 이름으로 클러스터의 영역을 구분하는 것이다.

### Pod
- 컨테이너가 모인 집합체 단위로, 최소 1개 이상의 컨테이너로 구성되어 있다.(`도커 컨테이너`를 의미함)
- 쿠버네티스에서는 결합이 강한 컨테이너를 Pod으로 묶어 일괄 배포한다(`spring` + `nginx`)
- 또, **1개의 Pod은 1개의 Node에 배치**된다. 즉 같은 Pod 내부의 다른 컨테이너들은 다른 Node에 배치되지 못한다.

#### Pod 생성 및 배포
- `kubectl`로 생성이 가능하나, 버전 관리 관점에서 `yaml`파일로 정의한다. 
- `manifest` 파일 : 쿠버네티스의 여러 리소스를 정의(우리가 작성하는 `yaml`파일이 매니페스트 파일)
- **도커 이미지** : 도커 컨테이너를 구성하는 **파일 시스템 +** 실행할 **어플리케이션 설정**을 하나로 합친 것. 컨테이너를 설정하는 **템플릿** 역할을 한다.
- 도커 컨테이너 : 도커 이미지를 기반으로 생성되며, 파일 시스템과 애플리케이션이 구체화되어 실행됨

- Pod 매니페스트 파일
```yaml
apiVersion: v1 
kind: Pod 
metadata: 
  name: springboot-web 
spec: 
  containers: 
    - name: springboot-web 
      image: 1223yys/springboot-web:0.1.6 
      ports: 
        - containerPort: 8080
```
- `kind` : 쿠버네티스 리소스의 유형 정의
	- `kind`에 따라 `spec` 아래의 스키마가 변화함.
- `metadata` : 리소스에 부여되는 메타 데이터
- `spec` : 리소스 정의 속성. 
	- `Pod`의 경우 컨테이너는 `containers` 내부에 정의된다.
	- `containers`
		- `name` : 컨테이너 이름
		- `image` : 도커 허브에 저장된 이미지 태그값
		- `port` : 외부에 노출시킬 포트 번호

#### ReplicaSet
```yaml
apiVersion: apps/v1 
kind: ReplicaSet
metadata: 
  name: sample-replicaset 
  labels: 
    app: springboot-web 
spec: # replicaset 정보
  replicas: 3 
  selector: 
    matchLabels: 
      app: springboot-web 
  template: # pod 정보
    metadata: 
      labels: 
        app: springboot-web 
    spec: 
      containers: 
        - name: web-app 
          image: 1223yys/springboot-web:0.1.6
          ports: 
          - containerPort: 8080
```
- 먼저 나오는 `spec` : `ReplicaSet`에 대한 정보
	- `replicas` : 복제본의 갯수
	- `selector` : 어떤 `Pod`을 대상으로 `ReplicaSet`을 만들 것인가를 지정함
		- `matchLabels`에는 지정할 Pod의 정보를 입력한다
		- 실제로 예제의 `template.metadata.labels`의 정보와 일치함을 알 수 있다.

### Deployment
```yaml
apiVersion: apps/v1 
kind: Deployment
metadata: 
  name: sample-deployment 
  labels: 
    app: springboot-web 
spec: # Deployment 정보
  replicas: 3 
  selector: 
    matchLabels: 
      app: springboot-web 
  template: # pod 정보
    metadata: 
      labels: 
        app: springboot-web 
    spec: 
      containers: 
        - name: web-app 
          image: 1223yys/springboot-web:0.1.6
          ports: 
          - containerPort: 8080
```
- `ReplicaSet`과 크게 다르지 않으며, `Deployment`는 `ReplicaSet`의 리비전 관리를 할 수 있다.
	- `리비전 관리` : `kubectl rollout history ~~`를 봤을 때, 변경 내역과 내용이 남아 있음
- 배포의 기본 단위는 `Deployment`이다. 

- 롤백, 팟 갯수 수정 등은 다 다뤘기 때문에 넘어감
