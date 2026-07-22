# Kubernetes 전체 용어 색인 📇

쿠버네티스에서 자주 사용되는 모든 용어를 **가나다순**으로 정렬했습니다.
각 용어를 클릭하면 상세 설명 문서로 이동합니다.

---

## A
- **Admission Controller** - Pod/Request 생성 시 검증 및 변조
  - [고급 주제에서 자세히 보기](docs/08-advanced.md)
- **Affinity** - Pod 배치 정책 (NodeAffinity, PodAffinity)
  - [워크로드에서 자세히 보기](docs/02-workloads.md)
- **AGIC (Application Gateway Ingress Controller)** - ⭐ AKS 고유: Azure Application Gateway 기반 Ingress
  - Azure의 관리형 Application Gateway와 통합하여 L7 라우팅 제공
  - [K8s vs AKS에서 자세히 보기](docs/10-kubernetes-vs-aks.md)
- **API Server** - Kubernetes의 중앙 제어 평면 (etcd와 통신)
  - [핵심 개념에서 자세히 보기](docs/01-core-concepts.md)
- **API Version** - 리소스의 API 버전 (v1, apps/v1 등)
  - [핵심 개념에서 자세히 보기](docs/01-core-concepts.md)
- **Azure CNI** - ⭐ AKS 고유: Azure Virtual Network 네트워크 플러그인
  - Pod가 VNet IP를 직접 사용 (가상 IP 아님), 더 나은 성능과 VNet 기능 제공
  - 대안: Kubenet, Cilium
  - [K8s vs AKS에서 자세히 보기](docs/10-kubernetes-vs-aks.md)
- **Azure Disk** - ⭐ AKS 고유: Azure Managed Disk 기반 StorageClass
  - Pod의 영구 저장소 제공 (Premium SSD, Standard HDD)
  - `managed-csi` StorageClass로 자동 프로비저닝
  - [저장소에서 자세히 보기](docs/04-storage.md)
- **Azure Files** - ⭐ AKS 고유: Azure File Share 기반 볼륨
  - 여러 Pod 간 공유 저장소 (SMB/NFS)
  - `azurefile-csi` CSI 드라이버로 지원
  - [저장소에서 자세히 보기](docs/04-storage.md)
- **Azure Policy** - ⭐ AKS 고유: Kubernetes 정책 준수 관리
  - 클러스터의 정책 위반 감시 및 거부
  - [K8s vs AKS에서 자세히 보기](docs/10-kubernetes-vs-aks.md)

## B
- **Binding** - Service가 Endpoint를 선택
  - [네트워킹에서 자세히 보기](docs/03-networking.md)

## C
- **ClusterIP** - Service 타입: 내부 통신만 (기본값)
  - [네트워킹에서 자세히 보기](docs/03-networking.md)
- **CronJob** - 주기적 배치 작업 (쳀 형식 스케줄)
  - 신택: `schedule: "0 2 * * *"` (매일 새벽 2시)
  - Job을 생성하고 완료 시 종료
  - [워크로드에서 자세히 보기](docs/02-workloads.md)
- **ConfigMap** - 환경 변수 및 설정 파일 저장
  - [설정 관리에서 자세히 보기](docs/06-configuration.md)
- **Container** - 애플리케이션 실행 단위 (Docker, containerd 등)
  - [핵심 개념에서 자세히 보기](docs/01-core-concepts.md)
- **Container Insights** - ⭐ AKS 고유: Azure의 모니터링 솔루션
  - Pod/Node 메트릭, 로그, 성능 분석을 Azure Portal에서 통합 제공
  - Prometheus/Grafana 대신 사용 가능
  - [K8s vs AKS에서 자세히 보기](docs/10-kubernetes-vs-aks.md)
- **Container Registry** - 컨테이너 이미지 저장소 (Docker Hub, ECR, ACR)
  - [핵심 개념에서 자세히 보기](docs/01-core-concepts.md)
- **Control Plane** - Kubernetes 제어 시스템 (API Server, Scheduler, Controller Manager)
  - 순수 K8s: 직접 관리 / AKS: Microsoft가 관리 ⭐ (차이점)
  - [K8s vs AKS에서 자세히 보기](docs/10-kubernetes-vs-aks.md)
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
- **Entra ID** - ⭐ AKS 고유: Microsoft Azure Active Directory 통합
  - AKS에서 pod 인증, kubectl 접근, RBAC를 Entra ID와 연동
  - 순수 K8s는 kubeconfig 기반 인증
  - [K8s vs AKS에서 자세히 보기](docs/10-kubernetes-vs-aks.md)
- **etcd** - Kubernetes 상태 저장 (key-value store)
  - [핵심 개념에서 자세히 보기](docs/01-core-concepts.md)

## F
- **Field Selector** - 필드로 리소스 필터링 (metadata.name, status.phase)
  - [핵심 개념에서 자세히 보기](docs/01-core-concepts.md)

## G
- **Garbage Collection** - 미아 Pod/리소스 자동 삭제
  - [핵심 개념에서 자세히 보기](docs/01-core-concepts.md)
- **Gatekeeper** - ⭐ AKS 고유: 기본 namespace에 설치되는 정책 엔진
  - OPA (Open Policy Agent) 기반으로 kubernetes 정책 검증
  - Pod, Service 등의 리소스 생성/수정 시 정책 위반 체크
  - Azure Policy와 연동되어 정책 위반 시 리소스 거부 또는 감시
  - 예: 특정 이미지 레지스트리만 허용, 리소스 제한 의무화
  - `gatekeeper-system` namespace에서 실행
  - [K8s vs AKS에서 자세히 보기](docs/10-kubernetes-vs-aks.md)

## H
- **AKS: AGIC (Application Gateway Ingress Controller) 옵션도 있음
  - HealthCheck** - Pod 상태 확인 (Liveness, Readiness, Startup Probe)
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
- **LimitRange** - Namespace별 기본/최대 리소스 추정 설정
  - Container에 limits/requests를 지정 안 해도 자동 적용
  - [보안 & RBAC에서 자세히 보기](docs/05-security-rbac.md)
- **Liveness Probe** - Pod가 살아있는지 확인 → 실패 시 **Pod 재시작**
  - HTTP GET, TCP, exec 세 가지 방식 지원
  - `failureThreshold × periodSeconds` 이후 재시작
  - [워크로드에서 자세히 보기](docs/02-workloads.md) | [실습](labs/03-liveness-readiness/README.md)
- **LoadBalancer** - Service 타입: 클라우드 LB 사용
  - AKS에서는 Azure Load Balancer 자동 할당
  - [네트워킹에서 자세히 보기](docs/03-networking.md)

## M
- **Manifest** - Kubernetes 리소스 정의 (YAML/JSON)
  - [핵심 개념에서 자세히 보기](docs/01-core-concepts.md)
- **Managed Disk** - ⭐ AKS 고유: Azure에서 관리하는 디스크
  - Pod의 PersistentVolume으로 사용되는 Azure Managed Disk
  - Premium (SSD) / Standard (HDD) 옵션 제공
  - [저장소에서 자세히 보기](docs/04-storage.md)
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
- **PDB (PodDisruptionBudget)** - 유지보수 중 최소 Pod 생존 보장
  - `minAvailable: 2` 또는 `maxUnavailable: 1`로 설정
  - 노드 업그레이드, 스케일 다운 시 서비스 중단 방지
  - [워크로드에서 자세히 보기](docs/02-workloads.md)
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
- **Readiness Probe** - Pod가 트래픽 받을 준비가 되었는지 → 실패 시 **Endpoint에서 제외** (Pod 유지)
  - Liveness와 크게 다르는 점: Pod를 재시작하지 않음
  - DB 연결 대기, 초기화 중 등 일시적 비활성화
  - [워크로드에서 자세히 보기](docs/02-workloads.md) | [실습](labs/03-liveness-readiness/README.md)
- **ReplicaSet** - 정해진 수의 Pod 복제본 유지
  - [워크로드에서 자세히 보기](docs/02-workloads.md)
- **Resource Quota** - 네임스페이스별 리소스 사용 제한
  - [설정 관리에서 자세히 보기](docs/06-configuration.md)

## S
- **Scheduler** - Pod를 노드에 배치하는 제어 평면
  - [핵심 개념에서 자세히 보기](docs/01-core-concepts.md)
- **Startup Probe** - 느린 앱 시작 시간 보장 Probe
  - Startup 성공 전당은 Liveness/Readiness Probe가 동작하지 않음
  - JVM, 레거시 앱에서 유용 (`failureThreshold: 30, periodSeconds: 10` 시 5분 대기)
  - [워크로드에서 자세히 보기](docs/02-workloads.md)
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
- **Zero-downtime Deployment** - 무중단 배포 패턴
  - Deployment RollingUpdate + Readiness Probe 조합으로 달성
  - `maxUnavailable: 0` 설정 필수
  - [워크로드에서 자세히 보기](docs/02-workloads.md)

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
- [K8s vs AKS](docs/10-kubernetes-vs-aks.md) - 차이점 비교 및 선택 가이드

## 🧪 실습 (Labs)

- [labs/03-liveness-readiness/](labs/03-liveness-readiness/README.md) - Liveness & Readiness Probe 직접 실습

---

**마지막 업데이트: 2026-07-22**
