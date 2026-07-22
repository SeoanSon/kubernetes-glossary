# 4️⃣ Kubernetes 저장소 💾

> **목표**: Volume, PV, PVC로 데이터 저장 이해
> 
> **학습 시간**: 1.5-2시간 | **난이도**: ⭐⭐ 초급

---

## 📋 목차
1. [Volume - 기본](#volume--기본)
2. [PersistentVolume & PVC](#persistentvolume--pvc)
3. [StorageClass - 동적 프로비저닝](#storageclass--동적-프로비저닝)

---

## Volume - 기본

**Volume**은 **Pod 내 컨테이너 간 데이터 공유** 또는 **임시 저장**입니다.

### emptyDir - 임시 저장소

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    volumeMounts:
    - name: cache
      mountPath: /tmp/cache      # 마운트 경로
  volumes:
  - name: cache
    emptyDir: {}                  # Pod 시작 시 생성, 종료 시 삭제
```

**특징:**
- Pod 내 컨테이너 간 공유
- Pod 삭제 시 데이터도 삭제

### hostPath - 노드 디렉토리

```yaml
volumes:
- name: logs
  hostPath:
    path: /var/logs               # 호스트 경로
    type: Directory
```

**주의:** 노드별로 다르므로 DaemonSet에만 사용!

---

## PersistentVolume & PVC

**문제:** Pod이 삭제되면 데이터도 삭제됨!

**해결:** PersistentVolume (관리자 생성) + PersistentVolumeClaim (사용자 요청)

### 2-단계 프로세스

```
1️⃣ 관리자가 PersistentVolume 생성
   (저장소 할당)

2️⃣ 사용자가 PersistentVolumeClaim 생성
   (저장소 요청)

3️⃣ Kubernetes가 PVC ↔ PV 바인딩
   (자동 할당)
```

### 예제

**Step 1: 관리자가 PV 생성**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-1
spec:
  capacity:
    storage: 1Gi                  # 1GB
  accessModes:
    - ReadWriteOnce               # 1개 노드에서만
  storageClassName: standard
  hostPath:
    path: /data/pv-1
```

**Step 2: 사용자가 PVC 생성**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: db-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: standard
  resources:
    requests:
      storage: 1Gi
```

**Step 3: Deployment에서 PVC 사용**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  template:
    spec:
      containers:
      - name: mysql
        image: mysql:5.7
        volumeMounts:
        - name: db-data
          mountPath: /var/lib/mysql
      volumes:
      - name: db-data
        persistentVolumeClaim:
          claimName: db-pvc
```

---

## 🧪 직접 실습해보기

**클러스터에서 실제로 배포하고 테스트해보세요:**

→ [**실습 05: Kubernetes 저장소**](../labs/05-storage/README.md)
- ✅ emptyDir Volume으로 임시 저장소 사용
- ✅ hostPath Volume으로 노드 디렉토리 마운트
- ✅ PersistentVolume & PersistentVolumeClaim 생성
- ✅ 데이터 영속성 확인
- ✅ 실제 kubectl 명령어와 결과

---

## StorageClass - 동적 프로비저닝

**문제:** 관리자가 매번 PV를 만들어야 함!

**해결:** **StorageClass**로 자동 프로비저닝!

```
PVC 생성 → StorageClass → 자동으로 PV 생성
                        (클라우드 API 호출)
```

### 예제

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs    # AWS EBS
parameters:
  type: gp2                            # GP2 볼륨
  iops: "3000"
```

**사용:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  storageClassName: fast-ssd          # StorageClass 지정
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 100Gi                  # 자동으로 PV 생성!
```

---

## 🎯 핵심 요점

| 타입 | 용도 | 수명 |
|------|------|------|
| **emptyDir** | 임시 공유 저장소 | Pod와 함께 삭제 |
| **hostPath** | 노드 디렉토리 접근 | 노드 의존적 |
| **PV + PVC** | 영구 저장소 | Pod 삭제 후에도 유지 |
| **StorageClass** | 자동 프로비저닝 | 동적 할당 |

---

## 📚 다음 단계

1. **실습하기**: [Lab 05 - Kubernetes 저장소](../labs/05-storage/README.md)
2. **보안**: [Doc 05 - 보안 & RBAC](05-security-rbac.md)
3. **전체 Labs**: [Labs 홈](../labs/README.md)

---

**마지막 업데이트: 2026-07-22**
