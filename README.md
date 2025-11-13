# Formology

**문서중심 개발 방법론** (The Science of Forms and Flows)

```
    ●●  눈손이 (Nunsoni)
   ✋✋

"말하지 말고, 써라!"
```

## 개요

Formology는 비즈니스 프로세스 디지털화를 위한 실용적 방법론입니다. 추상적 개념(Entity, Aggregate, Domain Model)부터 시작하는 대신, **현업이 실제로 사용하는 문서**(의뢰서, 보고서, 기록지)부터 시작하여 자연스럽게 데이터 모델을 도출합니다.

### 핵심 원칙

- **구체성 우선**: 추상화 대신 구체적인 문서 양식부터
- **현업 중심**: 개발자 용어가 아닌 현장 언어 사용
- **자연스러운 도출**: 문서 양식에서 데이터 모델이 자동으로 보임
- **Bottom-Up**: 상향식 접근으로 실용성 확보

## 빠른 시작

### 3분 이해

```
기존 방식:
"Entity 모델링부터..." → 현업: "???" → 2시간 회의 → 결정 없음

Formology 방식:
"어떤 문서 쓰세요?" → 현업: "검사의뢰서, 보고서요" → 30분 완료
```

### 누구를 위한 것인가?

**주 독자**
- 제조/의료/물류 현장 관리자
- 업무 프로세스 개선 담당자
- IT 지식 불필요 - 업무 이해만 있으면 충분

**부 독자**
- 비즈니스 분석가
- 시스템 설계자/개발자
- 프로젝트 관리자

## 주요 특징

### ✅ 현업 친화적

```
Entity? Aggregate? Value Object? ❌
의뢰서? 보고서? 확인서? ✅

현업이 매일 쓰는 용어 그대로 사용
```

### ✅ 빠른 검증

```
워크숍 2시간 → 문서 목록 도출 → 양식 스케치
현업: "맞아요!" / "이것도 필요해요!"
→ 즉시 피드백, 즉시 수정
```

### ✅ 자동 데이터 모델링

```
문서 양식을 보면:
- [선택] 입력 → FK 관계
- 체크박스 여러개 → 1:N 관계
- "참조:" 섹션 → 참조 관계

ERD가 자연스럽게 도출됨
```

## 문서 구조

상세한 가이드는 [문서 인덱스](docs/index.md)를 참조하세요.

### 📚 주요 문서

| 섹션 | 설명 | 링크 |
|------|------|------|
| **소개** | 방법론 배경과 핵심 원칙 | [Introduction](docs/01-introduction/) |
| **방법론** | 문서 분류, 네이밍, 데이터 모델링 | [Methodology](docs/02-methodology/) |
| **워크숍** | 실전 워크숍 진행 가이드 | [Workshop](docs/03-workshop/) |
| **구현** | 시스템 설계 및 개발 가이드 | [Implementation](docs/04-implementation/) |
| **사례** | 실제 프로젝트 성공 사례 | [Case Studies](docs/05-case-studies/) |
| **참고** | FAQ, 템플릿, 체크리스트 | [Reference](docs/06-reference/) |

## 실제 성과

### 플라스틱 사출 공장 MES
- **기간**: 3개월
- **효과**: 작업지시 누락 0건, 불량 추적 100%, 재고 정확도 99%
- **현업 평가**: "우리 하던 거 그대로!"

### 반도체 장비 CMMS
- **기간**: 6개월
- **효과**: 장비 가동률 85%→92%, 수리시간 4.5h→3.2h, 부품재고 30% 감소

[더 많은 사례 보기](docs/05-case-studies/)

## 시작하기

### Step 1: 개념 이해
- [왜 문서중심인가?](docs/01-introduction/why-formica.md)
- [핵심 원칙](docs/01-introduction/principles.md)

### Step 2: 방법론 학습
- [문서의 종류와 본질](docs/02-methodology/document-types.md)
- [문서 이름 짓기](docs/02-methodology/naming-conventions.md)
- [데이터 모델 도출](docs/02-methodology/data-modeling.md)

### Step 3: 워크숍 실습
- [워크숍 준비](docs/03-workshop/preparation.md)
- [진행 가이드](docs/03-workshop/facilitation-guide.md)

### Step 4: 시스템 구현
- [데이터베이스 설계](docs/04-implementation/database-design.md)
- [API 설계](docs/04-implementation/api-design.md)
- [화면 설계](docs/04-implementation/ui-design.md)

## vs 기존 방법론

### vs 온톨로지/DDD
```
온톨로지/DDD:
- 추상적 개념 정의부터
- Entity, Aggregate, Bounded Context
- 개발자만 이해
- 회의만 길어짐

Formology:
- 구체적 문서 나열부터
- 의뢰서, 보고서, 확인서
- 누구나 이해
- 바로 실행
```

[상세 비교 보기](docs/02-methodology/ontology-comparison.md)

### vs 데이터 모델링
```
전통 데이터 모델링:
- ERD 그리기부터
- 정규화 (3NF, BCNF...)
- 현업 이해 어려움
- 3주 소요

Formology:
- 문서 양식 그리기
- ERD 자연 도출
- 현업 직접 참여
- 1일 소요
```

## 기여하기

Formology는 오픈 방법론입니다. 다음과 같은 기여를 환영합니다:

- 실제 적용 사례 공유
- 문서 개선 및 번역
- 템플릿 및 도구 개발
- 피드백 및 제안

## 라이선스

이 프로젝트는 CC BY 4.0 라이선스 하에 배포됩니다. [LICENSE](LICENSE) 파일을 참조하세요.

## 연락처

- 이슈 및 제안: [GitHub Issues](https://github.com/your-repo/formology/issues)
- 토론 및 질문: [GitHub Discussions](https://github.com/your-repo/formology/discussions)

---

**Formology** - "Workflow First, Ontology Follows" | "워크플로가 먼저, 존재론은 따라온다"
