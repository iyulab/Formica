# Ontology와 Formology의 관계

## 핵심 요약

**Formology ⊃ Ontology**

> Ontology는 Formology의 부분집합이며, Formology에서 자동으로 도출됩니다.

```
┌─────────────────────────────────────┐
│  Formology (형식론)                  │  ← 현실세계
│  What flows in reality?             │
│                                     │
│  Workflow → Status → Format → Form │
│                                     │
│         ↓ 자동 도출                  │
│  ┌───────────────────────────┐     │
│  │  Ontology (존재론)         │     │  ← 실재/실체
│  │  What exists?             │     │
│  │                           │     │
│  │  Object → Logic → Action  │     │
│  └───────────────────────────┘     │
└─────────────────────────────────────┘
```

---

## 레이어 구조

### Layer 1: Formology (상위 - 현실세계)

**관점**: 현실세계의 문서 흐름
**질문**: What flows? (무엇이 흐르는가?)
**요소**:
- **Workflow**: 업무의 큰 흐름
- **Status**: 워크플로에서의 상태
- **Format**: 문서의 필수 구조 (서식)
- **Form**: 서식의 시각적 구현 (양식)

**특징**:
- 관찰 가능 (현업이 직접 볼 수 있음)
- 구체적 (실제 문서, 실제 흐름)
- 시간성 (문서가 흐름)
- 현업 언어 (의뢰서, 보고서, 기록지)

### Layer 2: Ontology (하위 - 실재/실체)

**관점**: 실재하는 개념과 관계
**질문**: What exists? (무엇이 존재하는가?)
**요소**:
- **Object**: 존재하는 개체 (Entity)
- **Logic**: 개체 간 관계 (Relation)
- **Action**: 상태 변화 (Method)

**특징**:
- 추상화 (개념적 존재)
- 정적 (시간 독립적)
- 개발자 언어 (Entity, Aggregate, Value Object)
- Formology로부터 자동 도출

---

## 자동 도출 원리

### Formology → Ontology 사상 규칙

```
Formology Layer          →    Ontology Layer
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

1. Format (서식)         →    Entity (개체)
   품질검사의뢰서           →    QCRequest

2. Form Section          →    Entity Structure
   주요정보 섹션           →    Main Entity
   검사항목 섹션           →    Child Entity (1:N)

3. Field                →    Attribute
   제품명 [선택]           →    product_id (FK)
   수량 [숫자]             →    quantity (number)

4. Status               →    State
   검사대기               →    PENDING
   검사중                 →    IN_PROGRESS

5. Workflow Transition  →    State Machine
   의뢰 → 검사 → 보고     →    Transition Rules

6. Document Reference   →    Relation
   검사보고서 → 검사의뢰서  →    QCReport → QCRequest (FK)
```

### 구체적 예시

**Formology 정의**:
```
Workflow: 품질검사
  ↓
Status: 검사대기 → 검사중 → 검사완료
  ↓
Format: 검사의뢰서
  - 제품명 (제품 마스터 참조)
  - LOT번호 (텍스트)
  - 수량 (숫자)
  ↓
Form: 검사의뢰서 양식
  [주요정보]
  제품명: [▼ 선택]
  LOT: [____]
  수량: [____]
```

**자동 도출된 Ontology**:
```
Entity: QCRequest
  Attributes:
    - product_id: FK → Product (선택 → FK)
    - lot_number: String (텍스트 → String)
    - quantity: Number (숫자 → Number)

  States:
    - PENDING (검사대기)
    - IN_PROGRESS (검사중)
    - COMPLETED (검사완료)

  Transitions:
    - create() : → PENDING
    - start() : PENDING → IN_PROGRESS
    - complete() : IN_PROGRESS → COMPLETED
```

---

## 왜 Formology가 상위인가?

### 1. 인식론적 우위

**Ontology의 한계**:
```
질문: "품질이란 무엇인가?"
답변: 철학적 논쟁 → 합의 어려움
```

**Formology의 접근**:
```
질문: "어떤 문서를 쓰나요?"
답변: "검사의뢰서, 검사보고서요" → 즉시 합의
```

**결론**: 현실세계의 구체적 문서가 추상 개념보다 합의하기 쉬움

### 2. 실재론적 근거

**Ontology**: 무엇이 존재하는가? (Being)
- 개념적 존재
- 시간 독립적
- 관찰 불가 (추상)

**Formology**: 무엇이 흐르는가? (Flow)
- 물리적 존재 (문서)
- 시간 의존적
- 관찰 가능 (구체)

**결론**: 흐름(Flow)이 존재(Being)를 정의

### 3. 완전성

**정리**: Formology는 Ontology를 완전히 포함

```
∀ ontology ∃ formology representation
(모든 온톨로지는 포몰로지로 표현 가능)

But NOT:
∀ formology ∃ ontology representation
(모든 포몰로지가 온톨로지로 표현되는 것은 아님)
```

**이유**:
- Formology는 시간, 상태, 흐름 포함
- Ontology는 정적 구조만 표현
- Status, Workflow는 Ontology에 없음

---

## 실전 적용: 레이어 분리 개발

### 개발 프로세스

**Phase 1: Formology 설계 (현업 주도)**
```
Day 1: Workflow 관찰
  "품질검사 업무가 어떻게 흐르나요?"

Day 2: Status 정의
  "각 단계의 상태는 무엇인가요?"

Day 3: Format 도출
  "어떤 문서를 쓰시나요?"

Day 4: Form 스케치
  "양식이 어떻게 생겼나요?"

→ 현업이 이해하고 검증 가능
```

**Phase 2: Ontology 자동 도출 (개발자)**
```
Day 5: 사상 규칙 적용
  Format → Entity
  Field → Attribute
  Reference → Relation

Day 6: ERD 생성
  자동 도출된 Entity로 ERD 그리기

Day 7: 코드 생성
  Entity → Class/Table

→ 개발자가 자동화된 규칙으로 변환
```

### 레이어 간 독립성

```
Formology 변경 → Ontology 자동 갱신
  "검사항목 추가"
  → Form에 Section 추가
  → Entity 자동 생성 (사상 규칙)

Ontology 최적화 → Formology 영향 없음
  "정규화 3NF → BCNF"
  → DB 내부 최적화
  → Form은 그대로
```

---

## Ontology가 필요한 경우

Ontology가 **명시적으로** 필요한 1% 상황:

### 1. 표준 준수 필수

```
상황: 다기관 데이터 교환
예시: HL7 FHIR (의료), EDIFACT (물류)

전략:
┌─────────────────────────┐
│  내부: Formology        │ ← 빠른 개발
└────────┬────────────────┘
         ↓ 변환 레이어
┌─────────────────────────┐
│  외부: Standard Ontology│ ← 표준 준수
└─────────────────────────┘
```

### 2. 복잡한 추론

```
상황: AI 추론, 전문가 시스템
예시: 의료 진단 추론, 법률 자문

전략:
┌─────────────────────────┐
│  일반 기능: Formology    │ ← 99% 기능
└────────┬────────────────┘
         ↓
┌─────────────────────────┐
│  추론 모듈: Ontology     │ ← 1% 기능
└─────────────────────────┘
```

### 3. 장기 연구

```
상황: 5년+ 연구 프로젝트
예시: 학술 연구, 표준 개발

이유:
- 시간이 충분
- 이론적 완벽성 중요
- 전문가 확보 가능
```

---

## 오해와 진실

### 오해 1: "Formology는 Ontology를 무시한다"

**진실**:
```
Formology는 Ontology를 포함하고 자동 도출
Ontology ⊂ Formology

Formology를 쓰면 Ontology도 자동으로 얻음
```

### 오해 2: "Ontology가 더 정교하다"

**진실**:
```
정교함의 원천:
- Ontology: 추상화 수준
- Formology: 사상 규칙의 정확성

Formology의 사상 규칙이 정확하면
도출된 Ontology도 정교함
```

### 오해 3: "Formology는 간단한 시스템만"

**진실**:
```
복잡도는 레이어와 무관:
- 복잡한 Workflow → 복잡한 Formology
  → 복잡한 Ontology 도출

사례: 반도체 MES
- 300+ Format
- 자동 도출된 500+ Entity
```

---

## 비교 표: 관점의 차이

| 측면 | Ontology | Formology |
|------|----------|-----------|
| **철학** | 형이상학 (존재론) | 실용주의 (흐름론) |
| **질문** | What exists? | What flows? |
| **대상** | 실재/실체 | 현실세계 |
| **시간** | 시간 독립적 | 시간 의존적 |
| **레벨** | 하위 (도출됨) | 상위 (도출함) |
| **표현** | Object-Logic-Action | Workflow-Status-Format-Form |
| **언어** | Entity, Relation | 의뢰서, 보고서 |
| **주체** | 개발자 | 현업 + 개발자 |
| **산출물** | ERD, Class Diagram | 문서 목록, 양식 |
| **검증** | 전문가 검토 | 현업 즉시 검증 |
| **학습** | 2-3개월 | 1-2일 |
| **적용** | 1% (특수 상황) | 99% (일반 상황) |

---

## 실전 사례: 제조 MES

### Formology Layer (현실)

```
Workflow: 생산 관리
  ↓
작업지시 → 생산실적 → 품질검사 → 출하

Format 목록:
1. 작업지시서
2. 생산실적보고서
3. 검사의뢰서
4. 검사성적서
5. 출하지시서
6. 출하확인서

Status:
- 지시대기, 생산중, 생산완료
- 검사대기, 검사중, 검사완료
- 출하대기, 출하완료

→ 현업이 매일 쓰는 문서와 흐름
```

### Ontology Layer (자동 도출)

```
Entity 구조:
- WorkOrder (작업지시서)
  - ProductionResult (생산실적보고서) 1:N
    - QCRequest (검사의뢰서) 1:1
      - QCReport (검사성적서) 1:1
        - ShipmentOrder (출하지시서) 1:N
          - ShipmentConfirm (출하확인서) 1:1

State Machine:
- WorkOrder: PENDING → IN_PROGRESS → COMPLETED
- QCRequest: PENDING → IN_PROGRESS → PASSED/FAILED
- Shipment: PENDING → SHIPPED

Relation:
- WorkOrder → ProductionResult (1:N)
- QCRequest → QCReport (1:1)
- ShipmentOrder → WorkOrder (N:1)

→ 개발자가 사상 규칙으로 자동 생성
```

---

## 선택 가이드

### 시작점 선택

**질문**: 어디서 시작할까?

```
Always: Formology부터 시작
  ↓
필요시: Ontology 도출
  ↓
특수상황: Ontology 명시적 설계
```

**이유**:
1. Formology는 현업 검증 즉시 가능
2. Ontology는 자동 도출되므로 별도 노력 불필요
3. 특수한 경우만 Ontology 직접 설계

### 의사결정 플로우

```
시작
  ↓
Formology 설계 (필수)
  - Workflow 관찰
  - Format 도출
  - Form 스케치
  ↓
Ontology 자동 도출 (자동)
  - Format → Entity
  - Field → Attribute
  - Reference → Relation
  ↓
특수 요구사항?
  ├─ 표준 준수? → Ontology 변환 레이어 추가
  ├─ 복잡한 추론? → Ontology 추론 엔진 추가
  └─ 없음 → 그대로 사용
```

---

## 핵심 원칙

### Principle 1: Formology First

```
∀ system: Formology → Ontology
(모든 시스템은 Formology에서 Ontology로)

NOT: Ontology → Formology
```

### Principle 2: Ontology as Derivation

```
Ontology = f(Formology)
(Ontology는 Formology의 함수)

자동 도출 규칙:
- Format → Entity
- Section → Structure
- Field → Attribute
- Reference → Relation
- Status → State
- Workflow → Transition
```

### Principle 3: Layer Independence

```
Change Formology → Auto Update Ontology
Change Ontology (내부 최적화) → No Impact Formology

레이어 간 의존성은 단방향
```

---

## 다음 단계

### Formology 학습
- [왜 문서중심인가?](../01-introduction/why-formica.md) - 철학적 배경
- [문서 구조론](document-structure.md) - Format → Form → Document
- [데이터 모델 도출](data-modeling.md) - Formology → Ontology 사상

### 철학적 심화
- [철학적 기반](philosophical-foundations.md) - 인식론적 우위
- [양식학 이론](../03-theory/formology-foundations.md) - 이론적 토대

---

## 결론

**Formology ⊃ Ontology**

```
Formology는 Ontology를 대체하는 것이 아니라
Ontology를 포함하고 자동으로 도출하는 상위 개념

현실세계의 문서 흐름(Formology)을 정의하면
실재하는 개체와 관계(Ontology)가 자연스럽게 나타남
```

**실용적 조언**:
1. 항상 Formology로 시작
2. Ontology는 자동 도출
3. 특수한 경우만 Ontology 직접 설계
4. 두 레이어를 명확히 분리하여 관리

**핵심 메시지**:
> "Workflow First, Ontology Follows"
> "워크플로가 먼저, 존재론은 따라온다"
