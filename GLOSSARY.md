# Kubernetes 전체 용어 색인 📇

쿠버네티스에서 자주 사용되는 모든 용어를 **가나다순**으로 정렬했습니다.
각 용어를 클릭하면 상세 설명 문서로 이동합니다.

---

## A
- **Admission Controller** - Pod/Request 생성 시 검증 및 변조
  - [고급 주제에서 자세히 보기](docs/08-advanced.md)
- **Affinity** - Pod 배치 정책 (NodeAffinity, PodAffinity)
  - [워크로드에서 자세히 보기](docs/02-workloads.md)
- **API Server** - Kubernetes의 중앙 제어 평면 (etcd와 통신)
  - [핵심 개념에서 자세히 보기](docs/01-core-concepts.md)
- **API Version** - 리소스의 API 버전 (v1, apps/v1 등)
  - [핵심 개념에서 자세히 보기](docs/01-core-concepts.md)

## B
- **Binding** - Service가 Endpoint를 선택
  - [네트워킹에서 자세히 보기](docs/03-networking.md)

## C
- **ClusterIP** - Service 타입: 내부 통신만 (기본값)
  - [네트워킹에서 자세히 보기](docs/03-networking.md)
- **ConfigMap** - 환경 변수 및 설정 파일 저장
  - [설정 관리에서 자세히 보기](docs/06-configuration.md)
- **Container** - 애플리케이션 실행 단위 (Docker, containerd 등)
  - [핵심 개념에서 자세히 보기](docs/01-core-concepts.md)
- **Container Registry** - 컨테이너 이미지 저장소 (Docker Hub, ECR, ACR)
  - [핵심 개념에서 자세히 보기](docs/01-core-concepts.md)
- **Control Plane** - Kubernetes 제어 시스템 (API Server, Scheduler, Controller Manager)
  - [핵심 개념에서 자세히 보기](docs/01-core-concepts.md)
- **Controller** - 현재 상태를 원하는 상태로 조정하는 루프
  - [핵심 개념에서 자세히 보기](docs/01-core-concepts.md)
- **CRD (Custom Resource Definition)** - 커스텀 리소스 정의
  - [고급 주제에서 자세히 보기](docs/08-advanced.md)

## D
- **DaemonSet** - 모든 노드에서 1개 Pod 실행
  - [워크로드에서 자세히 보기](docs/02-workloads.md)
- **Deployment** - 상태 비저장 애플리케이션 관리 ⭐ 가장 중요!
  - [워크로드에서 자세히 보기](docs/02-workloads.md)
- **DNS** - Pod 간 이름 해석 (service.namespace.svc.cluster.local)
  - [네트워킹에서 자세히 보기](docs/03-networking.md)

## E
- **Endpoint** - Service가 트래픽을 보낼 수 있는 주소 목록
  - [네트워킹에서 자세히 보기](docs/03-networking.md)
- **etcd** - Kubernetes 상태 저장 (key-value store)
  - [핵심 개념에서 자세히 보기](docs/01-core-concepts.md)

## F
- **Field Selector** - 필드로 리소스 필터링 (metadata.name, status.phase)
  - [핵심 개념에서 자세히 보기](docs/01-core-concepts.md)

## G
- **Garbage Collection** - 미아 Pod/리소스 자동 삭제
  - [핵심 개념에서 자세히 보기](docs/01-core-concepts.md)

## H
- **HealthCheck** - Pod 상태 확인 (Liveness, Readiness, Startup Probe)
  - [워크로드에서 자세히 보기](docs/02-workloads.md)

## I
- **Ingress** - HTTP(S) 외부 라우팅
  - [네트워킹에서 자세히 보기](docs/03-networking.md)
- **Ingress Controller** - Ingress 규칙 구현 (Nginx, Traefik 등)
  - [네트워킹에서 자세히 보기](docs/03-networking.md)
- **Init Container** - Pod 시작 전 초기화 작업
  - [워크로드에서 자세히 보기](docs/02-workloads.md)

## J
- **Job** - 배치 작업 실행 (Pod 완료 후 종료)
  - [워크로드에서 자세히 보기](docs/02-workloads.md)

## K
- **kubectl** - Kubernetes CLI 도구
  - [핵심 개념에서 자세히 보기](docs/01-core-concepts.md)
- **kubelet** - 각 노드에서 실행되는 에이전트
  - [핵심 개념에서 자세히 보기](docs/01-core-concepts.md)

## L
- **Label** - 리소스 식별용 키-값 쌍
  - [핵심 개념에서 자세히 보기](docs/01-core-concepts.md)
- **Liveness Probe** - Pod가 살아있는지 확인
  - [워크로드에서 자세히 보기](docs/02-workloads.md)
- **LoadBalancer** - Service 타입: 클라우드 LB 사용
  - [네트워킹에서 자세히 보기](docs/03-networking.md)

## M
- **Manifest** - Kubernetes 리소스 정의 (YAML/JSON)
  - [핵심 개념에서 자세히 보기](docs/01-core-concepts.md)
- **Metrics Server** - Pod/Node 리소스 사용량 수집
  - [관찰성에서 자세히 보기](docs/07-observability.md)

## N
- **Namespace** - 가상 클러스터 격리 (멀티테넌트)
  - [핵심 개념에서 자세히 보기](docs/01-core-concepts.md)
- **NetworkPolicy** - 네트워크 접근 제어 정책
  - [네트워킹에서 자세히 보기](docs/03-networking.md)
- **Node** - 컨테이너 실행 머신
  - [핵심 개념에서 자세히 보기](docs/01-core-concepts.md)
- **NodePort** - Service 타입: 노드의 포트로 노출
  - [네트워킹에서 자세히 보기](docs/03-networking.md)

## O
- **Owner Reference** - Pod/리소스의 소유자 (Deployment, StatefulSet 등)
  - [핵심 개념에서 자세히 보기](docs/01-core-concepts.md)

## P
- **PersistentVolume (PV)** - 저장소 리소스 (관리자 생성)
  - [저장소에서 자세히 보기](docs/04-storage.md)
- **PersistentVolumeClaim (PVC)** - 저장소 요청 (사용자가 생성)
  - [저장소에서 자세히 보기](docs/04-storage.md)
- **Pod** - Kubernetes 최소 배포 단위 ⭐ 핵심!
  - [핵심 개념에서 자세히 보기](docs/01-core-concepts.md)
- **Pod Security Policy (PSP)** - Pod 보안 정책 (deprecated)
  - [보안 & RBAC에서 자세히 보기](docs/05-security-rbac.md)

## Q
- **QoS Class** - Pod 우선순위 (Guaranteed, Burstable, BestEffort)
  - [워크로드에서 자세히 보기](docs/02-workloads.md)

## R
- **RBAC (Role-Based Access Control)** - 권한 제어
  - [보안 & RBAC에서 자세히 보기](docs/05-security-rbac.md)
- **Readiness Probe** - Pod가 트래픽 받을 준비가 되었는지
  - [워크로드에서 자세히 보기](docs/02-workloads.md)
- **ReplicaSet** - 정해진 수의 Pod 복제본 유지
  - [워크로드에서 자세히 보기](docs/02-workloads.md)
- **Resource Quota** - 네임스페이스별 리소스 사용 제한
  - [설정 관리에서 자세히 보기](docs/06-configuration.md)

## S
- **Scheduler** - Pod를 노드에 배치하는 제어 평면
  - [핵심 개념에서 자세히 보기](docs/01-core-concepts.md)
- **Secret** - 민감 정보 저장 (암호, 토큰, 인증서)
  - [설정 관리에서 자세히 보기](docs/06-configuration.md)
- **SecurityContext** - Pod/Container 보안 설정
  - [보안 & RBAC에서 자세히 보기](docs/05-security-rbac.md)
- **Selector** - Label로 리소스 선택
  - [핵심 개념에서 자세히 보기](docs/01-core-concepts.md)
- **Service** - Pod 네트워크 접근 추상화 ⭐ 핵심!
  - [네트워킹에서 자세히 보기](docs/03-networking.md)
- **ServiceAccount** - Pod 인증 ID
  - [보안 & RBAC에서 자세히 보기](docs/05-security-rbac.md)
- **StatefulSet** - 상태 저장 애플리케이션 관리 (persistent identity)
  - [워크로드에서 자세히 보기](docs/02-workloads.md)
- **StorageClass** - 스토리지 동적 프로비저닝
  - [저장소에서 자세히 보기](docs/04-storage.md)

## T
- **Taint & Toleration** - 노드 격리 및 Pod 배치 제어
  - [워크로드에서 자세히 보기](docs/02-workloads.md)
- **Topology** - Pod 배치 지역 설정
  - [워크로드에서 자세히 보기](docs/02-workloads.md)

## V
- **Volume** - Pod 간 데이터 공유/저장
  - [저장소에서 자세히 보기](docs/04-storage.md)
- **VolumeMount** - 컨테이너에 볼륨 연결
  - [저장소에서 자세히 보기](docs/04-storage.md)

## W
- **Webhook** - 요청 사전/사후 처리 (Validating, Mutating)
  - [고급 주제에서 자세히 보기](docs/08-advanced.md)

## X
- (없음)

## Y
- (없음)

## Z
- (없음)

---

## 🔗 빠른 링크

- [핵심 개념](docs/01-core-concepts.md) - Pod, Node, Label, Namespace
- [워크로드](docs/02-workloads.md) - Deployment, StatefulSet, Job
- [네트워킹](docs/03-networking.md) - Service, Ingress, NetworkPolicy
- [저장소](docs/04-storage.md) - Volume, PV, PVC, StorageClass
- [보안 & RBAC](docs/05-security-rbac.md) - RBAC, Secret, SecurityContext
- [설정 관리](docs/06-configuration.md) - ConfigMap, Secret
- [관찰성](docs/07-observability.md) - Logs, Metrics, Events
- [고급 주제](docs/08-advanced.md) - CRD, Operator, Webhook
- [트러블슈팅](docs/09-troubleshooting.md) - 디버깅, 문제 해결

---

**마지막 업데이트: 2026-07-22**
