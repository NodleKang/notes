# ConfigMap

ConfigMap은 `키-값`으로 config 정보를 전달하는 Map 입니다.

## ConfigMap 정의

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_COLOR: blue
  APP_MODE: prod
```

```
kubectl create configmap <컨피그이름> --from-literal=<키>=<값> [--from-literal=<키>=<값> ...]
kubectl create configmap app-config --from-literal=APP_COLOR=blue \
                                    --from-literal=APP_MOD=prod 
```

```
kubectl create configmap <컨피그이름> --from-file=<파일경로>
kubectl create configmap app-config --from-file=app_config.properties 
```

## ConfigMap 확인

```
kubectl get configmaps

kubectl describe configmaps
```

## ConfigMap 주입

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: myapp
spec:
  containers:
  - name: myapp
    image: myapp-image
    env:
      - name: APP_COLOR
        value: pin-k
      - name: APP_CONFIGMAP
        valueFrom:
          configMapKeyRef:
            name: app-config
            key: APP_COLOR
      - name: APP_SECRETKEY
        valueFrom:
          secretKeyRef:
    envFrom:
      - configMapRef:
          name: app-config
```
