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
kubectl get ns     # 네임스페이스
kubectl get nodes  # 노드
kubectl get deploy # 디플로이
kubectl get rs     # 리플리카셋
kubectl get po     # 파드
kubectl get ds     # 데몬셋
```

```
# kube-system 네임스페이스에 파드 확인
kubectl get pods --n kube-system
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

# 파드 상태 지켜보기
kubctl get pods --watch
```

```
# 현재 실행중인 파드에서 yaml 형식의 정의 파일 받기
kubectl get pod <파드이름> -o yaml > my-new-pod.yaml
```

```
# selector와 라벨을 이용해서 모든 오브젝트 찾기
kubectl get all --selector env=prod --no-headers

# 여러 라벨을 and 조건으로 적용해서 오브젝트 찾기
kubectl get all --selector env=prod,bu=finance,tier=frontend

# selector와 라벨을 이용해서 원하는 파드 찾기
kubectl get pods --selecotr app=App1
```
