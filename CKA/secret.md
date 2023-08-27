# Secret

# Secret 정의

```
# base64 인코딩 적용해서 나온 내용을 secret에 사용해야 합니다.
echo -n 'mysql' | base64

# base64를 디코딩
echo -n 'bXlzWw=' | base64 --decode
```

```
kubectl create secret generic <시크릿이름> --from-literal=<키>=<값>
kubectl create secret generic app-secret --from-literal=DB_HOST=mysql

kubectl create secret generic <시크릿이름> --from-file=<파일경로>
```

## Secret 확인

```
kubectl get secrets

kubectl describe secrets
```

## Secret 주입

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
      - name: DB_password
        valueFrom:
          secretKeyRef:
            name: app-secret
            key: DB_password
    envFrom:
      - configMapRef:
          name: app-config
      - secretRef:
          name: app-config
    volumes:
      - name: app-secret-volume
        secret:
          secretName: a
```

## 암호화

```yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
    - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: <BASE 64 ENCODED SECRET>
      - identity: {}
```
