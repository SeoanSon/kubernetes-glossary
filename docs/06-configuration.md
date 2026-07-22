# 6️⃣ Kubernetes 설정 관리 ⚙️

> **목표**: ConfigMap과 Secret으로 설정 관리 이해
> 
> **학습 시간**: 1-1.5시간 | **난이도**: ⭐ 기초

---

## ConfigMap - 설정 파일

**ConfigMap**은 **구성 정보**(설정, 환경 변수)를 저장합니다.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  app.properties: |
    db.host=localhost
    db.port=5432
  APP_ENV: production
```

**Pod에서 사용:**
```yaml
spec:
  containers:
  - name: app
    image: myapp:1.0
    envFrom:
    - configMapRef:
        name: app-config           # 모든 key를 환경 변수로
    volumeMounts:
    - name: config
      mountPath: /etc/config
  volumes:
  - name: config
    configMap:
      name: app-config            # 파일로 마운트
```

---

## Secret - 민감 정보

**Secret**은 **민감한 정보**(암호, API Key)를 저장합니다.

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  username: YWRtaW4=           # Base64 인코딩
  password: cGFzc3dvcmQxMjM=   # "password123"
```

**Pod에서 사용:**
```yaml
spec:
  containers:
  - name: app
    image: myapp:1.0
    env:
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
```

### 주의
- Base64 인코딩만 제공 (암호화 아님)
- 프로덕션에서는 ETCD 암호화 활성화 필수
- 더 강력한 보안: [보안 & RBAC](05-security-rbac.md) 참고

---

## 🧪 직접 실습해보기

**클러스터에서 실제로 배포하고 테스트해보세요:**

→ [**실습 06: Kubernetes 설정 관리**](../labs/06-configuration/README.md)
- ✅ ConfigMap 생성 및 환경변수 주입
- ✅ ConfigMap을 파일로 마운트
- ✅ Secret 생성 및 민감 정보 주입
- ✅ ConfigMap + Secret 함께 사용
- ✅ 실제 kubectl 명령어와 결과

---

## 📚 다음 단계

1. **실습하기**: [Lab 06 - Kubernetes 설정 관리](../labs/06-configuration/README.md)
2. **보안**: [Doc 05 - 보안 & RBAC](05-security-rbac.md)
3. **전체 Labs**: [Labs 홈](../labs/README.md)

---

**마지막 업데이트: 2026-07-22**
