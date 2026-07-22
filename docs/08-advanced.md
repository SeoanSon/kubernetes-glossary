# 8️⃣ Kubernetes 고급 주제 🚀

> **목표**: CRD, Operator, Webhook의 Kubernetes 확장 방식 이해
> 
> **학습 시간**: 3-4시간 | **난이도**: ⭐⭐⭐⭐ 고급

---

## 📋 목차
1. [CRD (Custom Resource Definition)](#crd--custom-resource-definition)
2. [Operator Pattern](#operator-pattern)
3. [ValidatingWebhook & MutatingWebhook](#webhook)

---

## CRD - Custom Resource Definition

**CRD로 새로운 리소스 타입을 Kubernetes에 추가합니다.**

### 개념

```
기본 리소스:
  kubectl get pods, services, deployments

CRD로 추가:
  kubectl get databases ← 내가 정의한 리소스 타입!
  kubectl get backups
  kubectl get monitors
```

### 실습 1: CRD 정의 및 사용

```bash
# 0️⃣ Database라는 새로운 CRD 정의
kubectl apply -f - <<EOF
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.example.com
spec:
  group: example.com
  scope: Namespaced
  names:
    kind: Database
    plural: databases
    shortNames:
    - db
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              engine:
                type: string
                enum: ["postgres", "mysql", "mongodb"]
                description: "Database engine"
              version:
                type: string
                description: "Database version"
              size:
                type: string
                enum: ["small", "medium", "large"]
                description: "Instance size"
              backup:
                type: boolean
                description: "Enable automatic backup"
            required:
            - engine
            - version
EOF

# CRD 확인
kubectl get crd
kubectl describe crd databases.example.com
```

### Step 2: CRD 인스턴스 생성

```bash
# 1️⃣ 정의한 Database 리소스 생성
kubectl apply -f - <<EOF
apiVersion: example.com/v1
kind: Database
metadata:
  name: my-postgres-db
  namespace: default
spec:
  engine: postgres
  version: "14"
  size: medium
  backup: true

---
apiVersion: example.com/v1
kind: Database
metadata:
  name: app-mysql-db
spec:
  engine: mysql
  version: "8.0"
  size: large
  backup: true
EOF

# Database 리소스 확인
kubectl get databases
kubectl get db  # shortName으로도 조회 가능
kubectl describe database my-postgres-db
```

### Step 3: 스키마 검증

```bash
# 2️⃣ 유효하지 않은 Database 생성 시도 (engine이 enum에 없음)
kubectl apply -f - <<EOF
apiVersion: example.com/v1
kind: Database
metadata:
  name: invalid-db
spec:
  engine: redis  # ❌ enum에 없음 (postgres, mysql, mongodb만 가능)
  version: "7"
EOF

# 결과:
# error validating data: error validating "STDIN": error validating data: 
# spec.engine: Unsupported value: "redis": supported values: "postgres", "mysql", "mongodb"

# 3️⃣ 필수 필드 누락
kubectl apply -f - <<EOF
apiVersion: example.com/v1
kind: Database
metadata:
  name: incomplete-db
spec:
  engine: postgres
  # ❌ version 누락 (required)
EOF

# 결과:
# error validating data: error validating "STDIN": missing required field "version"
```

### 기대 결과

```bash
$ kubectl get crd
NAME                    CREATED AT
databases.example.com   2026-07-23T12:30:45Z

$ kubectl get databases
NAME              AGE
my-postgres-db    15s
app-mysql-db      12s

$ kubectl describe database my-postgres-db
Name:         my-postgres-db
Namespace:    default
Labels:       <none>
Spec:
  Engine:    postgres
  Version:   14
  Size:      medium
  Backup:    true
```

---

## Operator Pattern

**Operator는 CRD를 감시하고 자동으로 처리하는 Custom Controller입니다.**

### 개념

```
User가 Database CRD 생성
    ↓
Operator가 감시
    ↓
Pod, PVC, ConfigMap 등 자동으로 생성
    ↓
Database 실행
```

### 예: 간단한 Operator 시뮬레이션

```bash
# 0️⃣ 준비: 위에서 생성한 Database CRD 사용

# 1️⃣ Operator 역할을 하는 스크립트 (실제로는 Go/Python으로 작성)
# Pod에서 Database CRD 감시
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: database-operator
spec:
  serviceAccountName: operator-sa
  containers:
  - name: operator
    image: bitnami/kubectl:latest
    command:
    - /bin/bash
    - -c
    - |
      echo "=== Database Operator Started ==="
      
      # Database CRD 감시
      kubectl get databases --watch -o jsonpath='{.items[*].metadata.name}'
      
      # 각 Database에 대해 Pod 생성
      for db in $(kubectl get databases -o jsonpath='{.items[*].metadata.name}'); do
        echo "Processing database: $db"
        
        # Database 정보 추출
        ENGINE=$(kubectl get database $db -o jsonpath='{.spec.engine}')
        VERSION=$(kubectl get database $db -o jsonpath='{.spec.version}')
        
        echo "Creating Pod for $db: $ENGINE $VERSION"
        
        # Pod 자동 생성
        kubectl run ${db}-pod --image=${ENGINE}:${VERSION} --restart=Never 2>/dev/null || true
      done
      
      sleep infinity
  restartPolicy: Never

---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: operator-sa

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: operator
rules:
- apiGroups: ["example.com"]
  resources: ["databases"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["create", "delete", "get", "list", "watch"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: operator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: operator
subjects:
- kind: ServiceAccount
  name: operator-sa
EOF

# 2️⃣ Operator 실행 확인
kubectl logs -f database-operator

# 3️⃣ Operator가 자동으로 생성한 Pod 확인
kubectl get pods
```

---

## Webhook

**Webhook으로 API 요청을 가로채 검증(Validating) 또는 변조(Mutating)합니다.**

### Validating Webhook (검증만)

```
Pod 생성 요청 → Validating Webhook → 정책 검증 → 수락/거절

예: "모든 Pod은 livenessProbe가 있어야 함"
```

### MutatingWebhook (자동 수정)

```
Pod 생성 요청 → MutatingWebhook → 자동 수정 → Pod 생성

예: "모든 Pod에 기본 리소스 request/limit 추가"
```

### 실습: ValidatingWebhook 구현 (시뮬레이션)

```bash
# 0️⃣ Validating Webhook 배포 (실제로는 외부 서버)
kubectl apply -f - <<EOF
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: pod-validation
webhooks:
- name: pod.example.com
  clientConfig:
    url: https://webhook-server:8443/validate  # 실제로는 외부 서버
  rules:
  - operations: ["CREATE"]
    apiGroups: [""]
    apiVersions: ["v1"]
    resources: ["pods"]
  failurePolicy: Fail  # 검증 실패하면 Pod 생성 차단
  admissionReviewVersions: ["v1"]
  sideEffects: None
EOF

# 주의: 실제 Webhook 서버가 없으면 모든 Pod 생성이 실패합니다!
# 테스트를 위해 webhook을 삭제합니다.
kubectl delete validatingwebhookconfigurations pod-validation 2>/dev/null || true
```

### 실습: MutatingWebhook 시뮬레이션

```bash
# Webhook 없이 정책을 코드로 구현
# MutatingAdmissionController 역할을 하는 Operator

# 1️⃣ Pod를 생성할 때 자동으로 리소스 설정 추가
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: app-with-defaults
spec:
  containers:
  - name: app
    image: nginx:1.25
    # Mutating Webhook이 아래를 자동 추가:
    # resources:
    #   requests:
    #     memory: "128Mi"
    #     cpu: "100m"
    #   limits:
    #     memory: "256Mi"
    #     cpu: "500m"
EOF
```

---

## 🎯 핵심 요점

| 개념 | 목적 | 예제 |
|------|------|------|
| **CRD** | 새로운 리소스 타입 정의 | Database, Backup, Monitor |
| **Operator** | CRD를 자동으로 처리하는 컨트롤러 | Prometheus Operator, MySQL Operator |
| **ValidatingWebhook** | API 요청 검증 | "모든 Pod은 livenessProbe 필수" |
| **MutatingWebhook** | API 요청 자동 수정 | "모든 Pod에 기본 리소스 추가" |

---

## 🚀 실제 예제: Prometheus Operator

```bash
# 1. Prometheus Operator를 설치하면:
kubectl apply -f https://github.com/prometheus-operator/prometheus-operator/...

# 2. 새로운 CRD가 추가됨:
kubectl get crd | grep prometheus
# monitoring.coreos.com/prometheuses
# monitoring.coreos.com/servicemonitors
# monitoring.coreos.com/rules

# 3. 새로운 리소스 사용:
kubectl apply -f - <<EOF
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: nginx-monitor
spec:
  selector:
    matchLabels:
      app: nginx
  endpoints:
  - port: metrics
    interval: 30s
EOF
```

---

## 📚 권장 학습 경로

1. ✅ CRD 정의 및 스키마 검증 (위 실습)
2. ✅ Operator 패턴 이해 (위 실습)
3. 🔜 실제 Operator 코드 작성 (Go/Python)
4. 🔜 Webhook 서버 구현
5. 🔜 KUDO, Helm으로 Operator 배포 자동화

---

**마지막 업데이트: 2026-07-23**
