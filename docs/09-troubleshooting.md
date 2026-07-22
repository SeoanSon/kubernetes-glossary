# 9️⃣ Kubernetes 트러블슈팅 🔧

> **목표**: 일반적인 문제 해결 방법 학습
> 
> **학습 시간**: 2-3시간 | **난이도**: ⭐⭐⭐ 중급

---

## 📋 흔한 문제와 해결책

### 1️⃣ Pod Pending

**증상:** Pod이 생성되지 않고 `Pending` 상태

**원인:**
- 노드 리소스 부족
- 스케줄러 문제

**해결:**
```bash
# 상세 확인
kubectl describe pod pod-name

# 노드 리소스 확인
kubectl top nodes

# 이벤트 확인
kubectl get events --sort-by='.lastTimestamp'
```

### 2️⃣ CrashLoopBackOff

**증상:** Pod이 계속 재시작됨

**원인:**
- 애플리케이션 크래시
- 잘못된 이미지
- 설정 오류

**해결:**
```bash
# 이전 Pod 로그 확인
kubectl logs pod-name --previous

# 최근 로그
kubectl logs pod-name

# 상세 확인
kubectl describe pod pod-name
```

### 3️⃣ ImagePullBackOff

**증상:** 이미지를 다운로드하지 못함

**원인:**
- 이미지 이름 오류
- 레지스트리 인증 문제
- 네트워크 문제

**해결:**
```bash
kubectl describe pod pod-name  # 에러 메시지 확인
```

### 4️⃣ Pod 간 통신 불가

**증상:** Pod이 서로 통신할 수 없음

**원인:**
- NetworkPolicy 차단
- DNS 문제
- 방화벽

**해결:**
```bash
# DNS 테스트
kubectl exec pod-name -- nslookup service-name

# Service 확인
kubectl get svc
kubectl get endpoints

# 연결 테스트
kubectl exec pod-name -- curl http://other-service:8080
```

---

## 디버깅 명령어

```bash
# Pod 접속
kubectl exec -it pod-name -- bash

# 임시 디버그 Pod
kubectl run -it --image=curlimages/curl debug -- /bin/sh

# 포트 포워딩
kubectl port-forward pod/pod-name 8080:8080

# 로그 실시간 보기
kubectl logs pod-name -f

# 모든 리소스 상태
kubectl get all -A
```

---

**마지막 업데이트: 2026-07-22**
