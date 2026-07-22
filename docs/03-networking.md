# 3️⃣ Kubernetes 네트워킹 🌐

> **목표**: Service, Ingress, NetworkPolicy로 Pod 간 통신 이해
> 
> **학습 시간**: 2-3시간 | **난이도**: ⭐⭐⭐ 중급

---

## 📋 목차
1. [Service - 핵심](#service--핵심)
2. [Service 타입](#service-타입)
3. [DNS & 서비스 발견](#dns--서비스-발견)
4. [Ingress - 외부 접근](#ingress--외부-접근)
5. [NetworkPolicy - 보안](#networkpolicy--보안)

---

## Service - 핵심

**Service**는 **Pod 네트워크 접근 추상화**입니다.

```
클라이언트 → Service (VIP) → Endpoint (Pod IP 목록) → Pod
            stable    동적 변함
```

**문제 상황:**
- Pod IP는 재시작되면 변함
- Pod 여러 개일 때 어디로 접근?

**해결책:** Service!

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
spec:
  selector:
    app: nginx          # "app=nginx" Label 선택
  ports:
  - protocol: TCP
    port: 80            # Service 포트
    targetPort: 8080    # Pod 포트
  type: ClusterIP       # 기본값
```

**동작:**
```
Service (10.0.1.1:80) 
   ↓
Endpoint (Pod IP 목록)
   ├─ 192.168.1.10:8080
   ├─ 192.168.1.11:8080
   └─ 192.168.1.12:8080
```

---

## Service 타입

### 1️⃣ ClusterIP - 내부만 (기본값)

```yaml
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 8080
```

**접근:**
```bash
# 같은 클러스터 내에서만
kubectl run -it curl --image=curlimages/curl -- curl http://nginx:80

# 또는 IP로
curl http://10.0.1.1:80
```

### 2️⃣ NodePort - 노드 포트로 노출

```yaml
spec:
  type: NodePort
  ports:
  - port: 80            # Service 포트
    targetPort: 8080    # Pod 포트
    nodePort: 30008     # 노드 포트 (30000-32767)
```

**접근:**
```bash
# 노드의 IP:30008으로 접근
curl http://node-ip:30008

# 또는 localhost:30008 (Minikube)
curl http://localhost:30008
```

### 3️⃣ LoadBalancer - 클라우드 LB 사용

```yaml
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
```

**동작 (AWS/Azure):**
```
클라이언트 → 클라우드 LB (외부 IP) → NodePort → Pod
```

**확인:**
```bash
kubectl get svc
# EXTERNAL-IP에 클라우드 LB IP 표시
```

---

## DNS & 서비스 발견

### CoreDNS

Kubernetes의 **내장 DNS** 서버입니다.

```bash
# CoreDNS Pod 확인
kubectl get pods -n kube-system -l k8s-app=kube-dns
```

### FQDN (정규화된 도메인명)

```
nginx                                  # 같은 namespace
nginx.default                          # 다른 namespace (명시)
nginx.default.svc                      # 완전 이름
nginx.default.svc.cluster.local        # FQDN
```

**Pod에서 접근:**
```bash
# 같은 namespace
curl http://nginx:80

# 다른 namespace
curl http://nginx.production:80

# 완전 이름
curl http://nginx.default.svc.cluster.local:80
```

### 환경 변수 주입

```bash
# Service 접근 정보가 자동으로 Pod에 주입됨
kubectl exec pod-name -- env | grep NGINX
# NGINX_SERVICE_HOST=10.0.1.1
# NGINX_SERVICE_PORT=80
```

---

## 🧪 직접 실습해보기

**클러스터에서 실제로 배포하고 테스트해보세요:**

→ [**실습 04: Kubernetes 네트워킹**](../labs/04-networking/README.md)
- ✅ ClusterIP Service 배포 및 내부 통신
- ✅ NodePort Service로 외부 접근
- ✅ DNS 서비스 발견 테스트
- ✅ 실제 kubectl 명령어와 결과

---

## Ingress - 외부 접근

**Service는 포트 기반**인데, **HTTP/HTTPS 경로 기반**으로 라우팅하려면?

→ **Ingress!**

```
인터넷 → Ingress Controller (Nginx, Traefik) → Service → Pod
        (HTTP/HTTPS 라우팅)
```

### 예제: 경로 기반 라우팅

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: api-ingress
spec:
  ingressClassName: nginx        # Ingress Controller
  rules:
  - host: example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 8080
      - path: /web
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

**동작:**
```
GET /api/users    → api-service:8080
GET /web/home     → web-service:80
```

### HTTPS 지원

```yaml
spec:
  tls:
  - hosts:
    - example.com
    secretName: tls-cert         # Secret에 인증서 저장
  rules:
  - host: example.com
    ...
```

---

## NetworkPolicy - 보안

**NetworkPolicy**는 **마이크로세그멘테이션** (Pod 간 네트워크 격리)입니다.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}                # 모든 Pod
  policyTypes:
  - Ingress                      # 수신 차단
  - Egress                       # 송신 차단
```

이는 모든 트래픽을 차단합니다!

### 예제: 특정 Pod만 허용

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
spec:
  podSelector:
    matchLabels:
      app: backend               # backend Pod만 대상
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend          # frontend에서만 허용
    ports:
    - protocol: TCP
      port: 8080
```

---

## 🎯 핵심 요점

| 개념 | 역할 |
|------|------|
| **Service (ClusterIP)** | Pod 내부 통신 |
| **Service (NodePort)** | 노드 포트로 노출 |
| **Service (LoadBalancer)** | 클라우드 LB로 노출 |
| **Ingress** | HTTP/HTTPS 경로 라우팅 |
| **NetworkPolicy** | Pod 간 통신 제어 |

---

## 📚 다음 단계

1. **실습하기**: [Lab 04 - Kubernetes 네트워킹](../labs/04-networking/README.md)
2. **저장소**: [Doc 04 - Kubernetes 저장소](04-storage.md)
3. **전체 Labs**: [Labs 홈](../labs/README.md)

---

**마지막 업데이트: 2026-07-22**
