# Multiple Scheduler

## 파드로 추가적인 Scheduler 배포하기

```yaml
# 스케줄러 구성
# my-scheduler-config.yaml
apiVersion: kubescheduler.config.k8s.io/v1
kind: KubeSchedulerConfiguration
profiles:
  - schedulerName: my-scheduler
  leaderElection:
    leaderElect: true
    resourceNamespace: kube-system
    resourceName: lock-object-my-scheduler
```

```yaml
# 스케줄러용 파드 정의
apiVersion: v1
kind: Pod
metadata:
  name: my-custom-scheduler
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-scheduler
    - --address=127.0.0.1
    - --kubeconfig=/etc/kubernetes/scheduler.conf
    - --config=/etc/kubernetes/my-scheduler-config.yaml
    image: k8s.gcr.io/kube-scheduler-amd74:v1.11.3
    name: kube-scheduler
```

```
# 이벤트 확인하기
kubectl get events -o wid
```

```
# 로그 보기
kubectl logs my-custom-scheduler --name-space=kube-system
```

```
kubectl create configmap <컨피그맵이름> --from-file=<yaml파일이름> -n <네임스페이스>
```

```yaml
# my-scheduler-configmap.yaml
apiVersion: v1
data:
  my-scheduler-config.yaml: |
    apiVersion: kubescheduler.config.k8s.io/v1beta2
    kind: KubeSchedulerConfiguration
    profiles:
      - schedulerName: my-scheduler
    leaderElection:
      leaderElect: false
kind: ConfigMap
metadata:
  creationTimestamp: null
  name: my-scheduler-config
  namespace: kube-system
```
