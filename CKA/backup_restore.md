# 백업과 복구

## 방법 1 - kube-apiserver 쿼리를 이용한 백업
```
kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml
```

## 방법 2 - etcd를 이용한 방법

```
# etcd 백업
ETCDCTL_API=3 etcdctl snapshot save snaphost.db

service kube-apiserver stop
```

```
# etcd 복구
ETCDCTL_API=3 etcdctl snapshot restore snaphost.db ==data-dir /var/lib/etcd-from-backup

systemctl daemon-reload
service etcd restart
service kube-apiserver start
```

## etcdctl

```
export ETCDCTL_API=3
```

```
# 도움말보기
etcdctl snapshot save -h
etcdctl snapshot restore -h
```

### etcdctl 옵션들
- --cacert : TLS 인증서 파일 확인용 ca 파일
- --cert: TLS 인증서 파일
- --endpoints=[127.0.0.1:2379]: ETCD가 마스터 노드에서 실행되고 localhost의 2379 포트로 노출되기 때문에, 이 옵션은 기본값입니다.
- --key: TLS 키 파일

```
etcdctl snapshot save \
  --cacert="/etc/kubernetes/pki/etcd/ca.crt" \
  --cert="/etc/kubernetes/pki/etcd/server.crt" \
  --key="/etc/kubernetes/pki/etcd/server.key" \
  --endpoints=127.0.0.1:2379 \
  /opt/snapshot-pre-boot.db
```