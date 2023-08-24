# Pod

```yaml
# pod-definition.yml

apiVersion: v1
kind: Pod
metadata: 
  name: nginx
  labels:
    name: nginx
spec:
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 8080
  nodeName: node02
```

## nodeName

nodeName 속성이 설정되어 있으면 scheduler가 없어도 지정된 노드에 포드를 할당합니다.

## 포드 이동

한 노드에서 실행중인 포드는 다른 노드로 `move`할 수 없습니다.

## scheduler

scheduler는 컨테이너화된 어플리케이션을 클러스터 내의 노드에 배치하는 역할을 합니다.

노드를 모니터링할 scheduler가 없으면 포드는 계속 Pending 상태에 있게 됩니다.

### Pod binding

"포드를 바인딩한다"는 것은 포드를 특정 노드에 할당하고, 해당 노드에서 포드를 실행할 수 있도록 설정하는 것을 의미합니다.

포드 바인딩은 다음과 같은 단계를 포함합니다.

1. **scheduling**: 포드가 생성되면 스케줄러가 해당 포드를 어떤 노드에 배치할지 결정합니다.
2. **binding**: 스케줄러가 포드를 실행할 노드가 결정하면, 포드는 그 노드에 **바인딩** 됩니다. 즉, 포드와 노드 간에 연결이 설정되고 포드가 해당 노드에서 실행될 수 있도록 준비됩니다.
3. **execution**: 포드가 바인딩된 노드에서 컨테이너화된 어플리케이션을 실행합니다. 이 단계에서 포드의 컨테이너 이미지가 다운로드되고, 컨테이너가 실행되어서 어플리케이션이 동작합니다.

```yaml
# pod-bind-definition.yml

apiVersion: v1
kind: Pod
metadata:
  name: nginx
target:  # spec 절이 없고, target 절 사용
  apiVersion: v1
  kind: Node
  name: node02
```
