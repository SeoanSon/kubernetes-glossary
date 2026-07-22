# 5️⃣ Kubernetes 보안 & RBAC 🔐

> **목표**: RBAC, ServiceAccount, SecurityContext로 보안 이해
> 
> **학습 시간**: 2-3시간 | **난이도**: ⭐⭐⭐ 중급

---

## 📋 목차
1. [RBAC - 역할 기반 접근 제어](#rbac--역할-기반-접근-제어)
2. [ServiceAccount - Pod 인증](#serviceaccount--pod-인증)
3. [SecurityContext - 컨테이너 보안](#securitycontext--컨테이너-보안)
4. [Secret - 민감 정보](#secret--민감-정보)

---

## RBAC - 역할 기반 접근 제어

**RBAC**는 **"누가(Subject)" "무엇을(Resource)" "어떻게(Verb)" 할 수 있는지** 제어합니다.

### 3가지 구성 요소

1️⃣ **Role** - 권한 정의
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]              # API 그룹
  resources: ["pods"]          # 리소스
  verbs: ["get", "list"]       # 동작 (GET, DELETE 등)
```

2️⃣ **RoleBinding** - 권한 할당
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-reader
subjects:
- kind: User
  name: alice
```

3️⃣ **권한 확인**
```bash
# alice가 할 수 있는 것 확인
kubectl auth can-i get pods --as=alice

# alice가 Pod 삭제 가능?
kubectl auth can-i delete pods --as=alice
```

### ClusterRole - 클러스터 전체 권한

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-admin
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
```

---

## ServiceAccount - Pod 인증

**Pod도 API 서버에 접근할 때 인증이 필요합니다!**

### Pod의 기본 ServiceAccount

```bash
# 모든 Pod는 자동으로 ServiceAccount 마운트
kubectl get serviceaccount
# default (기본)

# Pod 내에서 Token 확인
kubectl exec pod-name -- cat /run/secrets/kubernetes.io/serviceaccount/token
```

### Pod용 RBAC 설정

```yaml
# 1. ServiceAccount 생성
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-reader

---
# 2. Role 생성
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: read-pods
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]

---
# 3. RoleBinding 생성
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-read-pods
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: read-pods
subjects:
- kind: ServiceAccount
  name: app-reader

---
# 4. Deployment에서 ServiceAccount 사용
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  template:
    spec:
      serviceAccountName: app-reader    # 이 SA 사용
      containers:
      - name: app
        image: myapp:1.0
```

---

## SecurityContext - 컨테이너 보안

**SecurityContext**는 **컨테이너 실행 권한**을 제어합니다.

### Pod-level SecurityContext

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsUser: 1000              # UID 1000으로 실행 (root 아님)
    runAsGroup: 3000
    fsGroup: 2000                # 볼륨 그룹
  containers:
  - name: app
    image: myapp:1.0
```

### Container-level SecurityContext

```yaml
spec:
  containers:
  - name: app
    image: myapp:1.0
    securityContext:
      allowPrivilegeEscalation: false   # 권한 상승 금지
      readOnlyRootFilesystem: true      # 읽기 전용
      capabilities:
        drop:
        - ALL                           # 모든 Linux 기능 제거
        add:
        - NET_BIND_SERVICE              # 필요한 기능만 추가
```

---

## Secret - 민감 정보

**Secret**은 암호, 토큰, API 키 등을 **암호화**하여 저장합니다.

### 생성 방법

```bash
# 1. 명령어로 생성
kubectl create secret generic db-secret \
  --from-literal=username=root \
  --from-literal=password=mypassword

# 2. 파일에서
kubectl create secret generic tls-secret \
  --from-file=tls.crt=/path/to/cert \
  --from-file=tls.key=/path/to/key
```

### YAML로 생성

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: cm9vdA==            # base64 (root)
  password: bXlwYXNzd29yZA==    # base64 (mypassword)
```

### Pod에서 사용

**환경 변수로:**
```yaml
spec:
  containers:
  - name: app
    image: myapp:1.0
    env:
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: username
    - name: DB_PASS
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
```

**파일로:**
```yaml
spec:
  containers:
  - name: app
    image: myapp:1.0
    volumeMounts:
    - name: secret-vol
      mountPath: /etc/secrets
  volumes:
  - name: secret-vol
    secret:
      secretName: db-secret
```

---

## 🎯 핵심 요점

| 개념 | 목적 |
|------|------|
| **RBAC** | 사용자/Pod 권한 제어 |
| **ServiceAccount** | Pod 인증 ID |
| **SecurityContext** | 컨테이너 실행 권한 제어 |
| **Secret** | 민감 정보 암호화 저장 |

---

**마지막 업데이트: 2026-07-22**
