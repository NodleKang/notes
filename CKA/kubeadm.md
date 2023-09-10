# kubeadm 으로 클러스터 구성하기

## 컨테이너 런타임 설치

모든 노드에서 containerd를 설치하는데, cgroup 드라이버를 동일하게 맞춰야 합니다.

일반적으로 cgroup 드라이버의 기본값은 cgroupfs v2 이지만, systemd도 많이 사용합니다.

sudo vi /etc/containerd/config.toml


* Tip
```
# init 프로세스 정보 표시(보통 cgroupfs v2 아니면 systemd 입니다)
ps -p 1
```

## kubeadm, kubelet, kubectl 설치

모든 노드에서 kubeadm을 설치합니다. (아마도 kubelet, kubectl 도 설치합니다.)

### kubeadm

kubeadm은 클러스터 부트스트랩을 책임지는 도구입니다.


- 목적: kubeadm은 Kubernetes 클러스터를 시작하고 관리하기 위한 도구입니다. 초기화, 노드 추가 및 삭제, 업그레이드 등의 클러스터 관리 작업을 단순화하고 자동화합니다.

- 주요 특징
  - 새로운 Kubernetes 클러스터 초기화
  - 클러스터 구성 파일 생성
  - 마스터 노드와 워커 노드 간의 통신 설정
  - TLS 인증서 생성 및 관리

### kubelet

- 목적: kubelet은 각 노드에서 실행되는 Kubernetes 에이전트로, 클러스터에서 컨테이너를 관리하고 유지하는 역할을 합니다. 노드 상태를 마스터 노드로 보고하고 컨테이너 런타임과 상호 작용하여 Pod를 관리합니다.

- 주요 특징:
  - Pod 및 컨테이너 실행과 관리
  - 리소스 할당 및 노드 상태 보고
  - 정책 및 설정을 준수하여 컨테이너 시작 및 중지

### kubectl

- 목적: kubectl은 Kubernetes 클러스터와 상호 작용하기 위한 커맨드 라인 도구입니다. 클러스터 관리, 애플리케이션 배포, 디버깅, 로깅 등 다양한 작업을 수행할 수 있습니다.

- 주요 특징:
  - 클러스터 구성 관리
  - Pod 및 서비스 배포
  - 로그 및 이벤트 확인
  - 디버깅 및 모니터링

## control plane 노드를 초기화 합니다.

kubeadm init 명령을 master 노드에서만 실행합니다.

