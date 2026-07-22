# 8️⃣ Kubernetes 고급 주제 🚀

> **목표**: CRD, Operator, Webhook의 확장 방식 이해
> 
> **학습 시간**: 3-4시간 | **난이도**: ⭐⭐⭐⭐ 고급

---

## Custom Resource Definition (CRD)

**CRD**로 새로운 리소스 타입을 정의합니다.

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.example.com
spec:
  group: example.com
  names:
    kind: Database
    plural: databases
  scope: Namespaced
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        properties:
          spec:
            properties:
              engine:
                type: string      # postgres, mysql 등
              version:
                type: string
```

**사용:**
```yaml
apiVersion: example.com/v1
kind: Database
metadata:
  name: my-db
spec:
  engine: postgres
  version: "13"
```

---

## Operator Pattern

**Operator**는 **애플리케이션 운영을 자동화하는 Custom Controller**입니다.

```
CRD (Database) → Operator (Custom Controller) → Pod, PVC 등 생성
                 (자동 설정, 백업, 업그레이드)
```

---

## Webhook

**Webhook**으로 요청을 가로채 검증/변조합니다.

### Validating Webhook - 검증
```
Pod 생성 요청 → Webhook → 검증 → 수락/거절
              (정책 확인)
```

### Mutating Webhook - 변조
```
Pod 생성 요청 → Webhook → 자동 수정 → Pod 생성
              (기본값 추가 등)
```

---

**마지막 업데이트: 2026-07-22**
