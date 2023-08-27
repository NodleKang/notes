# Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: myapp
    type: front-end
spec:
  replicas: 3
  selector:
    matchLabels:
      type: front-end
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
      - name: nginx-container
        image: nginx:1.7.1
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
              nodeSelectorTerms:
              - matchExpressions:
                - key: color
                  operator: In
                  values:
                    - blue
```

## rollout 전략

- Recreate
- RollingUpdate (기본)

## kubectl

### 롤아웃 관련 명령들

```
# 디플로이먼트 생성
kubectl create -f deployment-definition.yml

# 디플로이먼트 상태 보기
kubectl get deployments

# 디플로이먼트 갱신
kubectl apply -f deployment-definition.yml

# 디플로이먼트에 파드 이미지 갱신
kubectl set image deployment/myapp-deployment nginx=nginx:1.9.1

# 롤아웃 상태 보기
kubectl rollout status deployment/myapp-deployment

# 롤아웃 이력보기
kubectl rollout history deployment/myapp-deployment 

# 롤아웃 작업을 롤백하기
kubectl rollout undo deployment/myapp-deployment
```
