# 기여 가이드 🤝

이 프로젝트에 기여해주셔서 감사합니다!

## 기여 방법

### 1️⃣ 이슈 보고

오류나 개선 사항을 발견했나요?

- **명확한 제목** 작성
- **상세한 설명** 포함 (상황, 예상 결과, 실제 결과)
- **스크린샷** (있으면) 첨부

### 2️⃣ Pull Request

```bash
# 1. Fork 및 Clone
git clone https://github.com/YOUR-USERNAME/kubernetes-glossary.git
cd kubernetes-glossary

# 2. Feature 브랜치 생성
git checkout -b feature/my-improvement

# 3. 수정 후 커밋
git add .
git commit -m "Describe your changes"

# 4. Push
git push origin feature/my-improvement

# 5. GitHub에서 Pull Request 생성
```

## 작성 가이드

### Markdown 스타일

- 제목: H2 (`##`) 이상 사용
- 코드 블록: ` ``` ` 사용
- 목차: 문서 상단에 추가
- 링크: 상대 경로 사용

### 예제 코드

```yaml
# YAML 들여쓰기: 2칸
spec:
  containers:
  - name: app
    image: myapp:1.0
```

```bash
# 명령어는 주석으로 설명
kubectl get pods -n default  # 기본 namespace의 Pod 조회
```

## 콘텐츠 템플릿

```markdown
# 제목 📌

> **목표**: 간단한 설명
> 
> **학습 시간**: X시간 | **난이도**: ⭐ 수준

---

## 📋 목차
1. [개념 1](#개념-1)
2. [개념 2](#개념-2)

---

## 개념 1

**개념 설명** - 2-3줄로 간결하게

### 예제

\`\`\`yaml
# YAML 코드
\`\`\`

### 언제 사용?

- Use case 1
- Use case 2

---

**마지막 업데이트: 2026-07-22**
```

## PR 체크리스트

- [ ] 제목이 명확한가?
- [ ] 오류가 없나? (맞춤법, 링크)
- [ ] 예제 코드가 작동하나?
- [ ] 목차가 정확한가?
- [ ] 관련 링크를 추가했나?

## 개선 예시

| 개선 전 | 개선 후 |
|--------|--------|
| Pod이란 뭐임? | Pod은 Kubernetes의 최소 배포 단위입니다 |
| 걍 쓰면 됨 | 예제 코드 + 설명 포함 |
| 검사를 그냥 함 | Liveness Probe, Readiness Probe 구분 설명 |

---

**Happy Contributing! 🚀**
