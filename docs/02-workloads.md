# 2️⃣ Kubernetes 워크로드 관리 🚀

> **목표**: Deployment, StatefulSet, Job 등 워크로드 관리 방식 이해
> 
> **학습 시간**: 2-3시간 | **난이도**: ⭐⭐ 초급

---

## 📋 목차
1. [Deployment - 가장 중요!](#deployment--가장-중요)
2. [StatefulSet - 상태 저장](#statefulset--상태-저장)
3. [DaemonSet - 노드당 1개](#daemonset--노드당-1개)
4. [Job & CronJob - 배치](#job--cronjob--배치)
5. [Pod 라이프사이클](#pod-라이프사이클)
6. [실전 패턴](#실전-패턴)

---

## Deployment - 가장 중요!

**Deployment**는 **상태 비저장 애플리케이션 관리**의 표준입니다.

```
Your desired state → Deployment → ReplicaSet → Pod → Container
(몇 개 Pod 원함?)   (관리)       (복제)       (실행)
```

### 예제: 기본 Nginx Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3                    # 3개 Pod 유지
  selector:
    matchLabels:
      app: nginx                 # "app=nginx" Label 선택
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
        resources:
          limits:
            memory: "128Mi"
            cpu: "500m"
          requests:
            memory: "64Mi"
            cpu: "250m"
```

**배포 및 확인:**
```bash
kubectl apply -f nginx-deployment.yaml

# Deployment 확인
kubectl get deployments
kubectl describe deployment nginx-deployment

# ReplicaSet 확인 (자동 생성)
kubectl get replicasets

# Pod 확인
kubectl get pods -l app=nginx
```

### 🔄 Rolling Update (무중단 배포)

**목표**: 새 버전으로 교체하되 서비스 중단 없음

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1          # 추가로 만들 수 있는 Pod (3 + 1 = 4)
      maxUnavailable: 0    # 동시에 내릴 수 있는 Pod (안 내림)
```

**실행:**
```bash
# 새 이미지로 업데이트
kubectl set image deployment/nginx-deployment nginx=nginx:1.16.0

# 진행 상황 확인
kubectl rollout status deployment/nginx-deployment

# 롤백
kubectl rollout undo deployment/nginx-deployment

# 롤아웃 히스토리
kubectl rollout history deployment/nginx-deployment
```

---

## StatefulSet - 상태 저장

**StatefulSet**은 **상태 저장 애플리케이션** 관리합니다 (데이터베이스, 캐시).

**Deployment와 차이:**
| 항목 | Deployment | StatefulSet |
|------|-----------|------------|
| Pod 이름 | 무작위 (nginx-abc123) | 순서대로 (mysql-0, mysql-1, mysql-2) |
| 네트워크 ID | 변함 | 고정 (mysql-0.mysql.default.svc.cluster.local) |
| 저장소 | 공유 | 각각의 PVC |
| 시작 순서 | 병렬 | 순차적 |
| 스케일 다운 | 병렬 | 역순 (3 → 2 → 1) |

### 예제: MySQL StatefulSet

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql           # 헤드리스 Service 필요
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        env:
        - name: MYSQL_ROOT_PASSWORD
          value: "password"
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:        # 각 Pod마다 PVC 생성
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "standard"
      resources:
        requests:
          storage: 1Gi
```

**헤드리스 Service (필수):**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  clusterIP: None              # 헤드리스 (VIP 없음)
  selector:
    app: mysql
  ports:
  - port: 3306
```

---

## DaemonSet - 노드당 1개

**DaemonSet**은 **모든 노드에서 1개씩 Pod 실행**합니다.

```
Node 1 → DaemonSet Pod (로그 수집기)
Node 2 → DaemonSet Pod (로그 수집기)
Node 3 → DaemonSet Pod (로그 수집기)
```

**Use Case:**
- 로그 수집 (Fluentd, Logstash)
- 모니터링 (Prometheus Node Exporter)
- 네트워크 플러그인

### 예제: Fluentd 로그 수집

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
      - name: fluentd
        image: fluent/fluentd:v1.14
        volumeMounts:
        - name: log
          mountPath: /var/log
      volumes:
      - name: log
        hostPath:
          path: /var/log
```

---

## Job & CronJob - 배치

### Job - 일회성 작업

**Job**은 **작업 완료 후 Pod 종료**합니다.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi-calculation
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl:5.34
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4              # 실패 시 4번 재시도
```

**확인:**
```bash
kubectl get jobs
kubectl logs job/pi-calculation
```

### CronJob - 주기적 작업

**CronJob**은 **스케줄에 따라 Job 실행**합니다.

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: backup-db
spec:
  schedule: "0 2 * * *"        # 매일 02:00 실행 (cron 형식)
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: mysql:5.7
            command:
            - /bin/sh
            - -c
            - mysqldump -u root -p$MYSQL_ROOT_PASSWORD > /backup/db.sql
            env:
            - name: MYSQL_ROOT_PASSWORD
              valueFrom:
                secretKeyRef:
                  name: mysql-secret
                  key: password
          restartPolicy: OnFailure
```

---

## Pod 라이프사이클

### HealthCheck - 세 가지 Probe

#### 1️⃣ Liveness Probe - "살아있나?"

Pod가 정지 상태에 빠졌다면 재시작합니다.

```yaml
spec:
  containers:
  - name: app
    image: myapp:1.0
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 10    # 시작 후 10초 대기
      periodSeconds: 10          # 10초마다 체크
      failureThreshold: 3        # 3회 연속 실패 시 재시작
```

#### 2️⃣ Readiness Probe - "트래픽 받을 준비 됐나?"

Pod가 트래픽을 받을 준비가 안 됐으면 Service에서 제외합니다.

```yaml
spec:
  containers:
  - name: app
    image: myapp:1.0
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
```

#### 3️⃣ Startup Probe - "시작됐나?" (느린 앱용)

느린 시작 앱이 준비될 때까지 대기합니다.

```yaml
spec:
  containers:
  - name: app
    image: slow-app:1.0
    startupProbe:
      httpGet:
        path: /startup
        port: 8080
      failureThreshold: 30      # 최대 300초 대기 (30 * 10)
      periodSeconds: 10
```

---

## 실전 패턴

### 1. 블루-그린 배포

두 개의 Deployment를 번갈아가며 사용:

```yaml
# v1 (Green - 이전 버전)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-green
spec:
  replicas: 3
  selector:
    matchLabels:
      app: app
      version: green

---
# v2 (Blue - 새 버전)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-blue
spec:
  replicas: 3
  selector:
    matchLabels:
      app: app
      version: blue

---
# Service가 Green 선택 → Blue로 전환
apiVersion: v1
kind: Service
metadata:
  name: app
spec:
  selector:
    app: app
    version: green    # "blue"로 변경하여 트래픽 전환
  ports:
  - port: 80
    targetPort: 8080
```

### 2. 카나리 배포

새 버전을 작은 비율로 시작 후 점진적 확대:

```bash
# 기존 Deployment: 90 replicas
kubectl scale deployment/app-v1 --replicas=90

# 새 Deployment: 10 replicas (10%)
kubectl apply -f app-v2.yaml
# (replicas: 10)

# 모니터링 후 비율 조정
kubectl scale deployment/app-v2 --replicas=50
kubectl scale deployment/app-v1 --replicas=50

# 완전히 전환
kubectl scale deployment/app-v2 --replicas=100
kubectl scale deployment/app-v1 --replicas=0
```

### 3. Pod 배치 제어

#### Affinity - 특정 노드에 배치

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
```

#### Anti-Affinity - Pod 분산

```yaml
spec:
  affinity:
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: app
              operator: In
              values:
              - myapp
          topologyKey: kubernetes.io/hostname
```

---

## 🎯 핵심 요점

| 워크로드 | 용도 | Pod 이름 | 네트워크 |
|---------|------|---------|---------|
| **Deployment** | 웹앱, API (상태 비저장) | 무작위 | 변함 |
| **StatefulSet** | DB, 캐시 (상태 저장) | 순서대로 | 고정 |
| **DaemonSet** | 로그 수집, 모니터링 | 무작위 | 변함 |
| **Job** | 배치 작업 | 무작위 | 변함 |
| **CronJob** | 주기적 작업 | 무작위 | 변함 |

---

## 📌 다음 단계

1. **Deployment 숙달** → 실제 프로젝트에 적용
2. **Probe 직접 실습** → [labs/03-liveness-readiness/](../labs/03-liveness-readiness/README.md) 로 이동
3. **Service 이해** → [네트워킹](03-networking.md)
4. **저장소 연결** → [저장소](04-storage.md)

---

**마지막 업데이트: 2026-07-22**
