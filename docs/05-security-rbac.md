# 5️⃣ Kubernetes 보안 & RBAC 🔐

> **목표**: RBAC, ServiceAccount, SecurityContext로 보안 이해
> 
> **학습 시간**: 2-3시간 | **난이도**: ⭐⭐⭐ 중급

---

## 📋 목차
1. [RBAC - 역할 기반 접근 제어](#rbac--역할-기반-접근-제어)
2. [ServiceAccount - Pod 인증](#serviceaccount--pod-인증)
3. [SecurityContext - 컨테이너 보안](#securitycontext--컨테이너-보안)
4. [실습 가이드](#실습-가이드)

---

## RBAC - 역할 기반 접근 제어

**RBAC**는 **"누가(Subject)" "무엇을(Resource)" "어떻게(Verb)" 할 수 있는지** 제어합니다.

### 3가지 핵심 개념

```
사용자 (alice) 
    ↓ (할당되는 권한)
Role (Pod 조회, Pod 삭제 등 정의)
    ↓ (연결)
RoleBinding (alice ↔ Role 매핑)
```

---

## 실습 1: Role과 RoleBinding으로 사용자 권한 제어

### Step 1: Role 정의 (Pod 조회만 허용)

```bash
# 0️⃣ Role 생성: alice가 Pod 조회만 가능
kubectl apply -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]              # 기본 API 그룹 (v1)
  resources: ["pods"]          # Pod 리소스만
  verbs: ["get", "list", "watch"]  # 조회만 가능 (삭제 ❌)
EOF

# 확인
kubectl get roles
```

### Step 2: RoleBinding으로 권한 할당

```bash
# 1️⃣ RoleBinding 생성: alice 사용자에게 pod-reader Role 할당
kubectl apply -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: alice-read-pods
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-reader
subjects:
- kind: User
  name: alice
EOF

# 확인
kubectl get rolebindings
kubectl describe rolebinding alice-read-pods
```

### Step 3: 권한 확인

```bash
# 2️⃣ alice가 할 수 있는 것 확인
kubectl auth can-i get pods --as=alice
# yes

kubectl auth can-i list pods --as=alice
# yes

# 3️⃣ alice가 할 수 없는 것 확인
kubectl auth can-i delete pods --as=alice
# no

kubectl auth can-i create pods --as=alice
# no
```

### 기대 결과

```bash
$ kubectl get roles
NAME         CREATED AT
pod-reader   2026-07-23T10:15:30Z

$ kubectl get rolebindings
NAME              ROLE             AGE
alice-read-pods   Role/pod-reader  5s

$ kubectl auth can-i get pods --as=alice
yes

$ kubectl auth can-i delete pods --as=alice
no
```

---

## 실습 2: ServiceAccount와 Pod RBAC

**Pod도 API 서버에 접근할 때 인증이 필요합니다!**

### Step 1: ServiceAccount와 권한 생성

```bash
# 0️⃣ ServiceAccount + Role + RoleBinding 한번에 배포
kubectl apply -f - <<EOF
# 1. ServiceAccount 생성
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-reader
  namespace: default

---
# 2. Role: Pod 조회 권한만
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: read-pods
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]

---
# 3. RoleBinding: ServiceAccount에 Role 할당
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-read-pods
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: read-pods
subjects:
- kind: ServiceAccount
  name: app-reader
  namespace: default
EOF

# 확인
kubectl get sa,role,rolebinding
```

### Step 2: Pod에서 API 호출

```bash
# 1️⃣ 이전 Pod 삭제
kubectl delete pod api-access --ignore-not-found=true

# 2️⃣ ServiceAccount를 사용하는 Pod 배포
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: api-access
  namespace: default
spec:
  serviceAccountName: app-reader  # 이 ServiceAccount 사용
  containers:
  - name: kubectl-client
    image: bitnami/kubectl:latest
    command:
    - /bin/sh
    - -c
    - |
      # ServiceAccount 토큰으로 API 인증
      TOKEN=$(cat /run/secrets/kubernetes.io/serviceaccount/token)
      CACERT=/run/secrets/kubernetes.io/serviceaccount/ca.crt
      API_SERVER=https://kubernetes.default.svc.cluster.local
      
      echo "=== Pod 조회 (권한 있음) ==="
      curl -s --cacert $CACERT \
        -H "Authorization: Bearer $TOKEN" \
        $API_SERVER/api/v1/namespaces/default/pods | jq '.items[].metadata.name'
      
      echo ""
      echo "=== Pod 삭제 시도 (권한 없음) ==="
      curl -s -X DELETE --cacert $CACERT \
        -H "Authorization: Bearer $TOKEN" \
        $API_SERVER/api/v1/namespaces/default/pods/test-pod 2>&1 | jq '.message' || echo "Forbidden"
      
      sleep infinity
  restartPolicy: Never
EOF

# Pod이 실행될 때까지 대기
kubectl get pods api-access --watch

# 3️⃣ Pod 로그 확인
kubectl logs -f api-access
```

### 기대 결과

```bash
$ kubectl get pods api-access
NAME         READY   STATUS    RESTARTS   AGE
api-access   1/1     Running   0          10s

$ kubectl logs api-access
=== Pod 조회 (권한 있음) ===
"api-access"
"nginx-deployment-66b6c48dd5-4s5k2"

=== Pod 삭제 시도 (권한 없음) ===
"User "system:serviceaccount:default:app-reader" cannot delete resource..."
```

---

## 실습 3: SecurityContext로 컨테이너 보안 강화

**컨테이너를 root 권한이 아닌 일반 사용자로 실행하기**

### Step 1: 비안전한 Pod (root로 실행)

```bash
# 0️⃣ 기본 nginx Pod 배포 (root로 실행)
kubectl run unsafe-pod --image=nginx:1.25 --restart=Never

# 1️⃣ Pod 내부에서 실행 권한 확인
kubectl exec unsafe-pod -- id
# uid=0(root) gid=0(root) groups=0(root)  ← root!

# 2️⃣ Pod 파일시스템 쓰기 확인 (위험)
kubectl exec unsafe-pod -- touch /etc/test.txt
# (성공 - root이므로 어디든 쓰기 가능)

# 3️⃣ 정리
kubectl delete pod unsafe-pod
```

### Step 2: 안전한 Pod (제한된 권한)

```bash
# 4️⃣ SecurityContext를 적용한 Pod 배포
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  # Pod 레벨 SecurityContext
  securityContext:
    runAsUser: 1000              # UID 1000 (nobody가 아닌 앱 사용자)
    runAsGroup: 3000
    fsGroup: 2000                # 볼륨 마운트 그룹
  
  containers:
  - name: app
    image: nginx:1.25
    
    # Container 레벨 SecurityContext (더 강력한 제약)
    securityContext:
      allowPrivilegeEscalation: false      # 권한 상승 금지 (setuid 불가)
      readOnlyRootFilesystem: true         # 루트 파일시스템 읽기 전용
      capabilities:
        drop:
        - ALL                              # 모든 Linux 기능 제거
        add:
        - NET_BIND_SERVICE                 # 필요한 것만 추가
    
    # 임시 저장소 (읽기 전용 FS이므로 필요)
    volumeMounts:
    - name: cache
      mountPath: /var/cache/nginx
    - name: run
      mountPath: /var/run/nginx
  
  volumes:
  - name: cache
    emptyDir: {}
  - name: run
    emptyDir: {}
EOF

# 5️⃣ 안전한 Pod 확인
kubectl get pods secure-pod
```

### Step 3: 보안 검증

```bash
# 6️⃣ 실행 권한 확인 (root 아님)
kubectl exec secure-pod -- id
# uid=1000 gid=3000 groups=2000  ← 일반 사용자!

# 7️⃣ 루트 파일시스템 쓰기 시도 (실패)
kubectl exec secure-pod -- touch /test.txt
# Read-only file system

# 8️⃣ 권한 상승 시도 (실패)
kubectl exec secure-pod -- sudo id
# sudo: command not found (또는 불가능)

# 9️⃣ 임시 저장소에만 쓰기 가능
kubectl exec secure-pod -- touch /var/cache/nginx/test.txt
# (성공)

# 10️⃣ 정리
kubectl delete pod secure-pod
```

### 기대 결과

```bash
# 비안전한 Pod
$ kubectl exec unsafe-pod -- id
uid=0(root) gid=0(root) groups=0(root)

$ kubectl exec unsafe-pod -- touch /etc/test.txt
# 성공 (위험!)

# 안전한 Pod
$ kubectl exec secure-pod -- id
uid=1000 gid=3000 groups=2000

$ kubectl exec secure-pod -- touch /test.txt
Read-only file system

$ kubectl exec secure-pod -- touch /var/cache/nginx/test.txt
# 성공
```

---

## 실습 4: ClusterRole로 클러스터 전체 권한 제어

**Namespace 제약 없이 클러스터 전역 권한 설정**

```bash
# 0️⃣ ClusterRole 생성: 모든 리소스 조회 가능
kubectl apply -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-reader
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["get", "list", "watch"]

---
# 1️⃣ ClusterRoleBinding: bob 사용자에게 할당
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-reader-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-reader
subjects:
- kind: User
  name: bob
EOF

# 2️⃣ bob의 권한 확인
kubectl auth can-i get nodes --as=bob
# yes

kubectl auth can-i get pods --all-namespaces --as=bob
# yes

kubectl auth can-i delete nodes --as=bob
# no
```

---

## 🎯 핵심 요점

| 개념 | 범위 | 목적 |
|------|------|------|
| **Role** | Namespace 제한 | Namespace 내 권한 정의 |
| **ClusterRole** | 클러스터 전역 | 클러스터 전체 권한 정의 |
| **RoleBinding** | Namespace 제한 | Role을 사용자/SA에 할당 |
| **ClusterRoleBinding** | 클러스터 전역 | ClusterRole을 사용자/SA에 할당 |
| **ServiceAccount** | Pod 실행 시 | Pod 인증 ID (토큰 자동 마운트) |
| **SecurityContext** | Container/Pod 레벨 | 실행 권한, 파일시스템 보안 |

---

## 🚀 고급: RBAC 권한 조합

```bash
# 특정 동작만 허용하는 세분화된 Role
kubectl apply -f - <<EOF
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader-scaler
  namespace: default
rules:
# Pod 조회
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]

# Deployment 조회 및 수정 (스케일링)
- apiGroups: ["apps"]
  resources: ["deployments", "deployments/scale"]
  verbs: ["get", "list", "patch", "update"]

# 제외: Pod 삭제, Deployment 삭제 불가
EOF
```

---

**마지막 업데이트: 2026-07-23**
