# 🧪 Lab 04: Kubernetes 네트워킹 실습

> **목표**: ClusterIP, NodePort, LoadBalancer Service와 Ingress를 직접 배포해서 Pod 간 통신 이해
>
> **소요 시간**: 1-2시간 | **난이도**: ⭐⭐⭐ 중급

---

## 📖 개념

### Service가 필요한 이유

```
문제: Pod IP는 재시작되면 변함
     여러 Pod에 어떻게 접근?
     
해결: Service가 stable VIP 제공
     Pod IP가 변해도 Service는 그대로
```

---

## 🚀 실습 A: ClusterIP Service (내부 통신)

### 개념

- **ClusterIP**: 클러스터 내부에서만 접근 가능
- Pod이 다른 Pod에 Service 이름으로 접근

### 완전한 배포 YAML

```yaml
# 1. Deployment 생성 (nginx 앱)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
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
# 2. ClusterIP Service 생성
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP                    # 기본값 (생략 가능)
  selector:
    app: nginx-app                   # Deployment의 Label과 일치
  ports:
  - port: 80                         # Service 포트
    targetPort: 80                   # Pod 포트
```

### 실습 시작

```bash
# 1. Deployment + Service 배포
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - name: nginx
        image: nginx:1.25
        ports:
        - containerPort: 80

---
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  type: ClusterIP
  selector:
    app: nginx-app
  ports:
  - port: 80
    targetPort: 80
EOF

# 2. Pod 확인
kubectl get pods -l app=nginx-app

# 3. Service 확인 (ClusterIP와 Endpoint 보기)
kubectl get svc nginx-service
kubectl get endpoints nginx-service

# 4. Service 접근 (클러스터 내부에서)
# curl Pod 실행해서 nginx-service로 접근
kubectl run curl -it --image=curlimages/curl --rm --restart=Never -- curl http://nginx-service:80
```

### 기대 결과

```bash
$ kubectl get svc nginx-service
NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)
nginx-service   ClusterIP   10.0.123.45     <none>        80/TCP

$ kubectl get endpoints nginx-service
NAME            ENDPOINTS
nginx-service   10.244.1.5:80,10.244.1.6:80,10.244.1.7:80

# curl 응답
$ kubectl run curl -it --image=curlimages/curl --rm --restart=Never -- curl http://nginx-service:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

### ✅ 배운 것

- Service는 Pod IP 변화를 감시하며 자동 업데이트
- Endpoint = 실제 Pod IP 목록
- 클러스터 내부에서는 Service 이름(DNS)으로 접근 가능
- `nslookup nginx-service`로 DNS 확인 가능

---

## 🚀 실습 B: NodePort Service (외부 노드 포트 노출)

### 개념

- **NodePort**: 모든 노드의 특정 포트로 외부 접근
- 클러스터 바깥에서도 접근 가능 (노드 IP:NodePort)

### 완전한 배포 YAML

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx-app
  ports:
  - port: 80                    # Service 포트
    targetPort: 80              # Pod 포트
    nodePort: 30080             # 노드 포트 (30000-32767)
```

### 실습 시작

```bash
# 1. NodePort Service 배포 (이전 Deployment 사용)
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  type: NodePort
  selector:
    app: nginx-app
  ports:
  - port: 80
    targetPort: 80
    nodePort: 30080
EOF

# 2. Service 확인 (NodePort 확인)
kubectl get svc nginx-nodeport

# 3. 노드 IP 확인
kubectl get nodes -o wide

# 4. 외부에서 접근 (Minikube인 경우)
# Minikube: 자동으로 minikube ip:30080으로 접근 가능
minikube service nginx-nodeport
# 또는 수동으로
curl http://$(minikube ip):30080

# 5. 실제 클러스터인 경우
# 노드 외부 IP를 이용해 접근
EXTERNAL_IP=$(kubectl get nodes -o wide | grep -v INTERNAL | awk 'NR==2 {print $NF}')
curl http://$EXTERNAL_IP:30080
```

### 기대 결과

```bash
$ kubectl get svc nginx-nodeport
NAME              TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)
nginx-nodeport    NodePort   10.0.234.56     <none>        80:30080/TCP

$ curl http://minikube-ip:30080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...

# Pod IP가 바뀌어도 NodePort는 변하지 않음
$ kubectl delete pod <pod-name>
$ sleep 10
$ curl http://minikube-ip:30080  # 여전히 작동!
```

### ✅ 배운 것

- 모든 노드에서 같은 포트(NodePort)로 접근 가능
- Pod IP가 변해도 Service를 통해 자동 라우팅
- 클러스터 외부에서 접근 가능 (nodeIP:nodePort)
- 프로덕션에서는 LoadBalancer 권장

---

## 🚀 실습 C: DNS 서비스 발견

### 개념

- Kubernetes의 내부 DNS로 Service 이름 해석
- 같은 namespace: `service-name`
- 다른 namespace: `service-name.namespace.svc.cluster.local`

### 실습

```bash
# 1. DNS 테스트 Pod 실행
kubectl run -it dnsutils --image=nicolaka/netshoot --rm --restart=Never -- bash

# 2. nslookup으로 Service DNS 확인
nslookup nginx-service
# Expected: 10.0.123.45 (ClusterIP)

# 3. 다른 네임스페이스 Service 조회
nslookup nginx-service.kube-system.svc.cluster.local

# 4. curl로 Service 접근
curl http://nginx-service:80
# Expected: nginx 응답
```

### 기대 결과

```
# nslookup 출력
$ nslookup nginx-service
Server:         10.96.0.10
Address:        10.96.0.10#53

Name:   nginx-service.default.svc.cluster.local
Address: 10.0.123.45   ← ClusterIP

# curl 응답
$ curl http://nginx-service:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
```

### ✅ 배운 것

- Kubernetes 내부 DNS 자동 제공
- Service 이름으로 자동 DNS 해석
- FQDN: `service-name.namespace.svc.cluster.local`

---

## 🧹 정리

```bash
# 모든 리소스 삭제
kubectl delete deployment nginx-app
kubectl delete svc nginx-service nginx-nodeport
```

---

## 📚 다음 단계

- [docs/03-networking.md](../../docs/03-networking.md) - Ingress & NetworkPolicy
- [labs/](../) - 다른 실습 목록
