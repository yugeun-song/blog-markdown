# Kubernetes 네트워킹 모델 — Pod 간 통신의 모든 것

Kubernetes의 네트워킹 모델은 세 가지 근본 요구사항을 충족해야 한다. 모든 Pod은 NAT 없이 다른 모든 Pod과 통신할 수 있어야 하고, 노드의 에이전트는 모든 Pod과 통신할 수 있어야 하며, Pod은 자신이 보는 IP와 다른 Pod이 보는 IP가 동일해야 한다.

## Pod 네트워크 기초

각 Pod은 고유한 IP 주소를 가진다. 같은 Pod 내의 컨테이너들은 network namespace를 공유하므로 `localhost`로 통신한다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container
spec:
  containers:
  - name: web
    image: nginx
    ports:
    - containerPort: 80
  - name: sidecar
    image: busybox
    command: ["sh", "-c", "while true; do wget -qO- localhost:80; sleep 5; done"]
```

`sidecar` 컨테이너가 `localhost:80`으로 `web` 컨테이너에 접근할 수 있는 이유가 바로 공유된 network namespace다.

### pause 컨테이너

Pod의 network namespace를 소유하는 것은 `pause` 컨테이너다. 이 초경량 컨테이너(약 700KB)가 먼저 생성되어 namespace를 확보하고, 이후 애플리케이션 컨테이너들이 이 namespace에 합류한다.

```bash
# pause 컨테이너 확인
crictl ps | grep pause
# a1b2c3  registry.k8s.io/pause:3.9  ...
```

애플리케이션 컨테이너가 재시작되어도 pause 컨테이너가 살아있으므로 IP 주소가 유지된다.

## CNI 플러그인

Kubernetes는 직접 네트워킹을 구현하지 않고, CNI(Container Network Interface) 플러그인에 위임한다. 주요 CNI 구현체로는 Calico, Cilium, Flannel 등이 있다.

```json
{
  "cniVersion": "1.0.0",
  "name": "mynet",
  "type": "bridge",
  "bridge": "cni0",
  "isGateway": true,
  "ipMasq": true,
  "ipam": {
    "type": "host-local",
    "subnet": "10.244.1.0/24",
    "routes": [
      { "dst": "0.0.0.0/0" }
    ]
  }
}
```

CNI 플러그인은 Pod 생성 시 kubelet에 의해 호출되어, 네트워크 인터페이스를 생성하고 IP를 할당한다.

### 같은 노드 내 통신

같은 노드의 Pod 간 통신은 Linux 브릿지(또는 eBPF)를 통해 이루어진다. 각 Pod의 veth 페어가 노드의 브릿지 인터페이스에 연결되어, L2 스위칭으로 패킷이 전달된다.

```bash
# 노드에서 브릿지 확인
bridge link show
# 3: veth1234@if2: <BROADCAST,MULTICAST,UP> master cni0
# 5: veth5678@if2: <BROADCAST,MULTICAST,UP> master cni0
```

### 다른 노드 간 통신

노드 간 통신은 CNI 구현에 따라 다르다. Flannel은 VXLAN 터널링을, Calico는 BGP 라우팅을, Cilium은 eBPF를 사용한다.

```bash
# VXLAN 인터페이스 확인 (Flannel)
ip -d link show flannel.1
# flannel.1: <BROADCAST,MULTICAST,UP> mtu 1450
#   vxlan id 1 ... dstport 4789
```

## Service와 kube-proxy

Service는 Pod 집합에 대한 안정적인 접근점을 제공한다. ClusterIP Service의 경우, kube-proxy가 iptables 또는 IPVS 규칙을 생성하여 트래픽을 백엔드 Pod으로 분산한다.

```bash
# iptables 규칙 확인 (kube-proxy iptables 모드)
sudo iptables -t nat -L KUBE-SERVICES -n
# Chain KUBE-SERVICES
# KUBE-SVC-XXX  tcp  --  0.0.0.0/0  10.96.0.10  tcp dpt:53  /* kube-system/kube-dns:dns-tcp */

# IPVS 규칙 확인 (kube-proxy ipvs 모드)
sudo ipvsadm -Ln
# TCP  10.96.0.10:53 rr
#   -> 10.244.0.5:53    Masq  1
#   -> 10.244.1.3:53    Masq  1
```

IPVS 모드가 iptables 모드보다 대규모 클러스터에서 더 나은 성능을 보인다. Service 수가 많아져도 O(1) 조회가 가능하기 때문이다.

### DNS 기반 서비스 디스커버리

CoreDNS가 클러스터 내 DNS를 담당한다. `<service>.<namespace>.svc.cluster.local` 형태로 Service를 이름으로 접근할 수 있다.

```bash
# Pod 내에서 DNS 조회
nslookup my-service.default.svc.cluster.local
# Address: 10.96.123.45
```

## Ingress와 외부 트래픽

외부에서 클러스터 내부로의 트래픽은 Ingress 컨트롤러(nginx, traefik 등)나 Gateway API를 통해 라우팅된다. L7 로드밸런싱, TLS 종료, 경로 기반 라우팅 등을 제공한다.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
```

### NetworkPolicy로 트래픽 제어

NetworkPolicy는 Pod 수준의 방화벽 규칙이다. 기본적으로 모든 트래픽이 허용되며, NetworkPolicy를 적용하면 화이트리스트 방식으로 전환된다.

CNI 플러그인이 NetworkPolicy를 지원해야 한다. Flannel은 지원하지 않으므로, 보안이 필요하면 Calico나 Cilium을 사용해야 한다.
