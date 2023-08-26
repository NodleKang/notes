# Taints

Node에 적용하는 겁니다.

Node에 특정 조건을 부여하여 Node에 실행되는 파드의 선택을 제어하는 매커니즘

taint-effect의 종류
* NoSchedule: 이 효과를 가진 노드는 파드를 스케줄링하지 않습니다.
* PreferNoSchedule: 이 효과를 가진 노드는 해당 테인트를 가진 파드를 스케줄링하려고 하지만, 다른 노드에 스케줄링하는 것도 허용됩니다.
* NoExecute: 특정 조건을 충족하지 않은 파드가 노드에서 계속 실행되는 것을 막습니다.

```
# Node에 Taint 설정하기
kubectl taint nodes 노드이름 키=값:테인트효과
kubectl taint nodes node1 app=blue:NoSchedule

# Node에 설정했던 Taint 제거하기
kubectl taint nodes 노드이름 키-
kubectl taint nodes node1 app-
```

# Tolerations

Pod에 적용하는 겁니다.
Pod에 특정 taints를 가진 노드에도 배포될 수 있게 하는 tolerations를 설정할 수 있습니다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    name: myapp-pod
spec:
  containers
  - name: nginx-container
    image: nginx
  tolerations:
  - key: app
    operator: Equal
    value: blue
    effect: Noschedule
```
