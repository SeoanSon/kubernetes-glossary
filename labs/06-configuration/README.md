# 🧪 Lab 06: Kubernetes 설정 관리 실습

> **목표**: ConfigMap과 Secret으로 앱 설정 주입하기
>
> **소요 시간**: 1시간 | **난이도**: ⭐ 기초

---

## 📖 개념

### 설정 관리가 필요한 이유

```
문제: 앱 설정을 YAML에 하드코딩하면
     환경마다 이미지를 다시 빌드해야 함
     
해결: ConfigMap (설정) + Secret (민감 정보)로 분리
     같은 이미지로 다양한 환경 지원
```

---

## 🚀 실습 A: ConfigMap (설정 데이터)

### 개념

- 환경 변수, 설정 파일을 저장
- 평문 저장 (민감 정보 ❌)
- Pod 시작 시 주입

### 완전한 배포 YAML

```yaml
# 1. ConfigMap 생성
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  # 간단한 key-value
  APP_ENV: production
  DB_HOST: postgres.default.svc.cluster.local
  DB_PORT: "5432"
  LOG_LEVEL: info

  # 파일 형식
  application.properties: |
    app.name=My App
    app.version=1.0
    app.debug=false

---
# 2. Pod이 ConfigMap 사용 (환경변수)
apiVersion: v1
kind: Pod
metadata:
  name: configmap-env-demo
spec:
  containers:
  - name: app
    image: busybox:1.35
    command:
    - /bin/sh
    - -c
    - |
      echo "=== ConfigMap as Environment Variables ==="
      echo "APP_ENV: $APP_ENV"
      echo "DB_HOST: $DB_HOST"
      echo "DB_PORT: $DB_PORT"
      echo "LOG_LEVEL: $LOG_LEVEL"
      sleep 3600
    env:
    # 개별 변수로 주입
    - name: APP_ENV
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_ENV
    - name: DB_HOST
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: DB_HOST
    - name: DB_PORT
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: DB_PORT
    - name: LOG_LEVEL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: LOG_LEVEL
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"

---
# 3. Pod이 ConfigMap 사용 (파일 마운트)
apiVersion: v1
kind: Pod
metadata:
  name: configmap-file-demo
spec:
  containers:
  - name: app
    image: busybox:1.35
    command:
    - /bin/sh
    - -c
    - |
      echo "=== ConfigMap as Mounted Files ==="
      echo "Contents of /etc/config/application.properties:"
      cat /etc/config/application.properties
      sleep 3600
    volumeMounts:
    - name: config
      mountPath: /etc/config

  volumes:
  - name: config
    configMap:
      name: app-config
      items:
      - key: application.properties
        path: application.properties
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
```

### 실습 시작

```bash
# 1. ConfigMap 생성
kubectl apply -f - <<EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_ENV: production
  DB_HOST: postgres.default.svc.cluster.local
  DB_PORT: "5432"
  LOG_LEVEL: info
  application.properties: |
    app.name=My App
    app.version=1.0
    app.debug=false
EOF

# 2. ConfigMap 확인
kubectl get configmap app-config
kubectl describe configmap app-config

# 3. ConfigMap을 환경변수로 주입하는 Pod
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: configmap-env-demo
spec:
  containers:
  - name: app
    image: busybox:1.35
    command:
    - /bin/sh
    - -c
    - |
      echo "=== ConfigMap as Environment Variables ==="
      echo "APP_ENV: \$APP_ENV"
      echo "DB_HOST: \$DB_HOST"
      echo "DB_PORT: \$DB_PORT"
      echo "LOG_LEVEL: \$LOG_LEVEL"
      sleep 3600
    env:
    - name: APP_ENV
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_ENV
    - name: DB_HOST
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: DB_HOST
    - name: DB_PORT
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: DB_PORT
    - name: LOG_LEVEL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: LOG_LEVEL
EOF

# 4. 환경변수 주입 확인
kubectl logs configmap-env-demo

# 5. ConfigMap을 파일로 마운트하는 Pod
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: configmap-file-demo
spec:
  containers:
  - name: app
    image: busybox:1.35
    command:
    - /bin/sh
    - -c
    - |
      echo "=== ConfigMap as Mounted Files ==="
      echo "Contents of /etc/config/application.properties:"
      cat /etc/config/application.properties
      sleep 3600
    volumeMounts:
    - name: config
      mountPath: /etc/config

  volumes:
  - name: config
    configMap:
      name: app-config
      items:
      - key: application.properties
        path: application.properties
EOF

# 6. 파일 마운트 확인
kubectl logs configmap-file-demo
```

### 기대 결과

```bash
$ kubectl logs configmap-env-demo
=== ConfigMap as Environment Variables ===
APP_ENV: production
DB_HOST: postgres.default.svc.cluster.local
DB_PORT: 5432
LOG_LEVEL: info

$ kubectl logs configmap-file-demo
=== ConfigMap as Mounted Files ===
Contents of /etc/config/application.properties:
app.name=My App
app.version=1.0
app.debug=false
```

### ✅ 배운 것

- ConfigMap = 평문 설정 저장소
- 환경변수로 주입 또는 파일로 마운트
- Pod 재시작 시에만 ConfigMap 업데이트 적용
- 민감 정보는 Secret 사용

---

## 🚀 실습 B: Secret (민감 정보)

### 개념

- 암호, API Key, 인증서 저장
- Base64 인코딩 (암호화 아님)
- `kubectl get secret` 기본값으로 값 안 보임

### 완전한 배포 YAML

```yaml
# 1. Secret 생성 (3가지 방식)
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  # Base64 인코딩된 값
  DB_PASSWORD: cGFzc3dvcmQxMjM=           # "password123"
  API_KEY: c2VjcmV0LWtleS1hYmMxMjM=       # "secret-key-abc123"

---
# 2. Pod이 Secret 사용 (환경변수)
apiVersion: v1
kind: Pod
metadata:
  name: secret-demo
spec:
  containers:
  - name: app
    image: busybox:1.35
    command:
    - /bin/sh
    - -c
    - |
      echo "=== Secret as Environment Variables ==="
      echo "DB_PASSWORD: $DB_PASSWORD"
      echo "API_KEY: $API_KEY"
      echo "Secret으로부터 안전하게 주입됨!"
      sleep 3600
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: DB_PASSWORD
    - name: API_KEY
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: API_KEY
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
```

### 실습 시작

```bash
# 1. Secret 생성 (kubectl create로 쉽게)
kubectl create secret generic app-secret \
  --from-literal=DB_PASSWORD=password123 \
  --from-literal=API_KEY=secret-key-abc123

# 2. Secret 확인 (값은 안 보임)
kubectl get secret app-secret
kubectl describe secret app-secret

# 3. Secret 값 확인 (Base64 디코딩)
kubectl get secret app-secret -o json | jq '.data.DB_PASSWORD' | xargs echo -n | base64 -d

# 4. Secret을 환경변수로 주입
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: secret-demo
spec:
  containers:
  - name: app
    image: busybox:1.35
    command:
    - /bin/sh
    - -c
    - |
      echo "=== Secret as Environment Variables ==="
      echo "DB_PASSWORD: \$DB_PASSWORD"
      echo "API_KEY: \$API_KEY"
      sleep 3600
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: DB_PASSWORD
    - name: API_KEY
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: API_KEY
EOF

# 5. Pod 로그 확인
kubectl logs secret-demo
```

### 기대 결과

```bash
$ kubectl get secret app-secret
NAME         TYPE     DATA   AGE
app-secret   Opaque   2      2m

$ kubectl logs secret-demo
=== Secret as Environment Variables ===
DB_PASSWORD: password123
API_KEY: secret-key-abc123
```

### ✅ 배운 것

- Secret = 민감 정보 저장소
- Base64 인코딩 (프로덕션에서는 ETCD 암호화 권장)
- Pod 삭제 후에도 Secret은 유지
- 환경변수로 주입하거나 파일로 마운트

---

## 🚀 실습 C: ConfigMap + Secret 함께 사용

### 예제

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: full-config-demo
spec:
  containers:
  - name: app
    image: busybox:1.35
    command:
    - /bin/sh
    - -c
    - |
      echo "=== Full Configuration Demo ==="
      echo "Environment: $APP_ENV"
      echo "Database: $DB_HOST:$DB_PORT"
      echo "User: $DB_USER"
      echo "Password (first 3 chars): ${DB_PASSWORD:0:3}***"
      echo ""
      echo "Config file:"
      cat /etc/config/app.conf
      sleep 3600
    env:
    # ConfigMap에서
    - name: APP_ENV
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_ENV
    - name: DB_HOST
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: DB_HOST
    - name: DB_PORT
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: DB_PORT
    # Secret에서
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: DB_USER
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: DB_PASSWORD
    volumeMounts:
    - name: config-file
      mountPath: /etc/config

  volumes:
  - name: config-file
    configMap:
      name: app-config
```

### 실습

```bash
# 1. 필요한 Secret 항목 추가
kubectl create secret generic app-secret \
  --from-literal=DB_USER=admin \
  --from-literal=DB_PASSWORD=SecurePass123! \
  --dry-run=client -o yaml | kubectl apply -f -

# 2. Pod 배포
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: full-config-demo
spec:
  containers:
  - name: app
    image: busybox:1.35
    command:
    - /bin/sh
    - -c
    - |
      echo "=== Full Configuration Demo ==="
      echo "Environment: \$APP_ENV"
      echo "Database: \$DB_HOST:\$DB_PORT"
      echo "User: \$DB_USER"
      echo "Password (masked): \${DB_PASSWORD:0:3}***"
      sleep 3600
    env:
    - name: APP_ENV
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_ENV
    - name: DB_HOST
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: DB_HOST
    - name: DB_PORT
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: DB_PORT
    - name: DB_USER
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: DB_USER
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: app-secret
          key: DB_PASSWORD
    volumeMounts:
    - name: config-file
      mountPath: /etc/config

  volumes:
  - name: config-file
    configMap:
      name: app-config
EOF

# 3. 결과 확인
kubectl logs full-config-demo
```

### 기대 결과

```bash
$ kubectl logs full-config-demo
=== Full Configuration Demo ===
Environment: production
Database: postgres.default.svc.cluster.local:5432
User: admin
Password (masked): Sec***
```

### ✅ 배운 것

- ConfigMap과 Secret을 함께 사용해 완전한 설정 관리
- 개발/테스트/프로덕션 환경별로 다른 ConfigMap/Secret 생성 가능
- Pod 설정과 비즈니스 로직 분리

---

## 🧹 정리

```bash
# 모든 Pod 삭제
kubectl delete pod configmap-env-demo configmap-file-demo secret-demo full-config-demo

# ConfigMap/Secret은 필요시 유지 또는 삭제
kubectl delete configmap app-config
kubectl delete secret app-secret
```

---

## 📚 다음 단계

- [docs/06-configuration.md](../../docs/06-configuration.md) - 고급 설정 관리
- [labs/](../) - 다른 실습 목록
