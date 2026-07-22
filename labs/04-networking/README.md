# 🧪 Lab 04: Kubernetes 네트워킹 실습

> **목표**: Service(ClusterIP, NodePort), DNS, Ingress 직접 배포해서 Pod 간 통신 이해
>
> **소요 시간**: 2-3시간 | **난이도**: ⭐⭐⭐ 중급

---

## 📖 개념 (빠른 정리)

| Service 타입 | 접근 범위 | 사용처 |
|-------------|---------|--------|
| **ClusterIP** | 클러스터 내부만 | 내부 Pod 통신 |
| **NodePort** | 모든 노드 포트 | 개발/테스트 |
| **LoadBalancer** | 클라우드 LB | 프로덕션 |
| **ExternalName** | 외부 DNS 매핑 | 외부 서비스 연결 |

---

## 🚀 실습 A: ClusterIP Service (Pod 간 내부 통신)

**목표**: 클러스터 내부에서만 접근 가능한 Service 만들기

### 0️⃣ 사전 준비

```bash
# 현재 namespace 확인
kubectl get namespace

# 새로운 namespace 생성 (선택사항, default 사용해도 OK)
kubectl create namespace networking-demo
kubectl config set-context --current --namespace=networking-demo
```

**기대 결과:**
```
NAME              STATUS   AGE
default           Active   30d
kube-node-lease   Active   30d
kube-public       Active   30d
kube-system       Active   30d
networking-demo   Active   1s
```

### 1️⃣ Deployment + ClusterIP Service 배포

```bash
kubectl apply -f - <<'EOF'
# --- Deployment ---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-app
  labels:
    app: web-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web-app
  template:
    metadata:
      labels:
        app: web-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "32Mi"
            cpu: "50m"
          limits:
            memory: "64Mi"
            cpu: "100m"

---
# --- ClusterIP Service ---
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  type: ClusterIP
  selector:
    app: web-app
  ports:
  - port: 80
    targetPort: 80
EOF
```

**기대 결과:**
```
deployment.apps/web-app created
service/web-service created
```

### 2️⃣ Deployment 및 Service 확인

```bash
# Deployment 상태 확인
kubectl get deployment web-app

# Pod 확인 (3개 running)
kubectl get pods -l app=web-app

# Service 확인 (ClusterIP 할당됨)
kubectl get svc web-service

# Endpoints 확인 (Pod IP 목록)
kubectl get endpoints web-service
```

**기대 결과:**
```bash
$ kubectl get deployment web-app
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
web-app   3/3     3            3           10s

$ kubectl get pods -l app=web-app
NAME                       READY   STATUS    RESTARTS   AGE
web-app-5d4b4c4f6b-2k8qx   1/1     Running   0          10s
web-app-5d4b4c4f6b-7j9km   1/1     Running   0          10s
web-app-5d4b4c4f6b-tkxlm   1/1     Running   0          10s

$ kubectl get svc web-service
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)
web-service   ClusterIP   10.0.123.45     <none>        80/TCP

$ kubectl get endpoints web-service
NAME          ENDPOINTS
web-service   10.244.1.10:80,10.244.1.11:80,10.244.1.12:80
```

### 3️⃣ ClusterIP를 통한 Service 접근

```bash
# curl Pod 생성 (테스트용)
kubectl run curl-test --image=curlimages/curl --rm -it --restart=Never -- \
  curl http://web-service:80
```

**기대 결과:**
```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
<h1>Welcome to nginx!</h1>
...
</body>
</html>
```

### 4️⃣ Service 이름으로 DNS 조회

```bash
# nslookup 테스트용 Pod 생성
kubectl run dnsutils --image=nicolaka/netshoot --restart=Never -- sleep infinity

# Short name으로 조회
kubectl exec -it dnsutils -- nslookup web-service

# FQDN으로 조회
kubectl exec -it dnsutils -- nslookup web-service.networking-demo.svc.cluster.local

# DNS 서버 확인 (CoreDNS IP: 10.96.0.10)
kubectl exec -it dnsutils -- cat /etc/resolv.conf
```

**기대 결과:**
```bash
$ kubectl exec -it dnsutils -- nslookup web-service
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   web-service.networking-demo.svc.cluster.local
Address: 10.0.123.45

$ kubectl exec -it dnsutils -- cat /etc/resolv.conf
search networking-demo.svc.cluster.local svc.cluster.local cluster.local
nameserver 10.96.0.10
options ndots:5
```

### 5️⃣ Pod 삭제 후 자동 복구 확인

```bash
# 한 개 Pod 삭제
kubectl delete pod $(kubectl get pod -l app=web-app -o jsonpath='{.items[0].metadata.name}')

# 새 Pod이 생성됨 (Deployment가 보장)
kubectl get pods -l app=web-app

# Service Endpoints가 자동 업데이트됨
kubectl get endpoints web-service

# Service 접근은 여전히 작동
kubectl run curl-test2 --image=curlimages/curl --rm -it --restart=Never -- \
  curl http://web-service:80
```

**기대 결과:**
```bash
$ kubectl get pods -l app=web-app
NAME                       READY   STATUS    RESTARTS   AGE
web-app-5d4b4c4f6b-2k8qx   1/1     Running   0          5s   # 새로 생성됨
web-app-5d4b4c4f6b-7j9km   1/1     Running   0          12s
web-app-5d4b4c4f6b-tkxlm   1/1     Running   0          12s

$ kubectl get endpoints web-service
NAME          ENDPOINTS
web-service   10.244.1.13:80,10.244.1.11:80,10.244.1.12:80  # IP 변함 (자동 업데이트됨)
```

### ✅ 배운 것

- ✓ ClusterIP는 클러스터 내부에서만 접근 가능
- ✓ Pod IP가 변해도 Service는 Endpoint를 자동 업데이트
- ✓ Service 이름(DNS)으로 접근 가능
- ✓ CoreDNS(10.96.0.10)가 모든 DNS 쿼리 처리

---

## 🚀 실습 B: NodePort Service (외부 접근)

**목표**: 클러스터 외부에서 고정 포트로 접근하기

### 0️⃣ 사전 준비

```bash
# 이전 실습 A의 Deployment/Service 사용 (필수!)
# 아래 명령어로 확인
kubectl get svc web-service
kubectl get pods -l app=web-app

# 만약 없다면 실습 A의 "1️⃣ 배포" 부분을 다시 실행
```

### 1️⃣ NodePort Service 생성

```bash
kubectl apply -f - <<'EOF'
apiVersion: v1
kind: Service
metadata:
  name: web-service-nodeport
spec:
  type: NodePort
  selector:
    app: web-app
  ports:
  - port: 80              # Service 내부 포트
    targetPort: 80        # Pod 포트
    nodePort: 30080       # 노드 외부 포트 (30000-32767)
EOF
```

**기대 결과:**
```
service/web-service-nodeport created
```

### 2️⃣ NodePort Service 확인

```bash
# Service 확인 (NodePort 할당 확인)
kubectl get svc web-service-nodeport

# 노드 IP 확인
kubectl get nodes -o wide

# Endpoints 확인
kubectl get endpoints web-service-nodeport
```

**기대 결과:**
```bash
$ kubectl get svc web-service-nodeport
NAME                    TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)
web-service-nodeport    NodePort   10.0.234.56     <none>        80:30080/TCP

$ kubectl get nodes -o wide
NAME                                STATUS   ROLES    INTERNAL-IP   EXTERNAL-IP
aks-agentpool-12345678-vmss000000   Ready    agent    10.224.0.4    <none>
aks-agentpool-12345678-vmss000001   Ready    agent    10.224.0.5    <none>

$ kubectl get endpoints web-service-nodeport
NAME                    ENDPOINTS
web-service-nodeport    10.244.1.10:80,10.244.1.11:80,10.244.1.12:80
```

### 3️⃣ NodePort 접근 (로컬에서 테스트)

**로컬 머신에서:**
```bash
# kubectl port-forward로 테스트 (가장 간단)
kubectl port-forward svc/web-service-nodeport 8080:80 &

# localhost:8080으로 접근
curl http://localhost:8080
```

**기대 결과:**
```
Forwarding from 127.0.0.1:8080 -> 80
```

### 4️⃣ Minikube에서 테스트하는 경우

```bash
# Minikube의 NodePort 자동 열기
minikube service web-service-nodeport

# 또는 수동으로 Minikube IP 확인 후 접근
MINIKUBE_IP=$(minikube ip)
curl http://$MINIKUBE_IP:30080
```

### 5️⃣ 실제 AKS 클러스터에서 테스트

**방법 1: 클러스터 내부에서 테스트 (가장 간단)**

```bash
# 노드 IP 확인
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
echo "Node Internal IP: $NODE_IP"

# 클러스터 내부 Pod에서 NodePort로 접근
kubectl run curl-test --image=curlimages/curl --rm -it --restart=Never -- \
  curl http://$NODE_IP:30080
```

**기대 결과:**
```html
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<body>
<h1>Welcome to nginx!</h1>
</body>
</html>
```

**방법 2: kubectl port-forward를 Service에 사용 (권장)**

```bash
# ClusterIP Service로 port-forward 설정
kubectl port-forward svc/web-service-nodeport 8080:80 &

# 로컬에서 접근
curl http://localhost:8080

# 백그라운드 프로세스 중지
jobs
kill %1  # 또는 fg 후 Ctrl+C
```

**기대 결과:**
```
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80

<!DOCTYPE html>
<html>
...
```

**방법 3: LoadBalancer Service 사용 (프로덕션)**

```bash
# LoadBalancer로 변경 (기존 NodePort를 다시 배포하거나 수정)
kubectl patch svc web-service-nodeport -p '{"spec":{"type":"LoadBalancer"}}'

# 공개 IP 확인 (AKS에서 자동 할당)
kubectl get svc web-service-nodeport --watch

# 공개 IP로 접근
LOAD_BALANCER_IP=$(kubectl get svc web-service-nodeport -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
curl http://$LOAD_BALANCER_IP
```

**기대 결과:**
```bash
$ kubectl get svc web-service-nodeport
NAME                    TYPE           CLUSTER-IP      EXTERNAL-IP     PORT(S)
web-service-nodeport    LoadBalancer   10.0.234.56     20.123.45.67    80:30080/TCP

$ curl http://20.123.45.67
<!DOCTYPE html>
<html>
...
```

### ✅ 배운 것

- ✓ NodePort는 클러스터의 모든 노드에서 같은 포트 개방
- ✓ 외부에서 `nodeIP:nodePort`로 접근 가능
- ✓ Pod IP가 변해도 NodePort는 변하지 않음
- ✓ 프로덕션에서는 LoadBalancer 권장 (NodePort는 개발/테스트용)

---

## 🚀 실습 C: Ingress (HTTP/HTTPS 라우팅)

**목표**: HTTP 호스트명 기반 라우팅 설정하기

### 0️⃣ 사전 준비

```bash
# Ingress Controller 확인 (cluster에 따라 다름)
kubectl get ingressclass

# Ingress Controller가 없으면 NGINX Ingress Controller 설치 (선택사항)
# helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
# helm install ingress-nginx ingress-nginx/ingress-nginx \
#   --namespace ingress-nginx --create-namespace

# 이전 실습의 web-service 확인
kubectl get svc web-service
```

### 1️⃣ 두 개의 다른 앱 Deployment 생성

```bash
kubectl apply -f - <<'EOF'
# --- App 1: nginx ---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app-nginx
  template:
    metadata:
      labels:
        app: app-nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "32Mi"
            cpu: "50m"
          limits:
            memory: "64Mi"
            cpu: "100m"

---
# --- App 2: httpbin (JSON 응답) ---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-httpbin
spec:
  replicas: 2
  selector:
    matchLabels:
      app: app-httpbin
  template:
    metadata:
      labels:
        app: app-httpbin
    spec:
      containers:
      - name: httpbin
        image: kennethreitz/httpbin:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "32Mi"
            cpu: "50m"
          limits:
            memory: "64Mi"
            cpu: "100m"

---
# --- Service 1: nginx ---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: app-nginx
  ports:
  - port: 80
    targetPort: 80

---
# --- Service 2: httpbin ---
apiVersion: v1
kind: Service
metadata:
  name: httpbin-service
spec:
  type: ClusterIP
  selector:
    app: app-httpbin
  ports:
  - port: 80
    targetPort: 80
EOF
```

**기대 결과:**
```
deployment.apps/app-nginx created
deployment.apps/app-httpbin created
service/nginx-service created
service/httpbin-service created
```

### 2️⃣ 배포 확인

```bash
# Deployment 확인
kubectl get deployment -l app in (app-nginx, app-httpbin)

# Service 확인
kubectl get svc nginx-service httpbin-service

# Pod 확인
kubectl get pods
```

**기대 결과:**
```bash
$ kubectl get svc
NAME               TYPE        CLUSTER-IP      PORT(S)
nginx-service      ClusterIP   10.0.111.22     80/TCP
httpbin-service    ClusterIP   10.0.222.33     80/TCP
```

### 3️⃣ Ingress 생성 (호스트명 기반 라우팅)

```bash
kubectl apply -f - <<'EOF'
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ingress
  annotations:
    kubernetes.io/ingress.class: "nginx"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"  # HTTPS용 (선택사항)
spec:
  rules:
  # nginx.example.com → nginx-service
  - host: "nginx.example.com"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-service
            port:
              number: 80

  # api.example.com → httpbin-service
  - host: "api.example.com"
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: httpbin-service
            port:
              number: 80
EOF
```

**기대 결과:**
```
ingress.networking.k8s.io/web-ingress created
```

### 4️⃣ Ingress 상태 확인

```bash
# Ingress 확인
kubectl get ingress web-ingress

# Ingress 상세 정보 (External IP 확인)
kubectl describe ingress web-ingress
```

**기대 결과:**
```bash
$ kubectl get ingress
NAME          CLASS   HOSTS                         ADDRESS         PORTS   AGE
web-ingress   nginx   nginx.example.com,api...      20.123.45.67    80      10s

$ kubectl describe ingress web-ingress
Name:             web-ingress
Namespace:        networking-demo
Address:          20.123.45.67
Host                    Path  Backends
----                    ----  --------
nginx.example.com       /     nginx-service:80
api.example.com         /     httpbin-service:80
```

### 5️⃣ Ingress를 통한 접근 테스트

**로컬 머신에서:**
```bash
# Ingress IP 확인
INGRESS_IP=$(kubectl get ingress web-ingress -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo "Ingress IP: $INGRESS_IP"

# /etc/hosts 수정 (관리자 권한 필요)
# Windows: C:\Windows\System32\drivers\etc\hosts
# Linux/Mac: /etc/hosts
# 다음 줄 추가:
# $INGRESS_IP  nginx.example.com
# $INGRESS_IP  api.example.com

# 또는 curl에서 -H 옵션으로 테스트
curl -H "Host: nginx.example.com" http://$INGRESS_IP/
curl -H "Host: api.example.com" http://$INGRESS_IP/get
```

**기대 결과:**
```bash
# nginx.example.com 응답
$ curl -H "Host: nginx.example.com" http://20.123.45.67/
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...

# api.example.com 응답
$ curl -H "Host: api.example.com" http://20.123.45.67/get
{
  "args": {},
  "headers": {
    "Host": "api.example.com",
    "User-Agent": "curl/7.x.x"
  },
  "origin": "10.x.x.x",
  "url": "http://api.example.com/get"
}
```

### 6️⃣ kubectl port-forward로 테스트 (INGRESS_IP가 없는 경우)

```bash
# Ingress Controller Pod 찾기
kubectl get pods -n ingress-nginx  # 또는 현재 namespace

# Port-forward (Ingress Controller가 같은 namespace에 있는 경우)
kubectl port-forward -n ingress-nginx svc/ingress-nginx-controller 8080:80 &

# 테스트
curl -H "Host: nginx.example.com" http://localhost:8080/
curl -H "Host: api.example.com" http://localhost:8080/get
```

### ✅ 배운 것

- ✓ Ingress는 HTTP/HTTPS 레이어에서 라우팅
- ✓ 호스트명 기반으로 다른 Service로 라우팅 가능
- ✓ Path 기반 라우팅도 가능 (/api → api-service, /web → web-service)
- ✓ Ingress Controller가 필요 (Nginx, AGIC, Traefik 등)

---

## 🧹 정리 (모든 리소스 삭제)

```bash
# 방법 1: 각 리소스 개별 삭제
kubectl delete ingress web-ingress
kubectl delete svc web-service web-service-nodeport nginx-service httpbin-service
kubectl delete deployment web-app app-nginx app-httpbin
kubectl delete pod dnsutils curl-test curl-test2 2>/dev/null

# 방법 2: namespace 전체 삭제 (가장 간단)
kubectl delete namespace networking-demo

# 방법 3: 현재 namespace의 모든 리소스 삭제
kubectl delete all --all
```

**기대 결과:**
```
ingress.networking.k8s.io "web-ingress" deleted
service "web-service" deleted
service "web-service-nodeport" deleted
...
namespace "networking-demo" deleted
```

---

## 📚 다음 단계

- [docs/03-networking.md](../../docs/03-networking.md) - 상세 개념 학습
- [labs/03-service/README.md](../03-service/) - Service 더 알아보기
- [docs/04-storage.md](../../docs/04-storage.md) - Storage 학습

---

## 🔧 문제 해결

| 문제 | 원인 | 해결 |
|------|------|------|
| Service Pending | LB Controller 없음 | NodePort 사용 또는 port-forward |
| Ingress 주소가 <pending> | Ingress Controller 없음 | NGINX Ingress 설치 |
| DNS 조회 실패 | CoreDNS 문제 | `kubectl get pods -n kube-system` 확인 |
| Pod 간 통신 안 됨 | NetworkPolicy 제한 | `kubectl get networkpolicy` 확인 |

---

## 💡 팁

```bash
# 실시간 상태 모니터링
kubectl get pods -w

# 서비스 세부 정보
kubectl describe svc web-service

# 이벤트 확인
kubectl get events --sort-by='.lastTimestamp'

# 로그 확인
kubectl logs -l app=web-app --all-containers=true -f
```
