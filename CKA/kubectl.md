# kubectl

```
# 노드에 라벨붙이기
kubectl label nodes 노드이름 라벨키=라벨값
kubectl label nodes node-1 size=Large

# 노드에 라벨 모두 보기
kubectl get node 노드이름 --show-labels
```

```
# 노드에 Taint 설정하기
kubectl taint nodes 노드이름 key=value:테인트효과
kubectl taint nodes node1 app=blue:NoSchedule

# 노드에 설정했던 Taint 제거하기
kubectl taint nodes 노드이름 키-
kubectl taint nodes node1 app-
```

```
# 노드에서 top 명령 실행한 내용보기
kubectl top node

# 파드에서 top 명령 실행한 내용보기
kubectl top pod
```

```
kubectl get ns     # 네임스페이스
kubectl get nodes  # 노드
kubectl get deploy # 디플로이
kubectl get rs     # 리플리카셋
kubectl get po     # 파드
kubectl get ds     # 데몬셋
kubectl get sa     # 서비스 어카운트
```

```
# kube-system 네임스페이스에 파드 확인
kubectl get pods --n kube-system

# static POD 찾기 - 결과에서 파드이름 집미어로 노드이름이 붙어있는 경우가 static POD입니다.
# 예를 들어 파드 이름이 kube-apiserver-controlplane 같이 되어 있습니다.
kubectl get pods --A
```

```
# yml 파일을 사용한 오브젝트 생성
kubectl create -f nginx.yaml

# yml 파일을 이용한 오브젝트 교체 (delete & recreate)
kubectl replace --force -f nginx.yaml
```

```
kubectl create deployment 디플로이이름 --replicas=개수 --image=컨테이너이미지
```

```
# 파드 실행용 yaml 파일 생성
kubectl run 이름 --image=이미지 --dry-run=client -o yaml

# command 옵션 뒤에는 command 절에 넣을 내용만 두고 다른 내용을 두지 않아야 합니다.
kubectl run --restart=Never --image=busybox static-busybox --dry-run=client -o yaml --command -- sleep 1000

# 파드 상태 지켜보기
kubctl get pods --watch
```

```
# 현재 실행중인 파드에서 yaml 형식의 정의 파일 받기
kubectl get pod <파드이름> -o yaml > my-new-pod.yaml

# 받아진 yaml 파일에서 ownerReferences 섹션에 kind가 Node라면 static POD 입니다.
```

```
# selector와 라벨을 이용해서 모든 오브젝트 찾기
kubectl get all --selector env=prod --no-headers

# 여러 라벨을 and 조건으로 적용해서 오브젝트 찾기
kubectl get all --selector env=prod,bu=finance,tier=frontend

# selector와 라벨을 이용해서 원하는 파드 찾기
kubectl get pods --selecotr app=App1
```

```
# 서비스 생성 정의 만들기 (서비스 유형을 생략했으므로 ClusterIP 유형임)
kubectl expose pod <파드이름> --port=<포트번호> --name <서비스이름>
kubectl expose pod redis --port=6379 --name redise-service --dry-run=client -o yaml

# 서비스 생성 정의 만들기 (서비스 유형을 NodePort로 명시함)
kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml

# selector 없이 서비스 생성 정의 만들기 (ClusterIP 유형)
kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml

# selector 없이 서비스 생성 정의 만들기 (NodePort 유형)
kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
```

```
# 이벤트 확인하기
kubectl get events -o wid
```

```
# 로그 확인하기
kubectl logs <오브젝트이름(주로 파드이름)>
kubectl logs my-custom-scheduler --name-space=kube-system
```

```
# 롤아웃 상태 보기
kubectl rollout status deployment/myapp-deployment

# 롤아웃 이력보기
kubectl rollout history deployment/myapp-deployment 

# 롤아웃 작업을 롤백하기
kubectl rollout undo deployment/myapp-deployment
```

```
# 디플로이먼트에서 사용하는 파드 이미지 변경
kubectl set image deployment/myapp-deployment nginx-container=nginx:1.7.1
```

```
# 현재 context 업데이트
# /root/my-kube-config 파일에 있는 prod-user@production 컨텍스트 사용
# prod-user@production는 production 클러스터에 prod-user 사용자를 의미합니다.
kubectl config use-context prod-user@production --kubeconfig /root/my-kube-config
```

```
# kube-apiserver로 (인증없이) 접근 가능한 포트 노출(8001)
# kube proxy <> kubectl proxy 둘은 다른 것임!
kubectl proxy

# kubernetes cluster 내 정적 자원(deployments, pods)에 대한 포트 노출
kubectl port-forward
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
