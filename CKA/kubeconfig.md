# config 확인

```
# 현재 사용중인 config 확인
# 기본 config 위치인 $HOME/.kube/config 사용
kubectl config view

# 특정 config 확인
kubectl config view --kubeconfig=my-custom-conf
```

# 기본 config 변경

기본 config 파일인 `$HOME/.kube/config` 파일 내용을 수정합니다.

# context

`kubectl config` 명령이 바꾼건 config 파일에 반영됩니다.

```
# 현재 context 업데이트
# production 클러스터에 prod-user 사용자로 연결하도록 컨텍스트 변경
kubectl config use-context prod-user@production

# /root/my-kube-config 파일에 있는 research 컨텍스트 사용
kubectl config use-context research --kubeconfig /root/my-kube-config

# config 파일을 확인해서 current_context가 research로 설정됐는지 확인
cat /root/my-kube-config | grep current
current-context: research
```
