# Lab 03: Kubernetes Service 타입 비교

이 Lab은 Kubernetes Service의 3가지 타입(ClusterIP, NodePort, LoadBalancer)을 **실제 운영 환경처럼** 파일 단위로 분리해서 학습합니다.

## 📁 파일 구조

```
03-service/
├── deployment.yaml           # 재사용 가능한 Nginx Deployment
├── service-clusterip.yaml    # 클러스터 내부 통신용
├── service-nodeport.yaml     # 외부 노드 접근용
├── service-loadbalancer.yaml # 외부 로드밸런서 노출용
└── README.md                 # 이 문서
```

---

## 🎯 학습 목표

| Service 타입 | 접근 범위 | 사용 시점 |
|----------|---------|----------|
| **ClusterIP** | 클러스터 내부만 | 내부 마이크로서비스 통신 |
| **NodePort** | 외부 + 노드 IP | 테스트/개발 환경 |
| **LoadBalancer** | 외부 + 로드밸런서 | 프로덕션 환경 |

---

## 📝 Step 1: Deployment 배포

모든 Service는 동일한 Deployment를 사용합니다.

```bash
# 1. Deployment 배포
kubectl apply -f deployment.yaml

# 2. Pod 생성 확인 (3개 replica)
kubectl get deployment web-server
kubectl get pods -l app=web-server

# 3. Pod IP 확인
kubectl get pods -l app=web-server -o wide
```

**기대 결과:**
```bash
$ kubectl get deployment web-server
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
web-server   3/3     3            3           10s

$ kubectl get pods -l app=web-server -o wide
NAME                          READY   STATUS    RESTARTS   AGE   IP
web-server-abc123-xyz1        1/1     Running   0          10s   10.244.1.5
web-server-abc123-xyz2        1/1     Running   0          10s   10.244.1.6
web-server-abc123-xyz3        1/1     Running   0          10s   10.244.2.7
```

---

## 🔵 Step 2: ClusterIP Service (클러스터 내부)

**사용 시나리오**: 마이크로서비스 간 내부 통신

```bash
# 1. ClusterIP Service 배포
kubectl apply -f service-clusterip.yaml

# 2. Service 확인 (ClusterIP 주소 확인)
kubectl get service web-server-clusterip
kubectl describe service web-server-clusterip

# 3. 클러스터 내부에서 접근 (다른 Pod에서)
kubectl run curl-test --image=curlimages/curl -it --rm -- curl http://web-server-clusterip

# 또는 exec를 사용해 기존 Pod에서 테스트
kubectl exec -it web-server-abc123-xyz1 -- curl http://web-server-clusterip

# 4. DNS 이름으로 접근 (FQDN)
kubectl exec -it web-server-abc123-xyz1 -- nslookup web-server-clusterip
kubectl exec -it web-server-abc123-xyz1 -- curl http://web-server-clusterip.default.svc.cluster.local
```

**기대 결과:**
```bash
$ kubectl get service web-server-clusterip
NAME                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
web-server-clusterip    ClusterIP   10.0.123.456    <none>        80/TCP    5s

$ kubectl exec -it web-server-abc123-xyz1 -- curl http://web-server-clusterip
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

**ClusterIP의 특징:**
- ✅ 클러스터 내부에서만 접근 가능
- ✅ 외부에서 접근 불가능
- ✅ Pod 재시작해도 ClusterIP는 유지
- ✅ DNS 이름으로 영구 접근 가능

---

## 🟢 Step 3: NodePort Service (노드 포트)

**사용 시나리오**: 개발/테스트 환경에서 외부 접근 필요

```bash
# 1. NodePort Service 배포
kubectl apply -f service-nodeport.yaml

# 2. NodePort 확인 (30080 포트)
kubectl get service web-server-nodeport
kubectl describe service web-server-nodeport

# 3. 노드 IP 확인
kubectl get nodes -o wide
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="ExternalIP")].address}')
echo $NODE_IP

# 4. 외부에서 접근 (로컬 머신에서)
curl http://$NODE_IP:30080

# 5. 클러스터 내부에서는 여전히 접근 가능
kubectl exec -it web-server-abc123-xyz1 -- curl http://web-server-nodeport

# 6. 클러스터 내부 DNS로도 접근
kubectl exec -it web-server-abc123-xyz1 -- curl http://web-server-nodeport.default.svc.cluster.local
```

**기대 결과:**
```bash
$ kubectl get service web-server-nodeport
NAME                 TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
web-server-nodeport  NodePort   10.0.234.567   <none>        80:30080/TCP     5s

$ kubectl describe service web-server-nodeport
Name:                     web-server-nodeport
Type:                     NodePort
IP:                       10.0.234.567
Port:                     http  80/TCP
TargetPort:               http/TCP
NodePort:                 http  30080/TCP
Endpoints:                10.244.1.5:80,10.244.1.6:80,10.244.2.7:80
```

**NodePort의 특징:**
- ✅ 노드의 30000-32767 포트를 통해 접근
- ✅ 모든 노드의 해당 포트로 접근 가능
- ✅ ClusterIP도 함께 생성됨 (내부 통신 가능)
- ✅ 외부 트래픽이 직접 Pod로 라우팅됨

---

## 🔧 AKS 환경에서 NodePort 테스트하기

AKS 환경에서는 노드에 EXTERNAL-IP가 없어서 직접 접근이 어렵습니다:

```bash
$ kubectl get nodes -o wide
NAME                                STATUS   ROLES    AGE    VERSION   INTERNAL-IP   EXTERNAL-IP
aks-agentpool-36284107-vmss00000a   Ready    <none>   13h    v1.35.5   10.224.0.7    <none>
aks-agentpool-36284107-vmss00000b   Ready    <none>   13h    v1.35.5   10.224.0.4    <none>
```

### 방법 1️⃣: kubectl port-forward (모든 환경)

**가장 간단하고 모든 환경에서 작동:**

```bash
# 0️⃣ 먼저 Deployment와 Service를 배포 (아직 안 했다면)
kubectl apply -f deployment.yaml
kubectl apply -f service-nodeport.yaml

# 배포 확인
kubectl get deployment web-server
kubectl get svc web-server-nodeport

# 1️⃣ Service로 port-forward 설정 (로컬 8080 → Service 80)
kubectl port-forward service/web-server-nodeport 8080:80 &

# 2️⃣ 로컬에서 접근
curl http://localhost:8080

# 3️⃣ 백그라운드 작업 종료
jobs
kill %1
```

**기대 결과:**
```bash
$ kubectl apply -f deployment.yaml
deployment.apps/web-server created

$ kubectl apply -f service-nodeport.yaml
service/web-server-nodeport created

$ kubectl get svc web-server-nodeport
NAME                 TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
web-server-nodeport  NodePort   10.0.234.567   <none>        80:30080/TCP     5s

$ kubectl port-forward service/web-server-nodeport 8080:80 &
[1] 12345
Forwarding from 127.0.0.1:8080 -> 80
Forwarding from [::1]:8080 -> 80

$ curl http://localhost:8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
```

### 방법 2️⃣: Pod 내부에서 테스트

**클러스터 내부의 다른 Pod에서 테스트:**

```bash
# 0️⃣ 먼저 Deployment와 Service를 배포 (아직 안 했다면)
kubectl apply -f deployment.yaml
kubectl apply -f service-nodeport.yaml

# 배포 확인
kubectl get pods -l app=web-server
kubectl get svc web-server-nodeport

# 1️⃣ 테스트용 Pod 실행
kubectl run test-curl --image=curlimages/curl -it --rm -- \
  curl http://web-server-nodeport:80

# 2️⃣ 또는 기존 Pod에서 직접 실행
POD_NAME=$(kubectl get pods -l app=web-server -o jsonpath='{.items[0].metadata.name}')
kubectl exec -it $POD_NAME -- curl http://web-server-nodeport:80
```

**기대 결과:**
```bash
$ kubectl apply -f deployment.yaml
deployment.apps/web-server created

$ kubectl apply -f service-nodeport.yaml
service/web-server-nodeport created

$ kubectl get pods -l app=web-server
NAME                          READY   STATUS    RESTARTS   AGE
web-server-abc123-xyz1        1/1     Running   0          10s
web-server-abc123-xyz2        1/1     Running   0          10s
web-server-abc123-xyz3        1/1     Running   0          10s

$ kubectl run test-curl --image=curlimages/curl -it --rm -- \
  curl http://web-server-nodeport:80
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
pod "test-curl" deleted
```

### 방법 3️⃣: AKS 노드에 직접 접근 (SSH)

**특정 AKS 노드에 SSH로 접근해서 테스트:**

```bash
# 0️⃣ 먼저 Deployment와 Service를 배포 (아직 안 했다면)
kubectl apply -f deployment.yaml
kubectl apply -f service-nodeport.yaml

# 배포 확인
kubectl get svc web-server-nodeport

# 1️⃣ Azure에서 노드에 접근하는 방법 설정
# (별도의 VM이나 Bastion 필요 - 보안상 제한)

# 2️⃣ kubectl debug를 사용한 노드 디버깅 Pod
kubectl debug node/aks-agentpool-36284107-vmss00000a -it --image=ubuntu

# 3️⃣ 디버그 Pod 내부에서 (NodePort 30080으로 접근)
curl http://10.224.0.7:30080
```

**기대 결과:**
```bash
$ kubectl apply -f deployment.yaml
deployment.apps/web-server created

$ kubectl apply -f service-nodeport.yaml
service/web-server-nodeport created

$ kubectl get svc web-server-nodeport
NAME                 TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
web-server-nodeport  NodePort   10.0.234.567   <none>        80:30080/TCP     5s

$ kubectl debug node/aks-agentpool-36284107-vmss00000a -it --image=ubuntu
Creating debugging pod node-debugger-aks-agentpool-... in namespace default
...
/root# curl http://10.224.0.7:30080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
```

### 방법 4️⃣: Azure Load Balancer 활용

**AKS에서 NodePort를 외부에 노출하려면 (프로덕션):**

```bash
# 0️⃣ 먼저 Deployment를 배포
kubectl apply -f deployment.yaml

# 1️⃣ LoadBalancer Service 사용 (권장)
kubectl apply -f service-loadbalancer.yaml

# 2️⃣ 외부 IP 확인 (Azure Load Balancer 생성됨)
kubectl get svc web-server-loadbalancer --watch

# 3️⃣ 외부에서 접근
EXTERNAL_IP=$(kubectl get svc web-server-loadbalancer -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
curl http://$EXTERNAL_IP
```

**기대 결과:**
```bash
$ kubectl apply -f deployment.yaml
deployment.apps/web-server created

$ kubectl apply -f service-loadbalancer.yaml
service/web-server-loadbalancer created

$ kubectl get svc web-server-loadbalancer --watch
NAME                       TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)        AGE
web-server-loadbalancer    LoadBalancer   10.0.345.678   <pending>        80:30567/TCP   3s
# 몇 초 후 Azure Load Balancer 공개 IP 할당됨
web-server-loadbalancer    LoadBalancer   10.0.345.678   40.71.123.100    80:30567/TCP   15s

$ EXTERNAL_IP=$(kubectl get svc web-server-loadbalancer -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
$ curl http://$EXTERNAL_IP
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
...
```

**Azure 리소스 생성 확인:**
```bash
# Azure Load Balancer 확인
az network lb list -o table

# 공개 IP 확인
az network public-ip list -o table

# 결과:
Name                                ResourceGroup  Location    AllocationMethod
----------------------------------  -----------  ----------  ------------------
kubernetes-40771234                 MC_rg_aks     eastus      Static
```

---

## 📊 환경별 테스트 방법 비교

| 방법 | 로컬 | Docker Desktop | AKS | EKS | GKE |
|------|------|---|---|---|---|
| **직접 IP 접근** | ✅ | ✅ | ❌ | ❌ | ❌ |
| **port-forward** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Pod에서 테스트** | ✅ | ✅ | ✅ | ✅ | ✅ |
| **LoadBalancer** | ⚠️ | ⚠️ | ✅ | ✅ | ✅ |

**결론**: AKS에서는 **port-forward** 또는 **Pod 내부 테스트** 추천!

---

## 🔴 Step 4: LoadBalancer Service (로드밸런서)

**사용 시나리오**: 프로덕션 환경, 외부 IP 필요

```bash
# 1. LoadBalancer Service 배포
kubectl apply -f service-loadbalancer.yaml

# 2. LoadBalancer 상태 확인
kubectl get service web-server-loadbalancer
kubectl describe service web-server-loadbalancer

# 3. EXTERNAL-IP가 할당될 때까지 대기 (클라우드 환경)
kubectl get service web-server-loadbalancer --watch

# 4. EXTERNAL-IP 주소로 접근
EXTERNAL_IP=$(kubectl get service web-server-loadbalancer -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
curl http://$EXTERNAL_IP

# 5. 클러스터 내부에서도 여전히 접근 가능
kubectl exec -it web-server-abc123-xyz1 -- curl http://web-server-loadbalancer
```

### AKS 환경에서의 LoadBalancer

**AKS에서 LoadBalancer를 배포하면 Azure Load Balancer가 자동으로 생성됩니다:**

```bash
$ kubectl get svc web-server-loadbalancer
NAME                       TYPE           CLUSTER-IP     EXTERNAL-IP      PORT(S)        AGE
web-server-loadbalancer    LoadBalancer   10.0.345.678   40.71.123.100    80:30567/TCP   15s

# EXTERNAL-IP에 실제 Azure Load Balancer의 공개 IP가 할당됨!

$ curl http://40.71.123.100
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
```

**또는 DNS 이름으로도 접근 가능:**

```bash
$ kubectl describe svc web-server-loadbalancer
...
LoadBalancer Ingress:     40.71.123.100
...

# Azure가 자동으로 생성한 DNS 이름
# <service-name>.eastus.cloudapp.azure.com 형식
```

### 로컬 테스트 환경에서:

```bash
$ kubectl get service web-server-loadbalancer
NAME                       TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
web-server-loadbalancer    LoadBalancer   10.0.345.678   <pending>     80:30123/TCP   10s
# EXTERNAL-IP가 <pending> 상태 (로컬 환경에서는 할당 안됨)

# port-forward로 테스트
kubectl port-forward service/web-server-loadbalancer 8080:80
curl http://localhost:8080
```

**LoadBalancer의 특징:**
- ✅ 클라우드 환경에서 실제 로드밸런서 IP 할당 (공개 IP)
- ✅ NodePort도 함께 생성됨
- ✅ ClusterIP도 함께 생성됨
- ✅ 프로덕션 환경에서 외부 노출의 표준
- ✅ AKS의 경우 Azure Load Balancer 자동 생성
- ✅ 비용 발생 (클라우드 제공자별로 청구)

---

## 🔄 Service 타입 비교 테이블

| 기능 | ClusterIP | NodePort | LoadBalancer |
|------|-----------|----------|-------------|
| 클러스터 내부 접근 | ✅ | ✅ | ✅ |
| 외부 접근 | ❌ | ✅ (NodeIP:30000~32767) | ✅ (외부 IP) |
| DNS 이름 | ✅ | ✅ | ✅ |
| 프로덕션 권장 | ✅ (내부용) | ❌ | ✅ (외부용) |
| 비용 | 무료 | 무료 | 사용료 있음 (클라우드) |
| 포트 범위 | 1~65535 | 30000~32767 | 1~65535 |

---

## 🧹 전체 정리

```bash
# 1. 만든 Service 모두 확인
kubectl get services -l app=web-server

# 2. 각 Service의 엔드포인트 확인
kubectl get endpoints -l app=web-server

# 3. 모든 리소스 정리
kubectl delete -f deployment.yaml
kubectl delete -f service-clusterip.yaml
kubectl delete -f service-nodeport.yaml
kubectl delete -f service-loadbalancer.yaml

# 또는 한 번에 정리 (namespace 삭제)
kubectl delete all -l app=web-server
```

**기대 결과:**
```bash
$ kubectl get services -l app=web-server
NAME                       TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
web-server-clusterip       ClusterIP      10.0.123.456    <none>        80/TCP           5m
web-server-nodeport        NodePort       10.0.234.567    <none>        80:30080/TCP     3m
web-server-loadbalancer    LoadBalancer   10.0.345.678    <pending>     80:30123/TCP     1m

$ kubectl get endpoints -l app=web-server
NAME                       ENDPOINTS                                          AGE
web-server-clusterip       10.244.1.5:80,10.244.1.6:80,10.244.2.7:80        5m
web-server-nodeport        10.244.1.5:80,10.244.1.6:80,10.244.2.7:80        3m
web-server-loadbalancer    10.244.1.5:80,10.244.1.6:80,10.244.2.7:80        1m
```

---

## 📌 실무 팁

### Service 선택 기준

```
시스템 설계할 때:

1️⃣ 마이크로서비스 내부 통신 → ClusterIP 사용
   (예: API Gateway ↔ Backend Service)

2️⃣ 개발/테스트 환경에서 외부 접근 필요 → NodePort 사용
   (예: 로컬에서 http://localhost:30080)
   또는 port-forward 사용 (모든 환경)

3️⃣ 프로덕션 환경 외부 노출 → LoadBalancer 사용
   (예: 실제 도메인, HTTPS, 트래픽 분산)
   - AKS: Azure Load Balancer 자동 생성
   - EKS: AWS Network Load Balancer / Application Load Balancer
   - GKE: Google Cloud Load Balancer

4️⃣ 고급: Ingress 컨트롤러 → 여러 Service를 하나의 IP로 관리
   (예: web.example.com → Service A, api.example.com → Service B)
```

### AKS 환경에서의 주의사항

**NodePort 테스트 시:**
```bash
# ❌ EXTERNAL-IP가 없으면 직접 접근 불가
kubectl get nodes -o wide
# EXTERNAL-IP 컬럼이 <none>

# ✅ 대신 이 방법들 사용:
# 1. port-forward (권장)
kubectl port-forward svc/web-server-nodeport 8080:80

# 2. Pod 내부에서 테스트
kubectl run test --image=curlimages/curl -it --rm -- curl http://web-server-nodeport

# 3. kubectl debug (노드 접근)
kubectl debug node/<node-name> -it --image=ubuntu
```

**LoadBalancer 배포 시:**
```bash
# AKS에서는 자동으로 Azure Load Balancer 생성
# 비용이 발생하므로 주의!

kubectl apply -f service-loadbalancer.yaml

# 공개 IP 확인
kubectl get svc web-server-loadbalancer

# 생성된 Azure 리소스 확인
az network lb list -o table
az network public-ip list -o table
```

---

### Service 디버깅

```bash
# Service와 Pod이 제대로 연결되었는지 확인
kubectl get endpoints web-server-clusterip

# Pod가 Service selector와 매칭되는지 확인
kubectl get pods -l app=web-server --show-labels

# Service 설정 상세 조회
kubectl get service web-server-clusterip -o yaml

# Pod에서 DNS 해석 확인
kubectl exec -it web-server-abc123-xyz1 -- nslookup web-server-clusterip
```

---

## 🎓 추가 학습 자료

- [공식 문서: Service](https://kubernetes.io/docs/concepts/services-networking/service/)
- [다음 단계: Ingress](../../docs/03-networking.md#ingress--외부-api-게이트웨이)
- [실습 Lab: Networking 심화](../04-networking/README.md)

---

**마지막 업데이트: 2026-07-22**
