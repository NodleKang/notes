```
# 브릿지 타입 네트워크 IP 주소 정보
ip address show type bridge

# 브리지 인터페이스 v-net-0를 생성(브리지: 네트워크 패킷을 전달하는 역할)
ip link add v-net-0 type bridge

# v-net-0 브리지 인터페이스를 활성화
ip link set dev v-net-0 up

# v-net-0에 IP 주소 10.244.1.1/24를 할당
ip addr add 10.244.1.1/24 dev v-net-0

# 두 개의 가상 이더넷 인터페이스를 생성하고 연결
ip link add veth-red type veth peer name veth-red-br

# veth-red 인터페이스를 네임스페이스 'red'로 이동
ip link set veth-red netns red

# 'red' 네임스페이스 내의 veth-red에 IP 주소 192.168.15.1을 할당
ip -n red addr add 192.168.15.1 dev veth-red

# 'red' 네임스페이스 내의 veth-red 인터페이스를 활성화
ip -n red red link set veth-red up

# veth-red-br을 v-net-0 브리지의 멤버로 추가
ip link set veth-red-br master v-net-0

# 'blue' 네임스페이스에서 192.168.1.0/24 네트워크로 가는 경로를 설정하고 이 경로를 192.168.15.5로 라우팅
ip netns exec blue ip route add 192.168.1.0/24 via 192.168.15.5

# NAT (Network Address Translation) 규칙을 추가하여 192.168.15.0/24 서브넷에서 나가는 패킷의 출발지 주소를 마스커레이드(변환)
iptables -t -nat -A POSTROUTING -s 192.168.15.0/24 -j MASQUERADE
```

```
# veth 쌍 생성
ip link add ...

# veth 쌍 붙이기
ip link set ...
ip link set ...

# IP 주소 할당
ip -n <namespace> addr add ...
ip -n <namespace> route add ...

# 인테피이스 가져오기
ip -n <namespace> link set ...
```

컨테이너를 가지고 있는 파드
네임스페이스와 인터페이스를 가지고 있는 컨테이너
인터페이스에 할당돼있는 IP들

서비스를 생성하면 사전에 정의돼 있는 범위에서 IP 주소를 할당받는다
kube-proxy 컴포넌트는 각 노드에서 실행된다
