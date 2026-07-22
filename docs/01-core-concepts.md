# 1️⃣ Kubernetes 핵심 개념 📌

> **목표**: Pod, Container, Node, Namespace의 기본 개념 이해
> 
> **학습 시간**: 1-2시간 | **난이도**: ⭐ 기초

---

## 📋 목차
1. [Kubernetes란?](#kubernetes란)
2. [핵심 개념](#핵심-개념)
3. [아키텍처](#아키텍처)
4. [kubectl 기초](#kubectl-기초)
5. [API Resources](#api-resources)
6. [라이프사이클](#라이프사이클)

---

## Kubernetes란?

**Kubernetes (K8s)**는 **컨테이너 오케스트레이션 플랫폼**입니다.

```
🎯 역할: 컨테이너 배포, 스케일링, 관리 자동화

예를 들어...
├─ Docker는 "단일 머신에서 컨테이너 실행"
└─ Kubernetes는 "여러 머신에서 컨테이너 통합 관리"
```

---

## 핵심 개념

### 1. Pod 🐳

**Pod**는 Kubernetes의 **최소 배포 단위**입니다.

**특징:**
- 1개 이상의 컨테이너를 포함
- 같은 Pod 내 컨테이너들은 **네트워크 네임스페이스 공유** (localhost로 통신)
- 일반적으로 1 Pod = 1 Application Container

**예제:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  namespace: default
spec:
  containers:
  - name: nginx
    image: nginx:1.14.2
    ports:
    - containerPort: 80
```

**언제 직접 Pod를 만드나요?**
- ❌ 거의 안 함 (Deployment/StatefulSet 사용)
- ✅ 테스트/디버깅용으로만 사용

---

### 2. Container 🐳

**Container**는 애플리케이션을 실행하는 **격리된 환경**입니다.

**특징:**
- 파일시스템, 네트워크, 프로세스 격리
- Docker, containerd 등이 구현
- Pod 내에 여러 개 가능

```yaml
spec:
  containers:
  - name: app
    image: myapp:1.0          # 이미지
    ports:
    - containerPort: 8080
    env:
    - name: ENV_VAR
      value: "production"
```

---

### 3. Node 🖥️

**Node**는 **Pod가 실행되는 워커 머신**입니다.

**구성:**
- kubelet - 노드의 K8s 에이전트
- Container runtime - Docker, containerd 등
- kube-proxy - 네트워킹 담당

```
┌─ Kubernetes Cluster ────────────────────┐
│  ┌─ Control Plane ─────────────────┐   │
│  │ API Server, Scheduler, etcd     │   │
│  └─────────────────────────────────┘   │
│                                         │
│  ┌─ Node 1 ─────────────────────────┐  │
│  │ kubelet                           │  │
│  │ ┌─ Pod 1 ─────┐  ┌─ Pod 2 ──┐  │  │
│  │ │ Container   │  │Container │  │  │
│  │ └─────────────┘  └──────────┘  │  │
│  └─────────────────────────────────┘  │
│                                         │
│  ┌─ Node 2 ─────────────────────────┐  │
│  │ kubelet                           │  │
│  │ ┌─ Pod 3 ─────┐                  │  │
│  │ │ Container   │                  │  │
│  │ └─────────────┘                  │  │
│  └─────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

**Node 확인:**
```bash
kubectl get nodes
kubectl describe node node-1
```

---

### 4. Namespace 🏢

**Namespace**는 **가상 클러스터** (격리된 환경)입니다.

**목적:**
- 여러 팀/프로젝트 격리
- 같은 이름의 리소스를 다른 네임스페이스에 허용
- 리소스 사용량 제한

**기본 Namespace:**
- `default` - 기본값
- `kube-system` - Kubernetes 시스템 Pod
- `kube-public` - 공개 정보
- `kube-node-lease` - Node 상태

```bash
# 네임스페이스 조회
kubectl get namespaces

# 특정 네임스페이스의 Pod 조회
kubectl get pods --namespace production

# 네임스페이스 생성
kubectl create namespace my-app
```

---

### 5. Label & Selector 🏷️

**Label**은 **키-값 쌍**으로 리소스를 식별합니다.

```yaml
metadata:
  labels:
    app: nginx           # 애플리케이션 이름
    version: v1.0        # 버전
    tier: frontend       # 계층
    environment: prod    # 환경
```

**Selector**로 Label을 기반 리소스 선택:

```bash
# Label이 app=nginx인 Pod 선택
kubectl get pods -l app=nginx

# 여러 조건
kubectl get pods -l 'app=nginx,tier=frontend'

# 특정 Label이 없는 Pod
kubectl get pods -l 'environment!=prod'
```

**Use Case:**
- Service가 Pod 선택
- Deployment가 관리할 Pod 선택
- 모니터링/로깅 필터링

---

### 6. API Resources 📚

Kubernetes의 모든 객체는 **API Resource**입니다.

```bash
# 지원 리소스 확인
kubectl api-resources

# 특정 리소스 정보
kubectl explain pod
kubectl explain deployment.spec.template.spec
```

**주요 API 그룹:**
- `v1` - 핵심 (Pod, Service, Node, Namespace)
- `apps/v1` - 워크로드 (Deployment, StatefulSet, DaemonSet)
- `batch/v1` - 배치 (Job, CronJob)
- `networking.k8s.io/v1` - 네트워킹 (NetworkPolicy, Ingress)
- `rbac.authorization.k8s.io/v1` - 보안 (Role, RoleBinding)

---

## 아키텍처

### Control Plane (마스터)

**역할**: 클러스터 제어 및 의사 결정

| 컴포넌트 | 역할 |
|---------|------|
| **API Server** | 모든 요청 처리, etcd와 통신 |
| **Scheduler** | Pod를 노드에 배치 |
| **Controller Manager** | 원하는 상태 유지 (Deployment → ReplicaSet → Pod) |
| **etcd** | 모든 상태 저장 (key-value store) |

### 데이터 플레인 (워커 노드)

**역할**: Pod 실행

| 컴포넌트 | 역할 |
|---------|------|
| **kubelet** | 노드의 에이전트, Pod 관리 |
| **kube-proxy** | 네트워킹, Service 구현 |
| **Container Runtime** | 컨테이너 실행 (Docker, containerd) |

---

## kubectl 기초

### 기본 명령어

```bash
# 조회
kubectl get pods
kubectl get pods -A                    # 모든 네임스페이스
kubectl get pods -o wide              # 상세 정보
kubectl describe pod my-pod            # 상세 설명

# 생성/수정
kubectl apply -f pod.yaml              # 파일로 생성
kubectl create pod my-pod --image=nginx

# 삭제
kubectl delete pod my-pod
kubectl delete pod -l app=nginx        # Label로 삭제

# 로그/실행
kubectl logs my-pod                    # 컨테이너 로그
kubectl exec -it my-pod -- bash        # 컨테이너 접속
kubectl port-forward pod/my-pod 8080:80  # 포트 포워딩
```

### Context & Namespace

```bash
# 현재 context 확인
kubectl config current-context

# Context 전환
kubectl config use-context my-cluster

# 기본 namespace 설정
kubectl config set-context --current --namespace=production

# 명령어에서 namespace 지정
kubectl get pods -n production
```

---

## API Resources

### Manifest 구조

모든 Kubernetes 리소스는 동일한 구조:

```yaml
apiVersion: v1                    # API 버전
kind: Pod                         # 리소스 종류
metadata:                         # 메타데이터
  name: my-pod                    # 리소스 이름
  namespace: default              # 네임스페이스
  labels:                         # 레이블
    app: my-app
  annotations:                    # 주석 (설명용)
    description: "My Pod"
spec:                             # 리소스 명세
  containers:
  - name: app
    image: myapp:1.0
```

### Owner Reference

Pod는 누군가의 **소유**입니다.

```bash
# Pod의 소유자 확인
kubectl get pod my-pod -o yaml | grep -A 5 ownerReferences
```

소유자가 삭제되면 Pod도 삭제됨 (Garbage Collection):
```
User → Deployment → ReplicaSet → Pod
 (삭제)              (자동 삭제)  (자동 삭제)
```

---

## 라이프사이클

### Pod 라이프사이클

```
Pending → Running → Succeeded/Failed
  ↓
  (초기화 진행 중)
```

**상태:**
- `Pending` - 스케줄 대기 중 또는 이미지 다운로드 중
- `Running` - 최소 1개 컨테이너 실행 중
- `Succeeded` - 모든 컨테이너 정상 종료
- `Failed` - 최소 1개 컨테이너 실패로 종료
- `Unknown` - 상태 불명확

```bash
kubectl get pods -o wide  # STATUS 컬럼 확인
```

### Container 상태

```
Waiting → Running → Terminated
```

**상태 확인:**
```bash
kubectl describe pod my-pod  # 상세 상태 보기
```

---

## 🧪 직접 실습해보기

### Pod 생성 및 관리 실습

```bash
# 1. Nginx Pod 생성
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: core-concepts-demo
  namespace: default
  labels:
    app: demo
    tier: test
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
EOF

# 2. Pod 확인
kubectl get pods
kubectl get pods -o wide
kubectl describe pod core-concepts-demo

# 3. Label로 Pod 찾기
kubectl get pods -l app=demo
kubectl get pods -l 'tier=test'

# 4. Pod 로그 확인
kubectl logs core-concepts-demo

# 5. Pod 내부 접속
kubectl exec -it core-concepts-demo -- bash
# 또는 명령어 실행만
kubectl exec core-concepts-demo -- curl localhost

# 6. 포트 포워딩 (외부에서 접근)
kubectl port-forward pod/core-concepts-demo 8080:80 &
# curl http://localhost:8080

# 7. Pod 삭제
kubectl delete pod core-concepts-demo
```

**기대 결과:**
```bash
$ kubectl get pods -o wide
NAME                  READY   STATUS    RESTARTS   AGE   NODE
core-concepts-demo    1/1     Running   0          10s   aks-node-1

$ kubectl describe pod core-concepts-demo
Name:         core-concepts-demo
Namespace:    default
Status:       Running
IP:           10.244.1.5
...

$ kubectl exec core-concepts-demo -- curl localhost
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
```

### Namespace 실습

```bash
# 1. Namespace 생성
kubectl create namespace test-namespace

# 2. 다른 Namespace에 Pod 생성
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: ns-demo
  namespace: test-namespace
spec:
  containers:
  - name: busybox
    image: busybox:1.35
    command: ['sleep', '3600']
EOF

# 3. Namespace별 Pod 확인
kubectl get pods                          # default namespace
kubectl get pods -n test-namespace        # test-namespace
kubectl get pods -A                       # 모든 namespace

# 4. 정리
kubectl delete namespace test-namespace   # Pod도 함께 삭제
```

**기대 결과:**
```bash
$ kubectl get pods -A
NAMESPACE              NAME                    READY   STATUS
default                core-concepts-demo      1/1     Running
test-namespace         ns-demo                 1/1     Running
kube-system            coredns-558bd4d5c9...  1/1     Running
```

---

## 🎯 핵심 요점

| 개념 | 핵심 포인트 |
|------|-----------|
| **Pod** | Kubernetes 최소 단위 |
| **Container** | Pod 내부의 애플리케이션 |
| **Node** | Pod이 실행되는 머신 |
| **Namespace** | 가상 클러스터 격리 |
| **Label** | 리소스 식별 및 선택 |
| **API Resources** | YAML로 정의된 모든 객체 |

---

## 📌 다음 단계

1. **기본 명령어 숙달** → [kubectl 레퍼런스](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands/)
2. **Deployment 학습** → [워크로드](02-workloads.md)
3. **Service 이해** → [네트워킹](03-networking.md)

---

**마지막 업데이트: 2026-07-22**
