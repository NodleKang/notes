# 233. Practice Test - Ingress - 1

6. Ingress 리소스가 디플로이되어 있는 네임스페이스를 찾으시오.

```sh
$ kubectl get ingress -A
NAMESPACE   NAME                 CLASS    HOSTS   ADDRESS        PORTS   AGE
app-space   ingress-wear-watch   <none>   *       10.99.59.172   80      13m
```

22. /pay 경로에 대응되는 ingress를 생성하시오.

```sh
# Deploy를 확인합니다.
$ kubectl get deploy -A
NAMESPACE        NAME                       READY   UP-TO-DATE   AVAILABLE   AGE
critical-space   webapp-pay                 1/1     1            1           5m39s
ingress-nginx    ingress-nginx-controller   1/1     1            1           30m

$ kubectl describe deploy webapp-pay -n critical-space
Name:                   webapp-pay
Namespace:              critical-space
CreationTimestamp:      Sun, 10 Sep 2023 22:24:27 -0400
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=webapp-pay
... 생략 ...

# 같은 네임스페이스에 있는 Deploy와 연계된 서비스에서 서비스하는 포트를 확인합니다.
$ kubectl get svc -n critical-space
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
pay-service   ClusterIP   10.101.52.192   <none>        8282/TCP   20m

# ingress 생성 문법을 참고합니다.
$ kubectl create ingress --help

# Ingress 생성합니다.
$ kubectl create ingress ingress-pay -n critical-space --rule="/pay=pay-service:8282"
```


# 233. Practice Test - Ingress - 2

7. /wear 및 /watch를 수신하는 ingress를 생성하시오. 
rewrite-target 어노테이션 필드를 사용해야 합니다.
```
nginx.ingress.kubernetes.io/rewrite-target: /
```
ingress 리소스는 app-space 네임스페이스에 생성해야 합니다.

```sh
$ cat in.yml 
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ngress-wear-watch
  namespace: app-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    nginx.ingress.kubernetes.io/ssl-redirect: "false"
spec:
  rules:
  - http:
      paths:
      - backend:
          service:
            name: wear-service
            port:
              number: 8080
        path: /wear
        pathType: Prefix
      - backend:
          service:
            name: video-service
            port:
              number: 8080
        path: /wear
        pathType: Prefix
status:
  loadBalancer: {}
```