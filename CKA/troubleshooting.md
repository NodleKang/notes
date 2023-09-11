# 섹션13: Trubleshooting

## 253. Practice Test - Application Failure

https://uklabs.kodekloud.com/topic/practice-test-application-failure-2/


### 1. 문제
간단한 2-tier 애플리케이션이 alpha 네임스페이스에 배포됩니다. 성공하면 녹색 웹 페이지가 표시되어야 합니다. 문제를 해결하고 수정하세요.

```
웹페이지에 접속하면 아래와 같이 "Name does not resolve" 문제 때문에 DB_Host에 접속하지 못 하고 있습니다.
Environment Variables: DB_Host=mysql-service; DB_Database=Not Set; DB_User=root; DB_Password=paswrd; 2003: Can't connect to MySQL server on 'mysql-service:3306' (-2 Name does not resolve)
From webapp-mysql-5456999f7b-jd87m!
```

### 1. 답안

```sh
# 현재 리소스들을 확인합니다.
kubectl get all -n alpha
NAME                                READY   STATUS    RESTARTS   AGE
pod/webapp-mysql-5456999f7b-jd87m   1/1     Running   0          15m
pod/mysql                           1/1     Running   0          15m

NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/mysql         ClusterIP   10.43.56.140    <none>        3306/TCP         15m
service/web-service   NodePort    10.43.148.107   <none>        8080:30081/TCP   15m

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/webapp-mysql   1/1     1            1           15m

NAME                                      DESIRED   CURRENT   READY   AGE
replicaset.apps/webapp-mysql-5456999f7b   1         1         1       15m
```

```sh
# 웹어플리케이션 상세 정보에 설정된 DB_Host가 mysql-service인 것을 확인합니다.
$ kubectl describe po webapp-mysql-5456999f7b-jd87m -n alpha
Name:             webapp-mysql-5456999f7b-jd87m
Namespace:        alpha
...
Containers:
  webapp-mysql:
    ...
    Environment:
      DB_Host:      mysql-service
      DB_User:      root
      DB_Password:  paswrd
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-8cqfh (ro)
...
```

```sh
# 현재 존재하는 서비스 중에는 mysql-service이 없으므로 mysql 서비스의 이름을 mysql-service로 변경합니다.
$ kubectl edit svc mysql
```

```sh
# 즉시 수정된 정보가 반영되지는 않으므로 기존 mysql 서비스를 삭제하고 mysql-service 서비스를 생성합니다.
$ kubectl delete svc mysql -n alpha
$ kubectl create -f /tmp/kubectl-edit-875122476.yaml
```

### 문제. 2

간단한 2-tier 애플리케이션이 beta 네임스페이스에 배포됩니다. 성공하면 녹색 웹 페이지가 표시되어야 합니다. 문제를 해결하고 수정하세요.

```
웹페이지에 접속하면 아래와 같이 "Connection refused" 문제 때문에 DB_Host에 접속하지 못 하고 있습니다.
Environment Variables: DB_Host=mysql-service; DB_Database=Not Set; DB_User=root; DB_Password=paswrd; 2003: Can't connect to MySQL server on 'mysql-service:3306' (111 Connection refused)
From webapp-mysql-5456999f7b-nq8ms!
```

### 답안. 2

```sh
# mysql-service에 포트 정보를 상황에 맞게 수정합니다.
$ kubectl edit svc mysql-service -n beta
service/mysql-service edited
```

### 문제. 3

간단한 2-tier 애플리케이션이 gamma 네임스페이스에 배포됩니다. 성공하면 녹색 웹 페이지가 표시되어야 합니다. 문제를 해결하고 수정하세요.

```
웹페이지에 접속하면 아래와 같이 "Connection refused" 문제 때문에 DB_Host에 접속하지 못 하고 있습니다.
Environment Variables: DB_Host=mysql-service; DB_Database=Not Set; DB_User=root; DB_Password=paswrd; 2003: Can't connect to MySQL server on 'mysql-service:3306' (111 Connection refused)
From webapp-mysql-5456999f7b-wpkxl!
```

### 답안. 3

```sh
# mysql-service 상세 정보에서 selector를 확인합니다.
$ kubectl describe svc mysql-service -n gamma
Name:              mysql-service
Namespace:         gamma
Labels:            <none>
Annotations:       <none>
Selector:          name=sql00001 # Selector에 매칭되는 파드없음
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.43.140.145
IPs:               10.43.140.145
Port:              <unset>  3306/TCP
TargetPort:        3306/TCP
Endpoints:         <none>
Session Affinity:  None
Events:            <none>
```

```sh
# mysql-service에 Selecotr 정보를 상황에 맞게 수정합니다.
$ kubectl edit svc mysql-service -n beta
service/mysql-service edited
```

### 문제. 5

간단한 2-tier 애플리케이션이 epsilon 네임스페이스에 배포됩니다. 성공하면 녹색 웹 페이지가 표시되어야 합니다. 문제를 해결하고 수정하세요.

```
웹페이지에 접속하면 아래와 같이 "Access denied" 문제 때문에 DB_Host에 접속하지 못 하고 있습니다.
Environment Variables: DB_Host=mysql-service; DB_Database=Not Set; DB_User=root; DB_Password=paswrd; 1045 (28000): Access denied for user 'root'@'10.42.0.20' (using password: YES)
From webapp-mysql-5456999f7b-2drxc!
```

### 답안. 5

```sh
# Deploy 정보를 확인합니다.
$ kubectl get deploy -n epsilon
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
webapp-mysql   1/1     1            1           83s
```

```sh
# Deploy에 Containers의 env 정보에서 잘 못 설정되어 있는 패스워드를 수정합니다.
$ kubectl edit deploy webapp-mysql -n epsilon
```

```sh
# mysql 파드의 env 정보에서 잘 못 설정되어 있는 패스워드를 수정합니다.
$ kubectl edit po mysql -n epsilon

# mysql 파드를 다시 생성합니다.
$ kubectl replace --force -f /tmp/kubectl-edit-3214964805.yaml
pod "mysql" deleted
pod/mysql replaced
```

### 문제. 6

간단한 2-tier 애플리케이션이 zeta 네임스페이스에 배포됩니다. 성공하면 녹색 웹 페이지가 표시되어야 합니다. 문제를 해결하고 수정하세요.

```
웹페이지 https://30081-port-1c4a6bd1065b4900.labs.kodekloud.com/에 접속하면 502 Bad Gateway 오류가 발생합니다.
```

### 답안. 6

```sh
# 웹서비스용 서비스의 상세 정보를 확인합니다.
# 웹페이지에서는 30081 포트를 사용했는데, 서비스에 30088 포트로 설정되어 있는 것을 확인합니다.
$ kubectl describe svc web-service -n zeta
Name:                     web-service
Namespace:                zeta
Labels:                   <none>
Annotations:              <none>
Selector:                 name=webapp-mysql
Type:                     NodePort
IP Family Policy:         SingleStack
IP Families:              IPv4
IP:                       10.43.7.140
IPs:                      10.43.7.140
Port:                     <unset>  8080/TCP
TargetPort:               8080/TCP
NodePort:                 <unset>  30088/TCP
Endpoints:                10.42.0.24:8080
Session Affinity:         None
External Traffic Policy:  Cluster
Events:                   <none>
```

```sh
# NodePort 타입 서비스의 포트 번호를 수정합니다.
$ kubectl edit svc web-service -n zeta
service/web-service edited
```

## 256. Practice Test - Control Plane Failure

https://uklabs.kodekloud.com/topic/practice-test-control-plane-failure-2/

### 문제. 1

클러스터가 손상되었습니다. 애플리케이션 배포를 시도했지만 작동하지 않습니다. 문제를 해결하세요.

### 답안. 1

```sh
# 모든 컴포넌트를 확인합니다.
#
# default 네임스페이스에 app-55d5fc5fcc-4ktnd 파드가 Pending인 것을 확인합니다.
# => 파드가 Pending이라는 것은 노드가 제대로 할당되지 않았을 가능성이 큽니다.
#
# kube-system 네임스페이스에 kube-scheduler-controlplane 파드가 CrashLoopBackOff인 것을 확인합니다.
$ kubectl get all -A
NAMESPACE      NAME                                       READY   STATUS             RESTARTS      AGE
default        pod/app-55d5fc5fcc-4ktnd                   0/1     Pending            0             72s
kube-flannel   pod/kube-flannel-ds-dht4q                  1/1     Running            0             5m30s
kube-system    pod/coredns-5d78c9869d-h5p8c               1/1     Running            0             5m30s
kube-system    pod/coredns-5d78c9869d-tw6c2               1/1     Running            0             5m30s
kube-system    pod/etcd-controlplane                      1/1     Running            0             5m45s
kube-system    pod/kube-apiserver-controlplane            1/1     Running            0             5m47s
kube-system    pod/kube-controller-manager-controlplane   1/1     Running            0             5m40s
kube-system    pod/kube-proxy-mmzk8                       1/1     Running            0             5m30s
kube-system    pod/kube-scheduler-controlplane            0/1     CrashLoopBackOff   3 (16s ago)   74s

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP                  5m46s
kube-system   service/kube-dns     ClusterIP   10.96.0.10   <none>        53/UDP,53/TCP,9153/TCP   5m42s

NAMESPACE      NAME                             DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-flannel   daemonset.apps/kube-flannel-ds   1         1         1       1            1           <none>                   5m39s
kube-system    daemonset.apps/kube-proxy        1         1         1       1            1           kubernetes.io/os=linux   5m42s

NAMESPACE     NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
default       deployment.apps/app       0/1     1            0           72s
kube-system   deployment.apps/coredns   2/2     2            2           5m42s

NAMESPACE     NAME                                 DESIRED   CURRENT   READY   AGE
default       replicaset.apps/app-55d5fc5fcc       1         1         0       72s
kube-system   replicaset.apps/coredns-5d78c9869d   2         2         2       5m31s

```

```sh
# 문제가 있는 app-55d5fc5fcc-4ktnd 파드의 상세 정보를 확인합니다.
$ kubectl describe po app-55d5fc5fcc-jklmv -n default 
Name:             app-55d5fc5fcc-jklmv
Namespace:        default
Priority:         0
Service Account:  default
Node:             <none> # 노드가 none 입니다. 노드에 파드를 할당하는 건 scheduler 일이니까 scheduler가 이상할 겁니다.
...
```

```sh
# 문제가 있는 kube-scheduler-controlplane 파드의 상세 정보를 확인합니다.
# "kube-schedulerrrr": executable file not found 메세지를 통해서 명령어가 잘 못 된 것을 확인합니다.
$ kubectl describe po kube-scheduler-controlplane -n kube-system
Name:                 kube-scheduler-controlplane
Namespace:            kube-system
...
Containers:
  kube-scheduler:
    ...
    Command:
      kube-schedulerrrr
      --authentication-kubeconfig=/etc/kubernetes/scheduler.conf
      --authorization-kubeconfig=/etc/kubernetes/scheduler.conf
      --bind-address=127.0.0.1
      --kubeconfig=/etc/kubernetes/scheduler.conf
      --leader-elect=true
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       StartError
      Message:      failed to create containerd task: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: exec: "kube-schedulerrrr": executable file not found in $PATH: unknown
...
```

```sh
# kube-scheduler는 static Pod 이므로 로컬에 있는 yaml 파일을 수정하고, 저장합니다.
# 파드 변경이 반영될 때까지 기다립니다.
vi /etc/kubernetes/manifests/kube-scheduler.yaml
```

```sh
# 다시 모든 컴포넌트를 확인해보면, 모든 파드가 Running임이 확인됩니다.
$ kubectl get all -A
```

### 문제. 2

app 디플로이의 파드를 2개로 늘리시오.

### 답안. 2

```sh
# 디플로이의 replicas를 2로 바꿔도 파드가 늘어나지 않습니다.
# Deployment/ReplicaSet를 업데이트하는 건 controler manager의 일입니다.
$ kubectl edit deploy app -n default
```

```sh
# controler manager 상태를 확인합니다.
$ kubectl get po -n kube-system
NAME                                   READY   STATUS             RESTARTS      AGE
kube-controller-manager-controlplane   0/1     CrashLoopBackOff   6 (91s ago)   7m25s
```

```sh
# controler manager 상세 정보를 확인합니다.
# Event에서 컨테이너 시작이 실패한 것을 확인합니다.
$ kubectl describe po kube-controller-manager-controlplane -n kube-system 
Name:                 kube-controller-manager-controlplane
Namespace:            kube-system
...
Containers:
  kube-controller-manager:
    ...
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Error
      Exit Code:    1
      Started:      Mon, 11 Sep 2023 07:40:45 -0400
      Finished:     Mon, 11 Sep 2023 07:40:45 -0400
...
Events:
  Type     Reason   Age                     From     Message
  ----     ------   ----                    ----     -------
  Normal   Pulled   7m18s (x5 over 8m56s)   kubelet  Container image "registry.k8s.io/kube-controller-manager:v1.27.0" already present on machine
  Normal   Created  7m17s (x5 over 8m56s)   kubelet  Created container kube-controller-manager
  Normal   Started  7m17s (x5 over 8m55s)   kubelet  Started container kube-controller-manager
  Warning  BackOff  3m52s (x27 over 8m51s)  kubelet  Back-off restarting failed container kube-controller-manager in pod kube-controller-manager-controlplane_kube-system(f3953fa63b68c6342d63526577111c32)
```

```sh
# controler manager 파드 로그에서 실패한 원인을 알 수 있습니다.
$ kubectl logs -n kube-system kube-controller-manager-controlplane 
I0911 11:40:45.981225       1 serving.go:348] Generated self-signed cert in-memory
E0911 11:40:45.981473       1 run.go:74] "command failed" err="stat /etc/kubernetes/controller-manager-XXXX.conf: no such file or directory"
```

```sh
# controler manager는 static Pod 이므로 로컬에 있는 yaml 파일을 수정하고, 저장합니다.
# 파드 변경이 반영될 때까지 기다립니다.
vi /etc/kubernetes/manifests/kube-controller-manager.yaml
```

### 문제. 4

scaling에 뭔가 문제가 또 생겼습니다. deployment가 3 replicas까지 늘어날 수 있게 하세요.

### 답안. 4

```sh
# 모든 컴포넌트를 확인해보면, 다시 controller manager에 문제가 있는게 보입니다.
$ kubectl get all -A
NAMESPACE      NAME                                       READY   STATUS    RESTARTS      AGE
kube-system    pod/kube-controller-manager-controlplane   0/1     Error     2 (23s ago)   29s
```

```sh
# controller manager 파드 로그를 확인하고, 인증서 파일이 문제임을 확인합니다.
$ kubectl logs -n kube-system kube-controller-manager-controlplane 
I0911 11:56:26.193803       1 serving.go:348] Generated self-signed cert in-memory
E0911 11:56:26.763146       1 run.go:74] "command failed" err="unable to load client CA provider: open /etc/kubernetes/pki/ca.crt: no such file or directory"
```

```sh
# 호스트에서 인증서 파일은 확인이 됩니다.
$ ls -al /etc/kubernetes/pki/ca.crt
-rw-r--r-- 1 root root 1099 Sep 11 07:19 /etc/kubernetes/pki/ca.crt
```

```sh
# 인증서 파일이 있음에도 controller manager는 인증서 파일을 못 찾았습니다.
# controller manager 정의 파일에서 volume과 volumeMounts 정보를 확인하고 잘 못된 부분을 수정합니다.
$ vi /etc/kubernetes/manifests/kube-controller-manager.yaml
...
spec:
  containers:
  - command:
    - kube-controller-manager
    - --allocate-node-cidrs=true
    - --authentication-kubeconfig=/etc/kubernetes/controller-manager.conf
    - --authorization-kubeconfig=/etc/kubernetes/controller-manager.conf
    - --bind-address=127.0.0.1
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --cluster-cidr=10.244.0.0/16
    - --cluster-name=kubernetes
    - --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt
    - --cluster-signing-key-file=/etc/kubernetes/pki/ca.key
    - --controllers=*,bootstrapsigner,tokencleaner
    - --kubeconfig=/etc/kubernetes/controller-manager.conf
    - --leader-elect=true
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --root-ca-file=/etc/kubernetes/pki/ca.crt
    - --service-account-private-key-file=/etc/kubernetes/pki/sa.key
    - --service-cluster-ip-range=10.96.0.0/12
    - --use-service-account-credentials=true
...
    volumeMounts:
    - mountPath: /etc/ssl/certs
      name: ca-certs
      readOnly: true
    - mountPath: /etc/ca-certificates
      name: etc-ca-certificates
      readOnly: true
    - mountPath: /usr/libexec/kubernetes/kubelet-plugins/volume/exec
      name: flexvolume-dir
    - mountPath: /etc/kubernetes/pki
      name: k8s-certs
      readOnly: true
...
  volumes:
  - hostPath:
      path: /etc/ssl/certs
      type: DirectoryOrCreate
    name: ca-certs
  - hostPath:
      path: /etc/ca-certificates
      type: DirectoryOrCreate
    name: etc-ca-certificates
  - hostPath:
      path: /usr/libexec/kubernetes/kubelet-plugins/volume/exec
      type: DirectoryOrCreate
    name: flexvolume-dir
  - hostPath:
      path: /etc/kubernetes/WRONG-PKI-DIRECTORY
      type: DirectoryOrCreate
    name: k8s-certs
...
```

## 259. Practice Test - Worker Node Failure

https://uklabs.kodekloud.com/topic/practice-test-worker-node-failure-2/

### Worker Node 확인에 유용한 명령어

```
$ kubectl get nodes

$ kubectl describe node worker-1

$ top

$ df -h

$ service kubelet status

$ sudo journalctl -u kubelet

# 인증서 발행자와 유효기간 확인
$ openssl x509 -in /var/lib/kubelet/worker-1.crt -text
```

### 1. 문제

broken된 클러스터를 고치시오.

### 1. 답안

```sh
# 노드 상태를 확인합니다.
$ kubectl get nodes
NAME           STATUS     ROLES           AGE     VERSION
controlplane   Ready      control-plane   3m37s   v1.27.0
node01         NotReady   <none>          3m7s    v1.27.0
```

```sh
# Worker 노드에 접속해서 kubelet 부터 확인합니다.
$ ssh node01
$ service kubelet status
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: inactive (dead) since Mon 2023-09-11 08:13:54 EDT; 5min ago
...
```

```sh
# Worker 노드에 kubelet이 비활성화 상태이니 서비스를 시작합니다.
$ service kubelet start
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: inactive (dead) since Mon 2023-09-11 08:13:54 EDT; 5min ago
...
```

```sh
# 다시 controlplane 노드로 돌아가서 노드 상태를 확인합니다.
$ kubectl get nodes 
NAME           STATUS   ROLES           AGE   VERSION
controlplane   Ready    control-plane   11m   v1.27.0
node01         Ready    <none>          10m   v1.27.0
```

### 2. 문제

다시 broken된 클러스터를 고치시오.

### 2. 답안

```sh
# 노드 상태를 확인합니다.
$ kubectl get nodes
NAME           STATUS     ROLES           AGE     VERSION
controlplane   Ready      control-plane   3m37s   v1.27.0
node01         NotReady   <none>          3m7s    v1.27.0
```

```sh
# Worker 노드에 접속해서 kubelet 부터 확인합니다.
$ ssh node01
$ service kubelet status
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: activating (auto-restart) (Result: exit-code) since Mon 2023-09-11 08:23:52 EDT; 8s ago
       Docs: https://kubernetes.io/docs/home/
    Process: 5330 ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_CONFIG_ARGS $KUBELET_KUBEADM_ARGS $KUBELET_EXTRA_ARGS (code=exited, status=1/FAILURE)
   Main PID: 5330 (code=exited, status=1/FAILURE)
```

```sh
# Worker 노드에 시스템 로그에서 kubelet 서비스 관련 로그를 조회합니다.
# 시스템 로그에서 인증서 파일을 찾지 못 해서 문제가 생긴 것을 확인합니다.
$ journalctl -u kubelet
Sep 11 08:27:20 node01 kubelet[6600]: I0911 08:27:20.917824    6600 server.go:199]  "--pod-infra-container-image will not be pruned by the image garbage collector in kubelet and should also be set in the remote runtime"
Sep 11 08:27:20 node01 kubelet[6600]: E0911 08:27:20.921974    6600 run.go:74] "command failed" err="failed to construct kubelet dependencies: unable to load client CA file /etc/kubernetes/pki/WRONG-CA-FILE.crt: open /etc/kubernetes/pki/WRONG-CA-FILE.crt: no such file or directory"
...
```

```sh
# Worker 노드에 kubelet의 기본 설정 파일은 아래와 같지만, 이것만 있는 건 아닙니다.
$ vi /etc/kubernetes/kubelet.conf

# Worker 노드에 kubelet의 또 다른 구성 파일은 아래와 같습니다.
# 이 파일에서 잘못된 clientCAFile 부분을 수정합니다.
$ vi /var/lib/kubelet/config.yaml
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
```

```sh
# Worker 노드에 kubelet이 비활성화 상태이니 서비스를 시작합니다.
$ service kubelet start
```

### 3. 문제

다시 broken된 클러스터를 고치시오.

### 3. 답안

```sh
# 노드 상태를 확인합니다.
$ kubectl get nodes
NAME           STATUS     ROLES           AGE     VERSION
controlplane   Ready      control-plane   3m37s   v1.27.0
node01         NotReady   <none>          3m7s    v1.27.0
```

```sh
# Worker 노드에 접속해서 kubelet 부터 확인합니다. active 상태인게 이상합니다.
$ ssh node01
$ service kubelet status
● kubelet.service - kubelet: The Kubernetes Node Agent
     Loaded: loaded (/lib/systemd/system/kubelet.service; enabled; vendor preset: enabled)
    Drop-In: /etc/systemd/system/kubelet.service.d
             └─10-kubeadm.conf
     Active: active (running) since Mon 2023-09-11 08:36:31 EDT; 1min 17s ago
```

```sh
# Worker 노드에 시스템 로그에서 kubelet 서비스 관련 로그를 조회합니다.
# 시스템로그에서 controlplane:6553에서 node01의 연결을 거부한 것이 확인됩니다.
$ journalctl -u kubelet
Sep 11 08:39:41 node01 kubelet[9672]: E0911 08:39:41.039232    9672 kubelet_node_status.go:92] "Unable to register node with API server" err="Post \"https://controlplane:6553/api/v1/nodes\": dial tcp 192.23.2.2:6553: connect: connection refused" node="node01"
```

```sh
# Worker 노드에 kubelet의 기본 설정 파일에서 잘 못 되어 있는 controlplane의 포트를 수정합니다.
$ vi /etc/kubernetes/kubelet.conf
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: ...
    server: https://controlplane:6443
  name: default-cluster

# Worker 노드에 kubelet의 또 다른 구성 파일은 아래와 같습니다.
$ vi /var/lib/kubelet/config.yaml
```

```sh
# Worker 노드에 kubelet이 비활성화 상태이니 서비스를 시작합니다.
$ service kubelet start
```
