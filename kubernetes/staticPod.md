# Static POD

## Static POD 정의 파일 경로 찾기

```
# kubelet 실행 프로세스 찾기
ps aux | grep kubelet | grep config

# kubelet 프로세스의 --config 옵션에 설정된 파일 경로 확인
root        4498  0.0  0.0 3849748 98124 ?       Ssl  03:11   0:19 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock --pod-infra-container-image=registry.k8s.io/pause:3.9

# --config 옵션에 설정된 구성 파일 내용 확인
grep -i static /var/lib/kubelet/config.yaml
```

## Static POD 정의 파일 추가

```
# static POD를 위한 정의 파일이 있어야 할 경로로 이동
cd /etc/kubernetes/manifests

# 해당 경로 내에서 static POD 생성용 정의 파일 작성
kubectl run --restart=Never --image=busybox static-busybox --dry-run=client -o yaml --command -- sleep 1000 > static.yaml
```

## Static POD 생성과 삭제

**생성**: static POD를 위한 정의 파일이 있어야 할 경로에 yaml 파일을 두면 자동으로 파드가 생성됩니다.
**삭제**: static POD를 위한 정의 파일이 있어야 할 경로에 yaml 파일을 삭제해야 파드가 삭제됩니다.
