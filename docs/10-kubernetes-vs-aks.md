# 📊 Kubernetes vs AKS 비교 가이드

> **목표**: 순수 Kubernetes와 Azure Kubernetes Service(AKS)의 차이 이해
> 
> **학습 시간**: 1.5-2시간 | **난이도**: ⭐⭐⭐ 중급

---

## 📋 목차
1. [개괄 비교](#개괄-비교)
2. [주요 차이점](#주요-차이점)
3. [네트워킹 비교](#네트워킹-비교)
4. [저장소 비교](#저장소-비교)
5. [보안 & RBAC](#보안--rbac)
6. [모니터링 & 로깅](#모니터링--로깅)
7. [관리형 서비스](#관리형-서비스)

---

## 개괄 비교

### Kubernetes (순수 K8s)
- **자체 관리** (Self-managed)
- **모든 것을 직접 구축** (Control Plane 포함)
- **높은 자유도**, 낮은 편의성
- **VM, 베어메탈, 온프레미스 등** 어디서나 실행
- **비용 절감** (오픈소스)

### AKS (Azure Kubernetes Service)
- **관리형 Kubernetes** (Managed Service)
- **Control Plane은 Microsoft가 관리**
- **낮은 운영 부담**, 높은 편의성
- **Azure 에코시스템과 통합**
- **SLA 보장** (99.95%)

---

## 주요 차이점

| 항목 | 순수 K8s | AKS |
|------|---------|-----|
| **Control Plane** | 직접 관리 필요 | Microsoft 관리 |
| **API Server** | 자신의 인프라 | Azure 호스팅 |
| **etcd** | 직접 백업 필요 | 자동 백업 |
| **업그레이드** | 수동 (다운타임) | 자동 또는 무중단 |
| **고가용성** | 직접 구성 | 기본 제공 |
| **네트워킹** | 플러그인 선택 (Flannel, Weave, Cilium) | Azure CNI 기본 (Cilium 선택 가능) |
| **저장소** | 자유로운 선택 | Azure Blob, Files, Managed Disk |
| **모니터링** | Prometheus 직접 설치 | Container Insights 기본 제공 |
| **인증** | 기본 인증 (Kubeconfig) | Entra ID 통합 |
| **네트워크 정책** | 플러그인 필요 | Cilium 또는 기본 정책 |

---

## 네트워킹 비교

### 순수 Kubernetes

**CNI 플러그인 선택:**
```yaml
# 1. Flannel (기본, 간단)
# 2. Weave (보안, 암호화)
# 3. Cilium (eBPF, 성능, 보안)
```

**Service Discovery:**
```bash
# CoreDNS + kube-proxy (기본)
# 또는 커스텀 DNS 솔루션
```

**Ingress:**
```bash
# 직접 설치 필요 (Nginx, Traefik, Istio 등)
```

### AKS

**네트워킹 옵션 (설치 시 선택):**
```yaml
# 1. Azure CNI (기본)
# - Pod이 VNet IP 사용
# - Azure Native 통합

# 2. Azure CNI Overlay
# - Pod은 별도 CIDR (오버레이)
# - 효율적인 IP 사용

# 3. Cilium (eBPF 기반)
# - 고성능, 보안 정책
# - 네트워크 정책 고도 지원
```

**Service Discovery:**
```bash
# CoreDNS + Azure DNS 통합
# service.namespace.svc.cluster.local 자동 해석
```

**Ingress:**
```bash
# Application Gateway Ingress Controller (AGIC)
# 또는 Nginx 직접 설치
```

**예제 비교:**

순수 K8s + Cilium:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

AKS + Cilium:
```yaml
# 동일한 NetworkPolicy
# 하지만 Cilium이 자동 설치됨
# kubeconfig에 Cilium 옵션 설정 가능
```

---

## 저장소 비교

### 순수 Kubernetes

**Volume 옵션:**
```yaml
# 1. hostPath - 노드 로컬 저장소
# 2. emptyDir - 임시 저장소
# 3. NFS - 네트워크 저장소
# 4. iSCSI, FC - 기업 저장소
# (CSI 드라이버 필요)
```

**Dynamic Provisioning:**
```bash
# StorageClass와 CSI 드라이버 필요
# 클라우드 제공자별로 다름
```

### AKS

**Volume 옵션 (기본 제공):**
```yaml
# 1. Azure Managed Disk
# 2. Azure Blob Storage
# 3. Azure Files (SMB)
# 4. Azure Disk CSI 드라이버 (기본)
```

**Dynamic Provisioning (자동):**
```bash
# StorageClass 자동 제공
# az-managed-premium, az-managed, standard 등
```

**예제:**

순수 K8s:
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: nfs-storage
provisioner: nfs.io/nfs
```

AKS:
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-premium
provisioner: disk.csi.azure.com
parameters:
  skuName: Premium_LRS
  cachingMode: ReadOnly
```

---

## 보안 & RBAC

### 순수 Kubernetes

**인증:**
```bash
# 1. X.509 Client Certificates (kubeconfig)
# 2. Static Token File (dev/test용)
# 3. Service Account Token (Pod)
```

**권한 관리:**
```bash
# RBAC 직접 구성
# Role, RoleBinding, ClusterRole, ClusterRoleBinding
```

**외부 인증:**
```bash
# 별도 구성 필요 (OpenID Connect, LDAP 등)
```

### AKS

**인증:**
```bash
# 1. Azure Entra ID (Microsoft Entra)
# 2. OIDC 자동 통합
# 3. Service Account (동일)
```

**권한 관리:**
```bash
# RBAC + Azure RBAC 통합
# AAD 그룹으로 권한 관리 가능
```

**예제:**

AKS Entra ID 통합:
```bash
# 1. kubeconfig 다운로드
az aks get-credentials --resource-group myRG --name myCluster

# 2. kubectl 실행 (자동으로 Entra 인증)
kubectl get pods  # 브라우저로 로그인 프롬프트

# 3. AAD 그룹으로 권한 제어
az aks update --resource-group myRG --name myCluster \
  --enable-aad --aad-admin-group-object-ids <group-id>
```

---

## 모니터링 & 로깅

### 순수 Kubernetes

**메트릭 수집:**
```bash
# Prometheus + Node Exporter 직접 설치
# 시간 시계열 데이터베이스 구성
```

**로깅:**
```bash
# ELK (Elasticsearch, Logstash, Kibana)
# 또는 Loki + Grafana
# 직접 구성, 관리 필요
```

**예제:**
```yaml
# Prometheus 설치
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm install prometheus prometheus-community/kube-prometheus-stack
```

### AKS

**메트릭 수집:**
```bash
# Container Insights (Azure Monitor)
# 자동 에이전트 배포
# Prometheus 메트릭도 스크래핑 가능
```

**로깅:**
```bash
# Log Analytics Workspace
# 자동 로그 수집
# KQL 쿼리로 분석
```

**예제:**
```bash
# Container Insights 활성화
az aks enable-addons --resource-group myRG \
  --name myCluster \
  --addons monitoring

# Log Analytics에서 KQL 쿼리
KQL
| where TimeGenerated > ago(1h)
| summarize count() by ContainerName
```

**Azure Portal 대시보드:**
```
Azure Portal
  → AKS 클러스터
    → Insights (자동 제공)
      → Performance, Logs, Events
```

---

## 관리형 서비스

### 순수 Kubernetes
- ❌ 관리형 데이터베이스 통합 없음
- ❌ 레지스트리 자동 연결 안 됨
- ❌ CI/CD 파이프라인 미제공

### AKS
- ✅ **Azure Container Registry (ACR)** 통합
- ✅ **Azure DevOps** CI/CD
- ✅ **Managed Database** (PostgreSQL, MySQL, SQL)
- ✅ **Key Vault** 통합
- ✅ **Azure Policy** 거버넌스
- ✅ **Network Policies** 기본 제공

**예제: ACR 통합**
```bash
# AKS와 ACR 연결
az aks update --resource-group myRG \
  --name myCluster \
  --attach-acr myRegistry

# Pod에서 이미지 자동 pull
kubectl run my-app --image myRegistry.azurecr.io/my-app:1.0
```

---

## 🎯 선택 가이드

### 순수 Kubernetes 선택:
- 온프레미스 또는 다중 클라우드
- 최대 커스터마이징 필요
- 운영 팀의 K8s 전문성 높음
- 비용 최소화 목표

### AKS 선택:
- Azure 에코시스템 사용 중
- 운영 부담 최소화
- 빠른 구축 필요
- SLA 보장 필요
- 통합 모니터링 원함

---

## 🔄 순수 K8s → AKS 마이그레이션

### 호환성: ✅ 매우 높음

```yaml
# 순수 K8s의 Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - name: app
        image: my-app:1.0

# AKS에서도 동일하게 작동!
# (YAML 변경 필요 없음)
```

### 마이그레이션 단계:

```
1. 현재 Deployment 내보내기
   kubectl get all -o yaml > manifests.yaml

2. AKS 클러스터 생성
   az aks create --resource-group myRG --name myCluster

3. 동일한 매니페스트 적용
   kubectl apply -f manifests.yaml

4. 차이점 조정 (선택사항)
   - Storage 클래스 변경
   - NetworkPolicy 추가
   - Container Insights 활성화
```

---

## 📚 다음 단계

1. **이 문서 읽고** → 개념 이해
2. **AKS 네트워킹 가이드 학습** → [aks-networking-hands-on](https://github.com/SeoanSon/aks-networking-hands-on)
3. **개발 환경** → 순수 K8s (Minikube/KinD) vs AKS 실습
4. **실무 적용** → 회사 워크로드를 K8s → AKS로 이동

---

**마지막 업데이트: 2026-07-22**
