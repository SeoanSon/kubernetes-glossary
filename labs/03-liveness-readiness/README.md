# 🧪 Lab 03: Liveness & Readiness Probe 실습

> **목표**: Pod 상태 감지 메커니즘을 실제로 동작시켜보고, 실패 시 Kubernetes가 어떻게 반응하는지 이해합니다.
>
> **소요 시간**: 30-45분 | **난이도**: ⭐⭐ 초급

---

## 📖 개념

### Kubernetes는 왜 Probe가 필요한가?

프로세스가 실행 중이어도 **실제로는 동작하지 않을 수 있습니다.**

```
일반적인 문제 시나리오:
┌────────────────────────────────────────────────┐
│  Java 앱이 Running 상태 (프로세스는 살아있음)      │
│  BUT: Deadlock 발생 → 응답 불가                  │
│  BUT: DB 연결 안 됨 → 요청 처리 불가              │
│  BUT: 초기화 중 → 아직 요청 받으면 안 됨           │
└────────────────────────────────────────────────┘

Kubernetes는 이걸 자동으로 감지하고 대응합니다!
```

---

## 🔍 세 가지 Probe 비교

| Probe | 질문 | 실패 시 동작 | 목적 |
|-------|------|------------|------|
| **Liveness** | "살아있나?" | Pod **재시작** | Deadlock, 무한 루프 감지 |
| **Readiness** | "요청 받을 준비 됐나?" | Service에서 **제외** | 초기화 중, DB 연결 중 |
| **Startup** | "시작이 완료됐나?" | Liveness/Readiness **대기** | 느린 앱 (JVM, 레거시) |

### 핵심 차이점

```
Liveness 실패:
  Pod 삭제 → 새 Pod 시작
  (Pod IP가 바뀜!)

Readiness 실패:
  Service Endpoint에서 제거 (트래픽 차단)
  Pod는 그대로 살아있음
  (Pod IP 안 바뀜, 복구 대기)
```

---

## 📋 파일 구조

```
03-liveness-readiness/
├── README.md          # 이 파일
├── liveness.yaml      # Liveness Probe 실습 (고의적 실패 시뮬레이션)
└── readiness.yaml     # Readiness Probe 실습 (준비 상태 제어)
```

---

## 🚀 실습 A: Liveness Probe (자동 재시작)

### 개념

앱이 **살아있는지** 주기적으로 파일 존재 여부를 확인합니다.  
실패하면 Kubernetes가 Pod를 **재시작**합니다.

> **참고**: 실제 운영에서는 `httpGet`으로 `/health` 엔드포인트를 체크하지만, 이 실습에서는  
> HTTP 서버 없이도 바로 실행할 수 있도록 `exec` 방식(파일 존재 확인)을 사용합니다.

### 완전한 배포 YAML (`liveness.yaml`)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness-demo
  labels:
    app: liveness-demo
spec:
  containers:
  - name: liveness-demo
    image: busybox:1.35
    # 시작 후 30초까지는 /tmp/healthy 파일 유지 (정상 상태)
    # 30초 뒤 파일 삭제 → Liveness Probe 실패 → Pod 재시작 시뮬레이션
    command:
    - /bin/sh
    - -c
    - |
      touch /tmp/healthy
      echo "Started. Will fail after 30 seconds..."
      (sleep 30 && rm -f /tmp/healthy && echo ">>> FAIL: /tmp/healthy removed") &
      while true; do sleep 5; done
    livenessProbe:
      exec:                       # /tmp/healthy 파일 존재 여부로 생존 판단
        command:
        - cat
        - /tmp/healthy
      initialDelaySeconds: 5      # 시작 후 5초 대기
      periodSeconds: 5            # 5초마다 체크
      failureThreshold: 3         # 3회 연속 실패 시 재시작 (= 15초 후)
      successThreshold: 1         # 1회 성공 시 정상 판단
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
# 1. Liveness Probe Pod 배포
kubectl apply -f liveness.yaml

# 2. Pod 상태 실시간 확인 (RESTARTS 카운트 증가 확인)
kubectl get pod liveness-demo -w
```

### 고의 실패 시뮬레이션

이 컨테이너는 **시작 30초 후 `/tmp/healthy` 파일을 스스로 삭제**합니다.  
Liveness Probe가 `cat /tmp/healthy` 실패 → 3회 연속 실패 → Pod 재시작.

```bash
# Pod 이벤트 실시간 관찰 (새 터미널에서 실행)
kubectl get events --field-selector=involvedObject.name=liveness-demo --sort-by='.lastTimestamp' -w

# Probe 상세 정보 확인
kubectl describe pod liveness-demo
```

### 기대 결과

```
# 사전: kubectl get pod liveness-demo -w

NAME            READY   STATUS             RESTARTS   AGE
liveness-demo   1/1     Running            0          10s    ← 정상 시작
liveness-demo   0/1     Running            1          45s    ← 첫 번째 재시작 (RESTARTS=1)
liveness-demo   1/1     Running            1          50s    ← 재시작 후 Running
liveness-demo   0/1     Running            2          80s    ← 30초 후 또 실패, 재시작 반복
liveness-demo   0/1     CrashLoopBackOff   2          95s    ← 반복 실패 시 Kubernetes가 백오프 적용
```

> **팅**: `CrashLoopBackOff`는 오류가 아닙니다!  
> 재시작을 반복할 경우 Kubernetes가 **지수 백오프** (10s, 20s, 40s...))를 적용하는 정상 동작입니다.

### `kubectl describe` 출력 확인

```bash
kubectl describe pod liveness-demo
```

기대 출력:
```
Events:
  Type     Reason     Age    From               Message
  ----     ------     ----   ----               -------
  Normal   Scheduled  2m     default-scheduler  Successfully assigned default/liveness-demo
  Normal   Pulling    2m     kubelet            Pulling image "busybox:1.35"
  Normal   Pulled     2m     kubelet            Successfully pulled image
  Normal   Created    2m     kubelet            Container created
  Normal   Started    2m     kubelet            Container started
  Warning  Unhealthy  94s    kubelet            Liveness probe failed: cat: can't open
                                                '/tmp/healthy': No such file or directory
  Normal   Killing    94s    kubelet            Container liveness-demo failed liveness
                                                probe, will be restarted
  Normal   Pulled     90s    kubelet            Container image already present, restarting
  Normal   Created    90s    kubelet            Container created (RESTARTS=1)
  Normal   Started    90s    kubelet            Container started
  Warning  Unhealthy  55s    kubelet            Liveness probe failed: cat: can't open
                                                '/tmp/healthy': No such file or directory
  Normal   Killing    55s    kubelet            Container liveness-demo failed liveness
                                                probe, will be restarted
  Warning  BackOff    10s    kubelet            Back-off restarting failed container
                                                liveness-demo in pod liveness-demo_default
```

> **`BackOff` 이벤트**: 재시작이 반복되면 Kubernetes가 10s → 20s → 40s 식으로 대기 시간을 늘립니다.  
> `kubectl get pod liveness-demo`으로 확인하면 `CrashLoopBackOff` 상태가 보입니다.

### ✅ 배운 것

- `RESTARTS` 카운트로 재시작 이력 확인 가능
- `Events` 섹션에서 실제 에러 메시지 확인: `cat: can't open '/tmp/healthy'`
- Probe 실패 후 재시작까지 `failureThreshold × periodSeconds` = **15초** 소요
- 재시작 이후도 시작 30초 후 또 실패 → **`CrashLoopBackOff`** 진입 (정상 동작)
- Service는 다른 Pod이 있다면 중단 없이 계속 동작

---

## 🚀 실습 B: Readiness Probe (트래픽 차단)

### 개념

앱이 **요청을 받을 준비**가 됐는지 확인합니다.  
실패하면 Service Endpoint에서 **제거** (트래픽 차단), Pod는 살아있습니다.

### 완전한 배포 YAML (`readiness.yaml`)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: readiness-demo
  labels:
    app: readiness-demo
spec:
  containers:
  - name: readiness-demo
    image: busybox:1.35
    command:
    - /bin/sh
    - -c
    - |
      touch /tmp/ready
      echo "Ready. To simulate NOT READY:"
      echo "  kubectl exec readiness-demo -- rm /tmp/ready"
      while true; do sleep 10; done
    readinessProbe:
      exec:                       # /tmp/ready 파일 존재 여부로 준비 상태 판단
        command:
        - cat
        - /tmp/ready
      initialDelaySeconds: 5      # 시작 후 5초 대기
      periodSeconds: 5            # 5초마다 체크
      failureThreshold: 3         # 3회 연속 실패 시 Endpoint에서 제거
      successThreshold: 2         # 2회 연속 성공 시 Endpoint 복구
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
---
apiVersion: v1
kind: Service
metadata:
  name: readiness-demo-svc
spec:
  selector:
    app: readiness-demo
  ports:
  - port: 80
    targetPort: 80
```

### 실습 시작

```bash
# 1. Readiness Probe Pod + Service 배포
kubectl apply -f readiness.yaml

# 2. Pod & Endpoint 확인
kubectl get pods -l app=readiness-demo
kubectl get endpoints readiness-demo-svc
```

### 준비 안 된 상태 강제로 만들기

```bash
# /tmp/ready 파일 삭제 (준비 안 된 상태로 전환)
kubectl exec readiness-demo -- rm /tmp/ready

# Endpoint에서 사라지는지 확인 (약 15초 후)
kubectl get endpoints readiness-demo-svc -w
```

### 복구 시뮬레이션

```bash
# /tmp/ready 파일 재생성 (준비 완료 상태로 전환)
kubectl exec readiness-demo -- touch /tmp/ready

# Endpoint 복구 확인
kubectl get endpoints readiness-demo-svc -w
```

### 기대 결과

```
# 초기 상태 (준비됨):
NAME                  ENDPOINTS        AGE
readiness-demo-svc    10.244.1.5:80    30s

# rm /tmp/ready 후 (준비 안 됨):
NAME                  ENDPOINTS        AGE
readiness-demo-svc    <none>           45s   ← Endpoint 제거!

# touch /tmp/ready 후 (복구):
NAME                  ENDPOINTS        AGE
readiness-demo-svc    10.244.1.5:80    60s   ← Endpoint 복구!
```
```

### `kubectl describe` 출력 확인

```bash
kubectl describe pod <readiness-pod>
```

기대 출력 (준비 안 된 상태):
```
Events:
  Type     Reason             Age   From               Message
  ----     ------             ----  ----               -------
  Warning  Unhealthy          10s   kubelet            Readiness probe failed:
                                                       Get "http://10.244.1.5:8080/ready": 
                                                       connection refused

Conditions:
  Type              Status
  Initialized       True
  Ready             False         ← 준비 안 됨
  ContainersReady   False
  PodScheduled      True
```

### ✅ 배운 것

- **Ready 컬럼**: `0/1`이면 Readiness 실패
- Pod은 살아있지만 Service에서 제외됨 → **무중단 트래픽 보호**
- DB 연결 실패, 캐시 워밍업 등의 상황에 유용
- `successThreshold`로 **복구 안정성** 제어 가능

---

## 🔄 Liveness vs Readiness 비교 실험

두 Probe를 같이 설정했을 때 어떻게 동작하나요?

```yaml
containers:
- name: app
  livenessProbe:
    httpGet:
      path: /health
      port: 8080
    failureThreshold: 3
    periodSeconds: 10
  readinessProbe:
    httpGet:
      path: /ready
      port: 8080
    failureThreshold: 1
    periodSeconds: 5
```

**시나리오별 동작:**

| 상황 | Liveness | Readiness | 결과 |
|------|---------|-----------|------|
| 정상 | ✅ | ✅ | 정상 서비스 |
| DB 연결 끊김 | ✅ | ❌ | Service 제외, Pod 유지 |
| Deadlock | ❌ | ❌ | Pod 재시작 |
| 시작 중 | ✅ | ❌ | Service 제외, 초기화 대기 |

---

## 🧹 정리

```bash
kubectl delete -f liveness.yaml
kubectl delete -f readiness.yaml
```

---

## 📚 다음 단계

- [docs/02-workloads.md](../../docs/02-workloads.md) - Startup Probe와 전체 워크로드 개념
- [labs/](../) - 다른 실습 목록
- [docs/09-troubleshooting.md](../../docs/09-troubleshooting.md) - CrashLoopBackOff 트러블슈팅
