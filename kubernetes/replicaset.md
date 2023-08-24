# ReplicaSet

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: simple-webapp
  labels:
    app: App1
    function: Front-end
  annotations:  # 리소스에 추가적인 정보 제공
    buildversion: 1.34
spec:
  replicas: 3
  selector: 
    matchLabels:
      app: App1
  template:
    metadata:
      labels:
        app: App1
        frunction: Front-end
    spec:
      containers:
      - name: simple-webapp
        image: simple-webapp
```

## annotations

Annotation은 주로 레이블과 유사하지만, 주로 비즈니스 로직이나 도구에 의해 사용되며, 쿠버네티스 자체에서는 직접적으로 사용되지 않는 데이터를 저장하는 데 사용됩니다.
