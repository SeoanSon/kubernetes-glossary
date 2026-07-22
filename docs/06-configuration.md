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

[보안 & RBAC](05-security-rbac.md) 문서 참고!

---

**마지막 업데이트: 2026-07-22**
