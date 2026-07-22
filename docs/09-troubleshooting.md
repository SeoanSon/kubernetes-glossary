# 9️⃣ Kubernetes 트러블슈팅 🔧

> **목표**: 일반적인 문제 해결 방법 학습
> 
> **학습 시간**: 2-3시간 | **난이도**: ⭐⭐⭐ 중급

---

## 📋 흔한 문제와 해결책

### 1️⃣ Pod Pending - 리소스 부족

**증상:** Pod이 생성되지 않고 `Pending` 상태

**원인:**
- 노드 리소스 부족
- 리소스 요청이 과도함

**🧪 직접 재현해보기:**

```bash
# 1. Pending Pod 배포 (거대한 리소스 요청)
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: pending-demo
spec:
  containers:
  - name: app
    image: nginx:1.25
    resources:
      requests:
        memory: "1000Gi"      # 1000GB (실제 클러스터에 없음)
        cpu: "100"            # 100 cores
EOF

# 2. Pod 상태 확인 (Pending)
kubectl get pod pending-demo

# 3. 상세 원인 확인
kubectl describe pod pending-demo

# 4. 이벤트 확인
kubectl get events --sort-by='.lastTimestamp'
```

**기대 결과:**
```bash
$ kubectl get pod pending-demo
NAME            READY   STATUS    RESTARTS   AGE
pending-demo    0/1     Pending   0          2m

$ kubectl describe pod pending-demo
...
Events:
  Type     Reason            Age   Message
  ----     ------            ---   -------
  Warning  FailedScheduling  2m    0/3 nodes are available: ...insufficient memory...
```

**해결책:**
```bash
# 1. Pod 정의 수정 (리소스 요청 감소)
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: pending-demo-fixed
spec:
  containers:
  - name: app
    image: nginx:1.25
    resources:
      requests:
        memory: "64Mi"      # 64MB로 감소
        cpu: "50m"          # 50m로 감소
      limits:
        memory: "128Mi"
        cpu: "100m"
EOF

# 2. 정상 실행 확인
kubectl get pod pending-demo-fixed
```

**정리:**
```bash
kubectl delete pod pending-demo pending-demo-fixed
```

---

### 2️⃣ CrashLoopBackOff - 애플리케이션 크래시

**증상:** Pod이 계속 재시작됨 (CrashLoopBackOff)

**원인:**
- 애플리케이션 에러
- 초기화 실패
- 의존성 부족

**🧪 직접 재현해보기:**

```bash
# 1. 크래시하는 Pod 배포
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: crash-demo
spec:
  containers:
  - name: app
    image: busybox:1.35
    command:
    - /bin/sh
    - -c
    - |
      echo "Starting app..."
      sleep 2
      echo "Error occurred!"
      exit 1  # 크래시!
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
EOF

# 2. Pod 상태 확인 (CrashLoopBackOff)
kubectl get pod crash-demo

# 3. 충돌 로그 확인
kubectl logs crash-demo
kubectl logs crash-demo --previous  # 이전 실행 로그

# 4. 상세 상태 확인
kubectl describe pod crash-demo
```

**기대 결과:**
```bash
$ kubectl get pod crash-demo
NAME          READY   STATUS             RESTARTS   AGE
crash-demo    0/1     CrashLoopBackOff   5          2m

$ kubectl logs crash-demo
Starting app...
Error occurred!

$ kubectl describe pod crash-demo
...
Last State:     Terminated
  Reason:       Error
  Exit Code:    1
  Started:      ...
  Finished:     ...
Restart Count:  5
```

**해결책:**
```bash
# 1. 원인 분석 (로그에서 확인)
# 2. 애플리케이션 수정
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: crash-demo-fixed
spec:
  containers:
  - name: app
    image: busybox:1.35
    command:
    - /bin/sh
    - -c
    - |
      echo "Starting app..."
      sleep 2
      echo "App running successfully!"
      sleep 3600  # 실행 유지
EOF

# 3. 정상 실행 확인
kubectl get pod crash-demo-fixed
kubectl logs crash-demo-fixed
```

**정리:**
```bash
kubectl delete pod crash-demo crash-demo-fixed
```

---

### 3️⃣ ImagePullBackOff - 이미지 문제

**증상:** Pod이 이미지를 다운로드하지 못함

**원인:**
- 이미지 이름 오류
- 레지스트리 인증 문제
- 네트워크 문제

**🧪 직접 재현해보기:**

```bash
# 1. 잘못된 이미지 배포
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: imagepull-demo
spec:
  containers:
  - name: app
    image: nginx:non-existent-version
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
EOF

# 2. Pod 상태 확인 (ImagePullBackOff)
kubectl get pod imagepull-demo

# 3. 에러 메시지 확인
kubectl describe pod imagepull-demo
```

**기대 결과:**
```bash
$ kubectl get pod imagepull-demo
NAME             READY   STATUS             RESTARTS   AGE
imagepull-demo   0/1     ImagePullBackOff   0          30s

$ kubectl describe pod imagepull-demo
...
Events:
  Type     Reason                 Age   Message
  ----     ------                 ---   -------
  Normal   Pulling                10s   Pulling image "nginx:non-existent-version"
  Warning  Failed                 5s    Failed to pull image "nginx:non-existent-version":
  Warning  BackOff                2s    Back-off pulling image "nginx:non-existent-version"
```

**해결책:**
```bash
# 1. 올바른 이미지 사용
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: imagepull-demo-fixed
spec:
  containers:
  - name: app
    image: nginx:1.25  # 올바른 태그
EOF

# 2. 정상 실행 확인
kubectl get pod imagepull-demo-fixed
```

**정리:**
```bash
kubectl delete pod imagepull-demo imagepull-demo-fixed
```

---

### 4️⃣ Pod 간 통신 불가

**증상:** Pod이 서로 통신할 수 없음

**원인:**
- NetworkPolicy 차단
- DNS 문제
- Service 설정 오류

**🧪 직접 재현해보기:**

```bash
# 1. 두 개의 Pod 배포
kubectl apply -f - <<EOF
# Server Pod
apiVersion: v1
kind: Pod
metadata:
  name: server-pod
spec:
  containers:
  - name: server
    image: nginx:1.25
    ports:
    - containerPort: 80
    resources:
      requests:
        memory: "64Mi"
        cpu: "50m"
      limits:
        memory: "128Mi"
        cpu: "100m"

---
# Client Pod
apiVersion: v1
kind: Pod
metadata:
  name: client-pod
spec:
  containers:
  - name: client
    image: curlimages/curl:8.1.0
    command:
    - /bin/sh
    - -c
    - |
      echo "Testing connectivity to server-pod..."
      # 직접 Pod IP로 접근
      curl -v http://server-pod  # 실패할 수도 있음
      # Service를 통해 접근하면 더 안정적
      sleep 3600
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
EOF

# 2. Pod 실행 확인
kubectl get pods

# 3. 클라이언트에서 테스트
kubectl exec client-pod -- curl -v http://server-pod

# 4. DNS 테스트
kubectl exec client-pod -- nslookup server-pod
```

**해결책 (Service 사용):**
```bash
# 1. Service 생성
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: server-service
spec:
  selector:
    run: server-pod
  ports:
  - port: 80
    targetPort: 80
  type: ClusterIP

---
# Deployment로 재배포 (Label과 Service Selector 일치)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: server
spec:
  replicas: 1
  selector:
    matchLabels:
      run: server-pod
  template:
    metadata:
      labels:
        run: server-pod
    spec:
      containers:
      - name: server
        image: nginx:1.25
        ports:
        - containerPort: 80
EOF

# 2. Service 확인
kubectl get svc server-service
kubectl get endpoints server-service

# 3. 클라이언트에서 Service로 접근
kubectl exec client-pod -- curl -v http://server-service
```

**정리:**
```bash
kubectl delete pod server-pod client-pod
kubectl delete deployment server
kubectl delete service server-service
```

---

## 🔧 디버깅 명령어

### Pod 접속

```bash
# 해당 Pod에 접속
kubectl exec -it pod-name -- bash

# 특정 컨테이너에 접속
kubectl exec -it pod-name -c container-name -- bash
```

### 임시 디버그 Pod

```bash
# 디버깅용 임시 Pod 실행
kubectl run -it debug --image=curlimages/curl --rm --restart=Never -- /bin/sh

# 또는 더 자세한 도구 포함
kubectl run -it debug --image=nicolaka/netshoot --rm --restart=Never -- bash
```

### 포트 포워딩

```bash
# Pod의 포트를 로컬 머신으로 포워딩
kubectl port-forward pod/pod-name 8080:8080

# Service 포트 포워딩
kubectl port-forward svc/service-name 9000:80
```

### 로그 확인

```bash
# 현재 로그
kubectl logs pod-name

# 실시간 로그
kubectl logs -f pod-name

# 이전 Pod 로그 (재시작 후)
kubectl logs pod-name --previous

# Pod 내 모든 컨테이너 로그
kubectl logs pod-name --all-containers=true
```

### 리소스 상태 확인

```bash
# Pod 상태 상세 확인
kubectl describe pod pod-name

# 이벤트 확인
kubectl get events --sort-by='.metadata.creationTimestamp'

# 노드 상태 확인
kubectl describe node node-name
```

---

## 🎯 트러블슈팅 체크리스트

| 단계 | 명령어 | 확인 항목 |
|------|--------|---------|
| 1 | `kubectl get pod` | Pod 상태 (Pending, CrashLoopBackOff 등) |
| 2 | `kubectl describe pod` | 이벤트 및 Resource 상태 |
| 3 | `kubectl logs` | 애플리케이션 에러 메시지 |
| 4 | `kubectl get events` | 클러스터 이벤트 |
| 5 | `kubectl top` | 리소스 사용량 |
| 6 | `kubectl exec` | Pod 내부 상태 확인 |

---

## 📚 다음 단계

- [실습 07: 관찰성](../docs/07-observability.md) - 로그, 메트릭, 이벤트 모니터링
- [전체 Labs](../labs/README.md)

---

**마지막 업데이트: 2026-07-22**

# 로그 실시간 보기
kubectl logs pod-name -f

# 모든 리소스 상태
kubectl get all -A
```

---

**마지막 업데이트: 2026-07-22**
