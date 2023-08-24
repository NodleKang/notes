# kubectl

```
# 네입스페이스 확인
kubectl get ns
```

```
# kube-system 네임스페이스에 포드 확인
kubectl get pods --n kube-system
```

```
# yml 파일을 사용한 리소스 생성
kubectl create -f nginx.yaml

# yml 파일을 이용한 리소스 교체 (delete & recreate)
kubectl replace --force -f nginx.yaml
```

```
# selector와 라벨을 이용해서 원하는 리소스 찾기
kubectl get pods --selecotr app=App1
```
