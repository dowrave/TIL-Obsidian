## 설치 파일
```sh
git clone -b v1.4.0 https://github.com/kubeflow/manifests.git 

cd manifests
```

## 각 구성 요소 설치
- `kubeflow/manifests` 레포지토리에 설치 커맨드가 적혀 있으나, 정상적으로 설치되었는지 확인하는 방법이 없음 
- 여기선 각 구성 요소 별로 정상적으로 설치되었는지 확인하는 법도 같이 작성함

### Cert-manager
- 설치
```sh
kustomize build common/cert-manager/cert-manager/base | kubectl apply -f -
```

- 정상 설치 시 `~~~ created` 다발로 뜸
- namespace : `cert-manager`의 **3개의 `pod`이 모두 `running`될 때까지 기다림**
```sh
kubectl get pod -n cert-manager
```

- `kubeflow-issuer` 설치
```sh
kustomize build common/cert-manager/kubeflow-issuer/base | kubectl apply -f -
```

### Istio
- `CRD : Custom Resource Definition` 설치 & `istio namespace` 설치
```sh
# CRD 설치
kustomize build common/istio-1-9/istio-crds/base | kubectl apply -f -

# istio namespace 설치
kustomize build common/istio-1-9/istio-namespace/base | kubectl apply -f -
```

- `istio` 설치
```sh
kustomize build common/istio-1-9/istio-install/base | kubectl apply -f -
```

- 2개의 `pod`이 모두 running될 때까지 기다림
```sh
kubectl get po -n istio-system
```

### Dex
- 설치
```sh
kustomize build common/dex/overlays/istio | kubectl apply -f -

# pod 실행 대기
kubectl get po -n auth
```


### OIDC AuthService
- 설치
```sh
kustomize build common/oidc-authservice/base | kubectl apply -f -

# pod 실행 대기
kubectl get po -n istio-system -w
```
- `authservice-0` Pod의 `running`을 기다리자
> 상태가 계속 `pending`
>> `describe pod/authservice-0 -n istio-system`으로 상태를 보면 `1 pod has unbound immediate PersistentVolumeClaim.`이라고 되어 있음

### Kubeflow Namespace
```sh
kustomize build common/kubeflow-namespace/base | kubectl apply -f -

# 조회
kubectl get ns kubeflow
```

### Kubeflow Roles
```sh
kustomize build common/kubeflow-roles/base | kubectl apply -f -

# 조회
kubectl get clusterrole | grep kubeflow
```

### Kubeflow Istio Resources
```sh
kustomize build common/istio-1-9/kubeflow-istio-resources/base | kubectl apply -f -

# 조회
kubectl get clusterrole | grep kubeflow-istio

# gateway 확인 : kubeflow namepsace에 대해
kubectl get gateway -n kubeflow # 출력되면 정상
```

### Kubeflow Pipelines
```sh
kustomize build apps/pipeline/upstream/env/platform-agnostic-multi-user | kubectl apply -f -
```
> 여러 리소스를 한번에 설치하나, 설치 순서의 의존성이 있는 리소스가 있어서 가끔 `unable to recognize "STDIN"` 에러가 발생할 수 있음
>>> 이 경우 10초 정도 기다린 뒤 다시 위의 명령을 수행함

- 정상 설치 확인
```sh
kubectl get po -n kubeflow
```
- 16개 pod의 running을 기다림
- 얘도 실행 한나절임 + 리소스 겁나 잡아먹음 + 여러 pod들 `pending`됨
- 예전에 `K-MOOC`돌릴 땐 잘 돌아갔는디..

- `ml-pipeline UI` 접속 확인
```sh
kubectl port-forward svc/ml-pipeline-ui -n kubeflow 8888:80
```
> [http://localhost:8888/#/pipelines/](http://localhost:8888/#/pipelines/) 에 접속해 `Pipelines`출력 확인
>> 만약 `localhost에서 연결을 거부했습니다`가 뜨면 커맨드로 접근함

- 보안사의 문제가 되지 않을 때만!
```sh
kubectl port-forward --address 0.0.0.0 svc/ml-pipeline-ui -n kubeflow 8888:80
```

- 그럼에도 오류가 발생한다면 
> 방화벽 접속 -> 모든 tcp에 대한 포트 접속 허가 or 8888번 포트의 접속 허가
>> `ttp://<당신의 가상 인스턴스 공인 ip 주소>:8888/#/pipelines/` 에 접속해 보세용

### Katib
```sh
kustomize build apps/katib/upstream/installs/katib-with-kubeflow | kubectl apply -f -

# 설치 확인 : 4개의 pod running 대기
kubectl get po -n kubeflow | grep katib

# UI 접속 확인
kubectl port-forward svc/katib-ui -n kubeflow 8081:80
```

### Central Dashboard
```sh
kustomize build apps/centraldashboard/upstream/overlays/istio | kubectl apply -f -

# pod 1개 running 확인
kubectl get po -n kubeflow | grep centraldashboard

# ui 접속 확인
kubectl port-forward svc/centraldashboard -n kubeflow 8082:80
```
-  [http://localhost:8082/](http://localhost:8082/) 접속 & `Kubeflow Dashboard` 출력 확인

### Admission Webhook
```sh
kustomize build apps/admission-webhook/upstream/overlays/cert-manager | kubectl apply -f -

# pod 1개 running 확인
kubectl get po -n kubeflow | grep admission-webhook
```

### Notebooks & Jupyter Web App
- `Notebooks` 설치
```sh
kustomize build apps/jupyter/notebook-controller/upstream/overlays/kubeflow | kubectl apply -f -

# 설치 확인 (pod 1개)
kubectl get po -n kubeflow | grep notebook-controller
```

- `Jupyter Web App` 설치
```sh
kustomize build apps/jupyter/jupyter-web-app/upstream/overlays/istio | kubectl apply -f -

# pod 1개
kubectl get po -n kubeflow | grep jupyter-web-app
```

### Profiles + KFAM
```sh
kustomize build apps/profiles/upstream/overlays/kubeflow | kubectl apply -f -

# pod 1개
kubectl get po -n kubeflow | grep profiles-deployment
```

### Volumes Web App
```sh
kustomize build apps/volumes-web-app/upstream/overlays/istio | kubectl apply -f -

# pod 1개
kubectl get po -n kubeflow | grep volumes-web-app
```

### Tensorboard & Tensorboard Web App
1. `Tensorboard web app`
```sh
kustomize build apps/tensorboard/tensorboards-web-app/upstream/overlays/istio | kubectl apply -f -

kubectl get po -n kubeflow | grep tensorboards-web-app
```

2. `Tensorboard Controller`
```sh
kustomize build apps/tensorboard/tensorboard-controller/upstream/overlays/kubeflow | kubectl apply -f -

kubectl get po -n kubeflow | grep tensorboard-controller
```

### Training Operator
```sh
kustomize build apps/training-operator/upstream/overlays/kubeflow | kubectl apply -f -

# pod 1개 running
kubectl get po -n kubeflow | grep training-operator
```

### User NameSpace
```sh
kustomize build common/user-namespace/base | kubectl apply -f -

kubectl get profile
```

## 마무리 : 정상 설치 확인
- 접속 : `kubeflow central dashboard`에 웹 브라우저로 접속하기 위해 포트포워딩
```sh
kubectl port-forward svc/istio-ingressgateway -n istio-system 8080:80
```
-  [http://localhost:8080](http://localhost:8080/)으로 접속해 로그인 화면 확인
> `Email Address` : `user@example.com`
> `Password` : `12341234`
