# 🧪 Lab 05: Kubernetes 저장소 실습

> **목표**: Volume, PV, PVC를 직접 배포해서 데이터 영속성 이해
>
> **소요 시간**: 1-1.5시간 | **난이도**: ⭐⭐ 초급

---

## 📖 개념

### 저장소가 필요한 이유

```
문제: Pod 삭제 → 데이터도 사라짐
     여러 Pod이 파일 공유 불가
     
해결: Volume이 Pod 라이프사이클과 독립적
     데이터 영속성 보장
```

---

## 🚀 실습 A: emptyDir (임시 저장소)

### 개념

- Pod이 시작될 때 생성, 삭제될 때 제거
- Pod 내 컨테이너 간 데이터 공유용
- **영속성 없음** (임시용)

### 완전한 배포 YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-demo
spec:
  containers:
  # Writer 컨테이너
  - name: writer
    image: busybox:1.35
    command:
    - /bin/sh
    - -c
    - |
      while true; do
        date >> /data/log.txt
        sleep 5
      done
    volumeMounts:
    - name: shared-data
      mountPath: /data
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"

  # Reader 컨테이너
  - name: reader
    image: busybox:1.35
    command:
    - /bin/sh
    - -c
    - |
      while true; do
        echo "=== Latest logs ==="
        tail -5 /data/log.txt 2>/dev/null || echo "Waiting for logs..."
        sleep 5
      done
    volumeMounts:
    - name: shared-data
      mountPath: /data
    resources:
      requests:
        memory: "32Mi"
        cpu: "50m"
      limits:
        memory: "64Mi"
        cpu: "100m"

  # Volume 정의
  volumes:
  - name: shared-data
    emptyDir: {}                # Pod 시작시 생성, 종료시 삭제
```

### 실습 시작

```bash
# 1. Pod 배포
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-demo
spec:
  containers:
  - name: writer
    image: busybox:1.35
    command:
    - /bin/sh
    - -c
    - |
      while true; do
        date >> /data/log.txt
        sleep 5
      done
    volumeMounts:
    - name: shared-data
      mountPath: /data

  - name: reader
    image: busybox:1.35
    command:
    - /bin/sh
    - -c
    - |
      while true; do
        echo "=== Latest logs ==="
        tail -5 /data/log.txt 2>/dev/null || echo "Waiting..."
        sleep 5
      done
    volumeMounts:
    - name: shared-data
      mountPath: /data

  volumes:
  - name: shared-data
    emptyDir: {}
EOF

# 2. Pod 실행 확인
kubectl get pod emptydir-demo

# 3. reader 컨테이너 로그 실시간 확인
kubectl logs -f emptydir-demo -c reader

# 4. writer 컨테이너 로그 확인
kubectl logs emptydir-demo -c writer

# 5. Pod 내부 파일 확인
kubectl exec emptydir-demo -c reader -- cat /data/log.txt
```

### 기대 결과

```bash
$ kubectl logs -f emptydir-demo -c reader
=== Latest logs ===
Waiting...
=== Latest logs ===
Tue Jul 22 10:15:32 UTC 2026
=== Latest logs ===
Tue Jul 22 10:15:32 UTC 2026
Tue Jul 22 10:15:37 UTC 2026
=== Latest logs ===
Tue Jul 22 10:15:32 UTC 2026
Tue Jul 22 10:15:37 UTC 2026
Tue Jul 22 10:15:42 UTC 2026

$ kubectl exec emptydir-demo -c reader -- cat /data/log.txt
Tue Jul 22 10:15:32 UTC 2026
Tue Jul 22 10:15:37 UTC 2026
Tue Jul 22 10:15:42 UTC 2026
Tue Jul 22 10:15:47 UTC 2026
```

### ✅ 배운 것

- 두 컨테이너가 같은 Volume으로 데이터 공유
- Pod 삭제 시 데이터도 함께 삭제
- 캐시, 임시 파일에 사용

---

## 🚀 실습 B: hostPath (노드 디렉토리 마운트)

### 개념

- 호스트 노드의 디렉토리를 Pod에 마운트
- **주의**: Pod가 다른 노드로 스케줄되면 접근 불가
- DaemonSet에만 권장

### 완전한 배포 YAML

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-demo
spec:
  containers:
  - name: app
    image: busybox:1.35
    command:
    - /bin/sh
    - -c
    - |
      echo "Host node info:" > /host-data/info.txt
      hostname >> /host-data/info.txt
      df -h >> /host-data/info.txt
      cat /host-data/info.txt
      sleep 3600
    volumeMounts:
    - name: host-logs
      mountPath: /host-data

  volumes:
  - name: host-logs
    hostPath:
      path: /tmp/k8s-demo          # 호스트 경로
      type: DirectoryOrCreate      # 없으면 생성
```

### 실습 시작

```bash
# 1. Pod 배포
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-demo
spec:
  containers:
  - name: app
    image: busybox:1.35
    command:
    - /bin/sh
    - -c
    - |
      echo "Host node info:" > /host-data/info.txt
      hostname >> /host-data/info.txt
      df -h >> /host-data/info.txt
      cat /host-data/info.txt
      sleep 3600
    volumeMounts:
    - name: host-logs
      mountPath: /host-data

  volumes:
  - name: host-logs
    hostPath:
      path: /tmp/k8s-demo
      type: DirectoryOrCreate
EOF

# 2. Pod 로그 확인
kubectl logs hostpath-demo

# 3. 호스트에서 파일 확인 (노드 접근 필요)
kubectl debug hostpath-demo -it --image=busybox -- cat /host-data/info.txt
```

### 기대 결과

```bash
$ kubectl logs hostpath-demo
Host node info:
aks-agentpool-12345678-vmss000000
Filesystem     Size  Used Avail Use% Mounted on
/dev/sda1       28G  5.2G   23G  19% /
```

### ✅ 배운 것

- hostPath는 특정 노드의 디렉토리 접근
- Pod 재시작해도 데이터 유지 (같은 노드)
- 다른 노드로 스케줄되면 다른 파일 접근

---

## 🚀 실습 C: PersistentVolume & PersistentVolumeClaim

### 개념

```
관리자 (PV 생성)
    ↓
Kubernetes (자동 바인딩)
    ↓
사용자 (PVC로 요청)
```

### 완전한 배포 YAML

```yaml
# 1. PersistentVolume (관리자가 생성)
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-demo
spec:
  capacity:
    storage: 1Gi                    # 1GB 용량
  accessModes:
    - ReadWriteOnce                 # 1개 노드에서만 읽쓰기
  storageClassName: manual
  hostPath:
    path: /tmp/pv-data

---
# 2. PersistentVolumeClaim (사용자가 요청)
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-demo
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  resources:
    requests:
      storage: 500Mi               # 500MB 요청

---
# 3. Pod이 PVC 사용
apiVersion: v1
kind: Pod
metadata:
  name: pvc-demo-pod
spec:
  containers:
  - name: app
    image: busybox:1.35
    command:
    - /bin/sh
    - -c
    - |
      echo "Writing to persistent storage..." > /data/test.txt
      ls -la /data/
      cat /data/test.txt
      sleep 3600
    volumeMounts:
    - name: persistent-storage
      mountPath: /data

  volumes:
  - name: persistent-storage
    persistentVolumeClaim:
      claimName: pvc-demo
```

### 실습 시작

```bash
# 1. PV 생성
kubectl apply -f - <<EOF
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-demo
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  hostPath:
    path: /tmp/pv-data
EOF

# 2. PVC 생성
kubectl apply -f - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-demo
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  resources:
    requests:
      storage: 500Mi
EOF

# 3. PV & PVC 상태 확인
kubectl get pv
kubectl get pvc

# 4. Pod 생성
kubectl apply -f - <<EOF
apiVersion: v1
kind: Pod
metadata:
  name: pvc-demo-pod
spec:
  containers:
  - name: app
    image: busybox:1.35
    command:
    - /bin/sh
    - -c
    - |
      echo "Writing to persistent storage..." > /data/test.txt
      ls -la /data/
      cat /data/test.txt
      sleep 3600
    volumeMounts:
    - name: persistent-storage
      mountPath: /data

  volumes:
  - name: persistent-storage
    persistentVolumeClaim:
      claimName: pvc-demo
EOF

# 5. Pod 로그 확인
kubectl logs pvc-demo-pod

# 6. PVC 상태 확인 (BOUND 상태)
kubectl describe pvc pvc-demo
```

### 기대 결과

```bash
$ kubectl get pv,pvc
NAME               CAPACITY   ACCESS MODES   STATUS   CLAIM            
pv/pv-demo         1Gi        RWO            Bound    default/pvc-demo

NAME              STATUS   VOLUME   CAPACITY   ACCESS MODES
pvc/pvc-demo      Bound    pv-demo  1Gi        RWO

$ kubectl logs pvc-demo-pod
Writing to persistent storage...
total 12
drwxr-xr-x   2 root  root        4096 Jul 22 10:20
-rw-r--r--   1 root  root          30 Jul 22 10:20 test.txt
Writing to persistent storage...
```

### ✅ 배운 것

- PV = 관리자가 제공하는 저장소
- PVC = 사용자가 요청하는 저장소
- Status: Bound = PV와 PVC 연결됨
- Pod 삭제 후 재생성해도 데이터 유지

---

## 🧹 정리

```bash
# 모든 리소스 삭제
kubectl delete pod emptydir-demo hostpath-demo pvc-demo-pod
kubectl delete pvc pvc-demo
kubectl delete pv pv-demo
```

---

## 📚 다음 단계

- [docs/04-storage.md](../../docs/04-storage.md) - StorageClass & 고급 개념
- [labs/](../) - 다른 실습 목록
