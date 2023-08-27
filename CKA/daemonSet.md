# DaemonSet

데몬셋(DaemonSet)은 각 노드에서 정확히 하나의 파드 인스턴스를 실행하는 컨트롤러 유형입니다.

## DaemonSet vs Deployment

|  | DaemonSet | Deployment |
|--------------|--------------|--------------|
| 파드 배치 전략 | 각 노드에 하나의 파드 인스턴스 | 노드에 상관없이 여러 개의 인스턴스 배포 |
| 실행 단위 | 노드 단위 | 노드와 관계 없이 실행/관리 |
| 용도 | 각각의 노드에서 특정 작업을 제공해야 할 때 | 어플리케이션 배포, 스케일링 등 라이프사이클 관리 |
| 업데이트 전략 | 보통 노드를 업데이터할 때 | 보통 롤링 업데이트 |

## 주요 특징
- 모든 노드에서 실행됩니다.
- 노드가 클러스터에 추가되거나 제거될 때마다 자동으로 추가되거나 제거됩니다.
- 각 노드에 하나씩 고유한 인스턴스를 유지합니ㅏㄷ.

## 정의 샘플

```
# ReplicaSet이나 Deployment 정의와 거의 같은 구조
apiVerion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-daemon
spec:
  selector:
    matchLabels:
      app: monitoring-agent
  template:
    metadata:
      labels:
        app: monitoring-agent
    spec:
      containers:
      - name: monitoring-agent
        image: monitoring-agent
```

## 사용처

- 예: kube-proxy
- 모니터링 에이전트
- 로깅 에이전트
