# 7️⃣ Kubernetes 관찰성 📊

> **목표**: Logs, Metrics, Events로 관찰성 확보
> 
> **학습 시간**: 2-2.5시간 | **난이도**: ⭐⭐⭐ 중급

---

## 📋 목차
1. [Logs - 로그 확인](#logs--로그-확인)
2. [Metrics - 리소스 사용량](#metrics--리소스-사용량)
3. [Events - 이벤트 추적](#events--이벤트-추적)

---

## Logs - 로그 확인

**기본 로그 명령어:**
```bash
# 컨테이너 로그
kubectl logs pod-name

# 이전 Pod (재시작 후)
kubectl logs pod-name --previous

# 실시간 로그
kubectl logs pod-name -f

# 여러 Pod
kubectl logs -l app=nginx

# 특정 컨테이너
kubectl logs pod-name -c container-name

# 시간 범위
kubectl logs pod-name --since=1h

# 마지막 100줄
kubectl logs pod-name --tail=100
```

### 🧪 실제 로그 확인 실습

```bash
# 1. 로그 생성하는 Pod 배포
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: log-demo
spec:
  containers:
  - name: app
    image: busybox:1.35
    command:
    - /bin/sh
    - -c
    - |
      counter=0
      while true; do
        counter=\$((counter+1))
        echo "[\$(date '+%H:%M:%S')] Request #\$counter processed"
        sleep 2
      done
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"
EOF

# 2. 실시간 로그 확인
kubectl logs -f log-demo

# 3. 마지막 10줄만 확인
kubectl logs log-demo --tail=10

# 4. Pod 삭제 후 이전 로그 확인 (재시작된 경우)
kubectl delete pod log-demo
```

**기대 결과:**
```
[10:15:23] Request #1 processed
[10:15:25] Request #2 processed
[10:15:27] Request #3 processed
```

---

## Metrics - 리소스 사용량

**주의:** `metrics-server`가 설치되어 있어야 함

```bash
# Pod 리소스 사용량
kubectl top pods

# Node 리소스 사용량
kubectl top nodes

# 전체 namespace
kubectl top pods -A

# Namespace별 사용량
kubectl top pods -n kube-system
```

### 🧪 리소스 사용량 실습

```bash
# 1. CPU 사용량이 많은 Pod 배포
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: cpu-heavy
spec:
  containers:
  - name: app
    image: busybox:1.35
    command:
    - /bin/sh
    - -c
    - |
      while true; do
        # CPU 사용량 증가
        awk 'BEGIN {for(i=0; i<100000000; i++);}'
        echo "CPU busy..."
      done
    resources:
      requests:
        memory: "32Mi"
        cpu: "100m"
      limits:
        memory: "64Mi"
        cpu: "200m"
EOF

# 2. 리소스 사용량 모니터링
kubectl top pod cpu-heavy

# 3. Node 리소스 확인
kubectl top nodes
```

**기대 결과:**
```bash
$ kubectl top pod cpu-heavy
NAME       CPU(cores)   MEMORY(Mi)
cpu-heavy  150m         5Mi
```

---

## Events - 이벤트 추적

**이벤트는 Pod 생성, 재시작, 에러 등의 모든 활동을 기록합니다.**

```bash
# 모든 이벤트
kubectl get events

# Pod 관련 이벤트
kubectl describe pod pod-name

# 시간순 정렬
kubectl get events --sort-by='.lastTimestamp'

# 최근 이벤트
kubectl get events --sort-by='.metadata.creationTimestamp' | tail -20
```

### 🧪 이벤트 추적 실습

```bash
# 1. 리소스 부족으로 인한 이벤트 생성
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: event-demo
spec:
  containers:
  - name: app
    image: nginx:1.25
    resources:
      requests:
        memory: "1000Gi"  # 거의 불가능한 리소스 요청
        cpu: "100"
EOF

# 2. Pod 상태 확인 (Pending 상태)
kubectl get pod event-demo

# 3. 상세 이벤트 확인
kubectl describe pod event-demo

# 4. 전체 이벤트 확인
kubectl get events --sort-by='.metadata.creationTimestamp'

# 5. Pod 삭제
kubectl delete pod event-demo
```

**기대 결과:**
```bash
$ kubectl describe pod event-demo
...
Events:
  Type     Reason            Age   Message
  ----     ------            ---   -------
  Warning  FailedScheduling  10s   0/3 nodes are available: ...insufficient memory...
```

---

## 🎯 관찰성 베스트 프랙티스

| 항목 | 실천 방법 |
|------|---------|
| **로그** | 구조화된 로그 (JSON) + 로그 레벨 설정 |
| **메트릭** | 리소스 요청/제한 명확히 설정 |
| **이벤트** | Pod describe로 이벤트 주시 |
| **모니터링** | Prometheus + Grafana (선택) |

---

## 📚 다음 단계

- [실습 09: 트러블슈팅](../labs/09-troubleshooting/README.md) (곧 추가)
- [전체 Labs](../labs/README.md)

---

**마지막 업데이트: 2026-07-22**
