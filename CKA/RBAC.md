# RBAC

## 특징

Role과 RoleBinding은 네임스페이스 생성됩니다.

Namespace 안에서 관리되는 리소스들: pods, replicasets, jobs, deployments, services, secrets, roles, rolebindings, configmaps, PVC

Cluster Wide(Cluster Scope) 레벨로 관리되는 리소스들: nodes, PV, clusterroles, clusterrolebindings, certificatesigningrequests, namespaces

Cluster Wide 혹은 Cluster Scope 범위는?

default 네임스페이스에 파드를 읽을 수 있는 롤 생성 샘플

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]  
```

secrets에 대한 읽기 액세스 권한을 부여하는 롤 생성 샘플

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # "namespace" omitted since ClusterRoles are not namespaced
  name: secret-reader
rules:
- apiGroups: [""]
  #
  # at the HTTP level, the name of the resource for accessing Secret
  # objects is "secrets"
  resources: ["secrets"]
  verbs: ["get", "watch", "list"]  
```

롤 바인딩 샘플

```yaml
apiVersion: rbac.authorization.k8s.io/v1
# This role binding allows "jane" to read pods in the "default" namespace.
# You need to already have a Role named "pod-reader" in that namespace.
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
# You can specify more than one "subject"
- kind: User
  name: jane # "name" is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  # "roleRef" specifies the binding to a Role / ClusterRole
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```

```
# 롤 현황 보기
kubectl get roles

# 롤바인딩 현황 보기
kubectl get rolebindings

# 롤 상세보기
kubectl describe role <롤이름>

# 롤바인딩 상세보기
kubectl describe rolebinding <롤바인딩이름>
```

```
# 특정 작업을 할 권한이 있는지 검사하는 명령들
kubectl auth can-i create deployment
kubectl auth can-i delete nodes
kubectl auth can-i create deployments --as dev-user
kubectl auth can-i create pods --as dev-user
kubectl auth can-i create pods --as dev-user --namespace test
```

```
# 롤 생성
kubectl create role --verb=list,create,delete --resources=pods -n default

# 현재 사용자 컨텍스트 확인 (현재 로그인한 사용자 정보를 알 수 있음)
kubectl config current-context

# 유저로서 명령 실행하기
kubectl --as dev-user get pod dark-blue-app -n blue
```

```
# 롤 바인딩 생성
kubectl create rolebinding dev-user-binding --role=developer --user=dev-user -n default
```

```
kubectl api-resources --namespaced=true

# Kubernetes 클러스터에서 사용 가능한 API 리소스를 나열하는 명령
# 네임스페이스가 아닌 (클러스터 전체 범위의) 리소스
kubectl api-resources --namespaced=false
```

```
# 서비스 어카운트 생성
kubectl create serviceaccount dashboard-sa

# 롤바인딩 목록
kubectl get rolebindings

# 롤바인딩 상세에서 서비스 어카운트 설정 상태 확인
kubectl describe rolebindings read-pods

# 토큰 생성
kubectl create token dashboard-sa
```