파드와 서비스의 IP범위는 겹치는 부분이 없어야 한다.

```sh
# 파드의 IP 확인
kubectl get pods -o wide


# 서비스의 IP 확인
$ kubectl get service
NAME  READY  STATUS   RESTARTS  AGE  IP          NODE
db    1/1    Running  0         14h  10.244.1.2  node-1

# apiserver에 지정한 서비스를 위한 클러스터IP 범위
$ kube-apiserver --service-cluster-ip-range ipNet
NAME        TYPE       CLUSTER-IP      PORT(S)   AGE
db-service  ClusterIP  10.103.132.104  3306/TCP  12h
```

```sh
# iptables를 확인해보면 NAT 
iptables -L -t nat | grep db-service
KUBE-SVC-XA50GUC7YRHOS3PU tcp -- anywhere  10.103.132.104  /* default/db-service: cluster IP */ tcp dpt:3306
DNAT                      tcp -- anywhere  anywhere        /* default/db-service: */ tcp to:10.244.1.2:3306
```

```sh
cat /var/log/kube-proxy.log
```

### 244. Practice Test - Service Networking

1. 이 클러스터의 `Node`가 사용할 IP 범위를 찾으시오.

```sh
# INTERNAL-IP 확인
$ kubectl get nodes -o wide
NAME           STATUS   ROLES           AGE    VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
controlplane   Ready    control-plane   144m   v1.27.0   192.6.216.3   <none>        Ubuntu 20.04.5 LTS   5.4.-1106-gcp   containerd://1.6.6
node01         Ready    <none>          144m   v1.27.0   192.6.216.6   <none>        Ubuntu 20.04.5 LTS   5.4.0-1106-gcp   containerd://1.6.6

# 앞서 확인한 INTERNAL-IP와 같은 범위에 있는 인터페이스를 찾습니다.
$ ip addr | grep 192.6
2721: eth0@if2722: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1450 qdisc noqueue state UP group default 
    link/ether 02:42:c0:06:d8:03 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.6.216.3/24 brd 192.6.216.255 scope global eth0
```

2. 이 클러스터에 구성되는 `Pods`가 사용할 IP 범위를 찾으시오.

```sh
# 모든 파드를 확인하고, weave용 파드가 있는 것을 확인합니다.
$ kubectl get po -A
...
NAMESPACE     NAME                                   READY   STATUS    RESTARTS       AGE
kube-system   weave-net-fdlgj                        2/2     Running   1 (154m ago)   154m
kube-system   weave-net-pkdfv                        2/2     Running   1 (154m ago)   154m

# weave를 사용하는 것을 확인했으니, weave용 파드 로그에서 ipalloc-range 값을 찾아서 파드에게 할당될 IP 범위를 확인합니다.
$ kubectl logs weave-net-fdlgj -n kube-system | grep ipalloc-range
Defaulted container "weave" out of: weave, weave-npc, weave-init (init)
INFO: 2023/09/10 05:00:21.158129 Command line options: map[conn-limit:200 datapath:datapath db-prefix:/weavedb/weave-net docker-api: expect-npc:true http-addr:127.0.0.1:6784 ipalloc-init:consensus=1 ipalloc-range:10.244.0.0/16 metrics-addr:0.0.0.0:6782 name:de:c7:5a:f5:78:83 nickname:controlplane no-dns:true no-masq-local:true port:6783]
```

3. 이 클러스터에 구성될 `Service`가 사용할 IP 범위를 찾으시오.

```sh
# 서비스가 사용할 IP범위는 apiserver에 의해서 결정되므로 apiserver의 옵션을 확인합니다.
$ ps aux | grep apiserver | grep ip-range
root        3566  0.0  0.1 1107448 321048 ?      Ssl  00:59   6:34 kube-apiserver ...다른옵션들... --service-cluster-ip-range=10.96.0.0/12
```

5. kube-proxy가 사용하는 프록시 타입을 찾으시오.

```sh
# kube-proxy 파드 로그에서 proxy 타입을 확인합니다.
$ kubectl logs kube-proxy-94kdk -n kube-system | grep -i proxy
I0910 05:00:12.751075       1 server_others.go:551] "Using iptables proxy"
I0910 05:00:12.854202       1 server_others.go:197] "kube-proxy running in dual-stack mode" ipFamily=IPv4
```

6. 클러스터에서 kube-proxy 파드가 모든 노드에서 실행되는 걸 보장하는 방법을 확인합니다.

```sh
# 클러스터에 있는 각 노드에 실행되는 걸 보장하는 방법이 daemonSet이라는 걸 이미 알아야 합니다.
# deamonset을 확인하면 kube-proxy가 있는걸 확인합니다.
$ kubectl get daemonset -A
NAMESPACE     NAME         DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   kube-proxy   2         2         2       2            2           kubernetes.io/os=linux   172m
kube-system   weave-net    2         2         2       2            2           <none>                   172m
```

### DNS 주소 규칙

* 형식: <hostname>.<namespace>.<type>.<root>
* 예1: web-service.apps.svc.clsuter.local
* 예2: 10-244-2-5.apps.pod.cluster.local
* 예3: 10-244-1-5.default.pod.cluster.local


### 229. Practice Test - Explore DNS

1. 클러스터에 구현된 DNS 솔루션을 확인하시오.

```sh
# coredns 이름이 들어 있는 파드를 확인합니다.
$ kubectl get all -A
```

4. CoreDNS 서버의 IP를 확인하시오.

```sh
# 파드 자체의 IP가 아니라 CoreNDS에 접근하기 위한 서비스의 IP를 확인해야 합니다.
$ kubectl get svc -n kube-system -o wide
NAME       TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE   SELECTOR
kube-dns   ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   11m   k8s-app=kube-dns
```

5. CoreDNS 서비스를 위한 config 파일 위치를 찾으시오.

```sh
# CoreDNS용 파드 상세보기에서 "-conf" Args 찾기
$ kubectl describe po coredns-5d78c9869d-4pq8t -n kube-system
... 생략 ...
Containers:
  coredns:
    Container ID:  containerd://c93c0cb1ba6a966c540e63b087e423685887a0118a44e7fc7c5aaf147e1db77e
    Image:         registry.k8s.io/coredns/coredns:v1.10.1
    Image ID:      registry.k8s.io/coredns/coredns@sha256:a0ead06651cf580044aeb0a0feba63591858fb2e43ade8c9dea45a6a89ae7e5e
    Ports:         53/UDP, 53/TCP, 9153/TCP
    Host Ports:    0/UDP, 0/TCP, 0/TCP
    Args:
      -conf
      /etc/coredns/Corefile
... 생략 ...
```

8. ConfigMap에서 root domain/zon을 찾으시오.

```sh
# configMap 상세보기에서 해당 부분을 찾습니다.
# 여기서는 cluster.local 입니다.
$ kubectl describe configmap coredns -n kube-system
Name:         coredns
Namespace:    kube-system
Labels:       <none>
Annotations:  <none>

Data
====
Corefile:
----
.:53 {
    errors
    health {
       lameduck 5s
    }
    ready
    kubernetes cluster.local in-addr.arpa ip6.arpa {
       pods insecure
       fallthrough in-addr.arpa ip6.arpa
       ttl 30
    }
    prometheus :9153
    forward . /etc/resolv.conf {
       max_concurrent 1000
    }
    cache 30
    loop
    reload
    loadbalance
}


BinaryData
====

Events:  <none>
```

10. default 네임스페이스에 있는 test파드가 hr파드에 curl로 접근하시오.

```sh
# default 네임스페이스에 있는 파드와 서비스를 확인합니다.
$ kubectl get all -n default
NAME                    READY   STATUS    RESTARTS   AGE
pod/hr                  1/1     Running   0          20m
pod/test                1/1     Running   0          20m

NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
service/kubernetes     ClusterIP   10.96.0.1      <none>        443/TCP        24m
service/test-service   NodePort    10.109.161.8   <none>        80:30080/TCP   20m
service/web-service    ClusterIP   10.97.135.3    <none>        80/TCP         20m

# 유력해보이는 web-service 서비스 상세보기
$ kubectl describe svc web-service
Name:              web-service
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          name=hr # 선택기=hr
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.97.135.3
IPs:               10.97.135.3
Port:              <unset>  80/TCP # 포트=80
TargetPort:        80/TCP
Endpoints:         10.244.0.5:80
Session Affinity:  None
Events:            <none>

# test 파드에서 hr에 접근하기
user@test$ curl web-service:80
```

14. 새로 배포한 webapp이 mysql에 접근할 수 있도록 하라. 

```sh
# deployment를 확인합니다.
$ kubectl get deploy webapp -o wide
NAME     READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS            IMAGES                         SELECTOR
webapp   1/1     1            1           8m44s   simple-webapp-mysql   mmumshad/simple-webapp-mysql   name=webapp

# webapp deployment로 배포된 파드를 확인합니다.
$ kubectl get po -A -o wide | grep webapp
default        webapp-54b76556d-x5hj8                 1/1     Running   0          10m   10.244.0.9    controlplane   <none>           <none>

# mysql 파드를 확인합니다.
$ kubectl get po mysql -n payroll -o wide
NAME    READY   STATUS    RESTARTS   AGE   IP            NODE           NOMINATED NODE   READINESS GATES
mysql   1/1     Running   0          11m   10.244.0.10   controlplane   <none>           <none>

# 앞의 과정에서 webapp과 mysql의 네임스페이스가 다른 것을 확인했습니다.

# webapp 파드 상세보기에서 DB 관련 환경정보를 확인합니다.
$ kubectl describe po webapp-54b76556d-x5hj8 | grep DB_ -B1
    Environment:
      DB_Host:      mysql
      DB_User:      root
      DB_Password:  paswrd

# webapp deployment에서 DB_Host를 mysql.payroll로 변경해줍니다.
$ kubectl edit deploy webapp
... 생략 ...
    spec:
      containers:
      - env:
        - name: DB_Host
          value: mysql.payroll # 네임스페이스 정보 붙이기
... 생략 ...

# deployment가 변경되어서 자동으로 파드들이 재배포됩니다.
```

15. mysql과 다른 네임스페이스에 있는 hr에서 mysql을 대상으로 한 nslookup 결과를 파일로 남기시오.

```sh
# 네임스페이스가 다르므로 <파드.네임스페이스> 형식으로 nslookup을 해야 합니다.
$ kubectl exec hr -- nslookup mysql.payroll > /root/CKA/nslookup.out

$ cat /root/CKA/nslookup.out
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   mysql.payroll.svc.cluster.local
Address: 10.108.212.99
```