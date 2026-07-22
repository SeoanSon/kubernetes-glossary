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
```

---

## Metrics - 리소스 사용량

```bash
# Pod 리소스 사용량
kubectl top pods

# Node 리소스 사용량
kubectl top nodes

# 전체 namespace
kubectl top pods -A
```

---

## Events - 이벤트 추적

```bash
# 모든 이벤트
kubectl get events

# Pod 관련 이벤트
kubectl describe pod pod-name

# 시간순 정렬
kubectl get events --sort-by='.lastTimestamp'
```

---

**마지막 업데이트: 2026-07-22**
