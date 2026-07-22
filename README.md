# Kubernetes 용어 및 개념 사전 📖

> 쿠버네티스 마스터를 위한 **필수 용어 및 개념 정리** 가이드

Kubernetes를 배우면서 마주치는 수백 개의 용어와 개념을 체계적으로 정리한 리소스입니다. 각 개념은 정의, 사용 사례, 실제 예제, 그리고 다른 개념과의 관계를 포함합니다.

## 🎯 목표

- ✅ K8s 용어를 **카테고리별로 체계화**
- ✅ 각 개념의 **"왜"와 "언제"** 설명
- ✅ **실제 예제 YAML** 제공
- ✅ **아키텍처 다이어그램**으로 관계 시각화
- ✅ 초급자부터 고급 사용자까지 **단계적 학습**

## 📚 학습 구조

| 단계 | 주제 | 소요 시간 | 난이도 |
|------|------|----------|--------|
| 1️⃣ | **핵심 개념** - Pod, Container, Node | 1-2시간 | ⭐ 기초 |
| 2️⃣ | **워크로드** - Deployment, StatefulSet, Job | 2-3시간 | ⭐⭐ 초급 |
| 3️⃣ | **네트워킹** - Service, Ingress, NetworkPolicy | 2-3시간 | ⭐⭐⭐ 중급 |
| 4️⃣ | **저장소** - PV, PVC, StorageClass | 1.5-2시간 | ⭐⭐ 초급 |
| 5️⃣ | **보안 & RBAC** - 권한, 인증, 정책 | 2-3시간 | ⭐⭐⭐ 중급 |
| 6️⃣ | **설정 관리** - ConfigMap, Secret | 1-1.5시간 | ⭐ 기초 |
| 7️⃣ | **관찰성** - Logs, Metrics, Events | 2-2.5시간 | ⭐⭐⭐ 중급 |
| 8️⃣ | **고급** - Operators, CRD, Webhooks | 3-4시간 | ⭐⭐⭐⭐ 고급 |
| 9️⃣ | **트러블슈팅** - 디버깅, 일반 문제 | 2-3시간 | ⭐⭐⭐ 중급 |

**총 학습 시간: ~18-24시간**

## 🚀 빠른 시작

### 1️⃣ 전체 용어 색인 보기
```bash
# 모든 용어를 한눈에 보려면:
open GLOSSARY.md
```

### 2️⃣ 주제별 심화 학습
```bash
# 핵심 개념부터 시작
open docs/01-core-concepts.md

# 관심 있는 주제로 이동
open docs/03-networking.md      # Networking
open docs/05-security-rbac.md   # RBAC & 보안
```

### 3️⃣ 실제 예제 YAML 확인
```bash
# 예제를 직접 클러스터에 적용:
kubectl apply -f examples/deployment-example.yaml
kubectl apply -f examples/service-example.yaml
```

### 4️⃣ 아키텍처 다이어그램으로 이해
```bash
# 시각적으로 개념 학습:
open diagrams/pod-lifecycle.md
open diagrams/service-discovery.md
```

## 📖 문서 구조

### 핵심 개념 (Level 1: 기초)
- [01-core-concepts.md](docs/01-core-concepts.md)
  - Pod, Container, Node, Cluster
  - Namespace, Label, Selector
  - API Resources, kubectl 기초

### 워크로드 (Level 2: 초급)
- [02-workloads.md](docs/02-workloads.md)
  - Deployment (가장 중요!)
  - StatefulSet (상태 관리)
  - DaemonSet, Job, CronJob
  - Pod 라이프사이클

### 네트워킹 (Level 3: 중급)
- [03-networking.md](docs/03-networking.md)
  - Service & Service Types (ClusterIP, NodePort, LoadBalancer)
  - Endpoint & Service Discovery
  - Ingress & Ingress Controller
  - NetworkPolicy & 보안
  - DNS & CoreDNS

### 저장소 (Level 2: 초급)
- [04-storage.md](docs/04-storage.md)
  - Volume & Volume Types
  - PersistentVolume (PV)
  - PersistentVolumeClaim (PVC)
  - StorageClass & Dynamic Provisioning
  - StatefulSet과 스토리지 통합

### 보안 & RBAC (Level 3: 중급)
- [05-security-rbac.md](docs/05-security-rbac.md)
  - RBAC (Role, RoleBinding, ClusterRole)
  - ServiceAccount & Pod Identity
  - SecurityContext & Pod Security
  - NetworkPolicy
  - Secret & 암호화

### 설정 관리 (Level 1: 기초)
- [06-configuration.md](docs/06-configuration.md)
  - ConfigMap (설정 파일)
  - Secret (민감 정보)
  - Environment Variables
  - 패턴 & 모범 사례

### 관찰성 (Level 3: 중급)
- [07-observability.md](docs/07-observability.md)
  - Logs (Container, Pod, Node)
  - Metrics & Metrics Server
  - Events & Monitoring
  - kubectl 디버깅 명령어

### 고급 주제 (Level 4: 고급)
- [08-advanced.md](docs/08-advanced.md)
  - Custom Resource Definition (CRD)
  - Operator Pattern
  - Webhook (Validating, Mutating)
  - Custom Controller
  - 플러그인 아키텍처

### 트러블슈팅 (Level 3: 중급)
- [09-troubleshooting.md](docs/09-troubleshooting.md)
  - Pod Pending 문제
  - CrashLoopBackOff 디버깅
  - Network 문제 해결
  - Resource 관련 문제
  - 일반적인 실수와 해결책

## 📇 전체 용어 색인

[GLOSSARY.md](GLOSSARY.md)에서 모든 용어를 빠르게 찾을 수 있습니다.

주요 용어 (가나다순):
- **Admission Controller** - Pod/Request 검증
- **Affinity** - Pod 배치 정책
- **API Server** - Kubernetes의 제어 평면
- **ConfigMap** - 설정 관리
- **Container** - 애플리케이션 실행 단위
- **CRD** - Custom Resource Definition
- **DaemonSet** - 노드당 1개 Pod
- **Deployment** - 상태 비저장 애플리케이션 관리 (가장 중요!)
- **Endpoint** - 서비스에 바인딩된 IP:Port 목록
- **Ingress** - HTTP(S) 라우팅
- **Job** - 배치 작업 실행
- **Label & Selector** - 리소스 식별 및 선택
- **Namespace** - 멀티테넌트 격리
- **NetworkPolicy** - 네트워크 접근 정책
- **Node** - 컨테이너 실행 머신
- **PersistentVolume** - 저장소 추상화
- **Pod** - 쿠버네티스 최소 단위
- **RBAC** - Role-Based Access Control
- **Secret** - 민감 정보 관리
- **Service** - Pod 네트워크 접근
- **ServiceAccount** - Pod 인증 ID
- **StatefulSet** - 상태 저장 애플리케이션 관리
- **StorageClass** - 스토리지 동적 프로비저닝
- **VolumeMount** - 컨테이너에 볼륨 연결

## 💡 학습 팁

1. **순서대로 학습하세요** - 기초 → 중급 → 고급
2. **예제를 직접 실행해보세요** - 이론만으로는 부족함
3. **kubectl 명령어를 자주 사용하세요** - CLI 익숙해지기
4. **다이어그램을 그려보세요** - 개념 간 관계 이해
5. **실패를 두려워하지 마세요** - 클러스터를 망쳐도 다시 만들 수 있음

## 🛠️ 필수 도구

이 가이드를 최대한 활용하려면:

```bash
# kubectl 설치
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"

# Kubernetes 클러스터 (3가지 옵션)
# 1. Docker Desktop with Kubernetes enabled
# 2. Minikube (로컬)
# 3. AKS/EKS/GKE (클라우드)
```

## 📚 추천 외부 자료

- [Kubernetes 공식 문서](https://kubernetes.io/docs/)
- [Kubernetes 설계 철학](https://kubernetes.io/docs/concepts/overview/working-with-objects/kubernetes-objects/)
- [CKAD 시험 가이드](https://training.linuxfoundation.org/certification/certified-kubernetes-application-developer-ckad/)

## 🤝 기여하기

오류를 발견했거나 개선 사항이 있으신가요?

- [CONTRIBUTING.md](CONTRIBUTING.md)를 읽어주세요
- 이슈 또는 PR 제출
- 예제 코드 추가 환영

## 📄 라이선스

이 프로젝트는 MIT 라이선스 하에 있습니다. [LICENSE](LICENSE) 참조.

---

## 🎓 다음 단계

1. **이 가이드 완료 후:**
   - [AKS 네트워킹 가이드](https://github.com/SeoanSon/aks-networking-hands-on) 학습
   - Helm 차트 작성 연습
   - Kustomize 또는 ArgoCD 배우기

2. **실무 적용:**
   - 회사 워크로드를 K8s에 마이그레이션
   - GitOps 파이프라인 구축
   - 모니터링/로깅 시스템 통합

3. **심화 학습:**
   - Service Mesh (Istio, Cilium)
   - Kubernetes 컨트롤러 개발
   - 멀티클러스터 관리

---

**Happy Learning! 🚀**

마지막 업데이트: 2026-07-22
