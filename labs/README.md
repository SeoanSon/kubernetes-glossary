# 🧪 Labs - 직접 해보는 실습 모음

> 개념 문서([docs/](../docs/))에서 학습한 내용을 직접 클러스터에서 실습합니다.

## 📋 실습 목록

| # | 실습 | 관련 개념 | 난이도 | 소요 시간 |
|---|------|---------|--------|---------|
| 03 | [Liveness & Readiness Probe](03-liveness-readiness/README.md) | 워크로드 / HealthCheck | ⭐⭐ | 30-45분 |

> 더 많은 실습이 추가될 예정입니다.

## 🚀 시작 전 준비

```bash
# kubectl 연결 확인
kubectl cluster-info

# 현재 namespace 확인
kubectl config view --minify | grep namespace
```

## 📚 개념 먼저 학습 후 실습 권장

각 실습은 연결된 개념 문서와 함께 학습하세요:

```
docs/02-workloads.md → labs/03-liveness-readiness/
```
