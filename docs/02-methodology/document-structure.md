# 문서 구조론

## 개요

Formology의 핵심은 **문서(Document)**를 중심으로 한 업무 모델링입니다. 이 문서는 두 개의 독립적 차원으로 구성된 Formology의 개념 체계를 정의합니다.

> **핵심**: 문서는 단순한 파일이 아니라 **구조화된 정보 단위**이며, 전화통화부터 법적 계약서까지 모든 업무 커뮤니케이션을 포괄합니다.

---

## 모든 것이 문서다

### 포멀리티 스펙트럼 (Formality Spectrum)

업무의 모든 커뮤니케이션은 **얼마나 공식적인가**의 차이일 뿐, 본질적으로 모두 문서입니다:

```
비공식 ←────────────────────────────────→ 공식
(informal)                           (formal)

구두지시   메모   카톡   이메일   의뢰서   계약서
   ↓       ↓     ↓      ↓       ↓       ↓
  암묵적  간단한  디지털  공식    승인    법적
  양식    양식    양식   양식    양식    양식
```

**핵심 통찰**: 차이는 다음 세 가지뿐입니다:
1. **얼마나 공들여 쓰느냐** (작성 노력)
2. **얼마나 오래 보관하느냐** (보관 기간)
3. **얼마나 공식적이냐** (법적/제도적 효력)

### 일상 커뮤니케이션의 문서화

| 커뮤니케이션 | 포멀리티 | 문서 양식 | 시스템화 |
|-------------|---------|----------|---------|
| 전화 통화 | 낮음 | 통화기록지 | 선택적 |
| 구두 지시 | 낮음-중간 | 구두지시확인서 | 권장 |
| 화이트보드 스케치 | 낮음 | 회의 스케치 | 사진 |
| 카카오톡 | 중간 | 업무연락지 | 선택적 |
| 이메일 | 중간-높음 | 업무연락서 | 권장 |
| 공식 의뢰 | 높음 | 품질검사의뢰서 | 필수 |
| 법적 계약 | 매우 높음 | 계약서 | 필수 |

**실무 원칙**:
```
✓ 낮은 포멀리티: 간단히, 빠르게
✓ 중간 포멀리티: 기록 남기되, 간소하게
✓ 높은 포멀리티: 엄격한 양식, 승인 필수
```

---

## Formology 개념 체계: 2차원 구조

Formology는 **두 개의 독립적 차원(dimension)**으로 구성됩니다:

```
차원 1 (Vertical): Instance Hierarchy
  → Type에서 Instance로의 진화

차원 2 (Horizontal): Structural Decomposition
  → 전체에서 부분으로의 분해
```

---

## 차원 1: Instance Hierarchy (인스턴스 계층)

### FormType → Form → Document → Record

```
FormType (양식 유형 / 서식)
  │ = 추상 개념, 모양 없음
  │ = 필수 본질 정의
  │ = "이것이 없으면 이 서식이 아니다"
  │
  ├─ realization (구체화)
  ↓
Form (양식)
  │ = 구체적 레이아웃, 모양 있음
  │ = FormType의 1:N 구현
  │ = 필수 요소 + 부가 요소 + Decoration
  │
  ├─ fill (작성)
  ↓
Document (문서)
  │ = 양식지를 유저가 작성
  │ = 사람이 보고 읽는 단위
  │ = 의사소통/공유/승인의 대상
  │
  ├─ persist (저장)
  ↓
Record (레코드)
  = 데이터베이스에 저장된 데이터
  = 시스템이 처리하는 단위
```

### L0: FormType (양식 유형 / 서식)

**정의**: 모양 없이 개념만 존재하는 추상적 정의

**역할**: 이 서식의 **가치를 만드는 필수 본질** 정의

**구성**:
- 필수 Section/Field 정의
- 엔티티 구조 결정
- Reference/Attachment 유형 정의

**예시**:
```
FormType: "품질검사의뢰서"

필수 Section/Field (본질):
  ├─ Section: 기본정보 (Main Entity)
  │   ├─ Field: 제품명 ✓ 필수
  │   ├─ Field: LOT번호 ✓ 필수
  │   └─ Field: 수량 ✓ 필수
  │
  ├─ Section: 검사항목 (Child, 1:N)
  │   └─ Field: 검사항목명 ✓ 필수
  │
  └─ Section: 참조정보 (Reference)
      └─ Field: 생산계획서 참조 ✓ 필수

의미: "품질검사의뢰서"로 인정받기 위한 최소 요건
```

**특징**:
- ✓ 플랫폼 독립적
- ✓ 구현 방법 무관
- ✓ 불변의 본질
- ✓ 업무 개념 자체

### L1: Form (양식)

**정의**: FormType의 구체적 구현, 모양을 가진 레이아웃

**구성**: 필수(상속) + 부가(확장) + Decoration(최적화)

**1:N 관계**: 하나의 FormType → 여러 Form

**예시**:
```
FormType: "품질검사의뢰서"
  │
  ├─ Form 1: Web 입력 양식 (입력 최적화)
  │   ├─ [필수] 기본정보, 검사항목, 참조정보 ← FormType 상속
  │   ├─ [부가] 의뢰자 정보 Section ← 추가
  │   ├─ [부가] 특이사항 Field ← 추가
  │   └─ [Decoration]
  │       ├─ 자동완성 UI
  │       ├─ 실시간 유효성 검사
  │       └─ 진행률 표시
  │
  ├─ Form 2: 모바일 입력 양식 (모바일 최적화)
  │   ├─ [필수] 동일 Section/Field
  │   ├─ [부가] QR 스캔 Field ← 모바일 전용
  │   └─ [Decoration]
  │       ├─ 터치 최적화 UI
  │       └─ 간소화 레이아웃
  │
  ├─ Form 3: PDF 출력 양식 (출력 최적화)
  │   ├─ [필수] 동일 Section/Field
  │   ├─ [부가] 서명란 ← 출력 전용
  │   └─ [Decoration]
  │       ├─ 인쇄 레이아웃
  │       └─ 로고/도장 위치
  │
  └─ Form 4: API JSON Schema (API 최적화)
      ├─ [필수] 동일 Section/Field → JSON
      └─ [Decoration]
          └─ REST 표준 준수
```

**특징**:
- ✓ 목적별 최적화 (입력/보기/출력/플랫폼)
- ✓ FormType 본질 유지 (필수 요소 상속)
- ✓ 맥락별 확장 (부가 요소 추가)
- ✓ UX 최적화 (Decoration)

**포멀리티 레벨**:
- **레벨 0**: 암묵적 양식 (전화, 구두)
- **레벨 1**: 간단한 양식 (메모, 카톡)
- **레벨 2-3**: 표준 양식 (공식 문서)
- **레벨 4-5**: 엄격한 양식 (법적 문서)

### L2: Document (문서)

**정의**: Form(양식지)을 소비자(유저)가 작성한 결과

**관점**: 사람의 관점 (보고, 읽고, 공유하고, 승인하는 단위)

**예시**:
```
Form: "Web 입력 양식"
  │
  ├─ 유저 작성
  │   ├─ 제품명: "ABC 모터" 입력
  │   ├─ LOT번호: "L20241108-001" 입력
  │   ├─ 수량: 500 입력
  │   ├─ 검사항목: [외관검사, 치수측정] 선택
  │   └─ 생산계획서: PLAN-045 참조
  │
  ↓ 작성 완료
  │
Document: "QC-REQ-001"
  = 보여지고 읽히는 완성된 문서

  내용:
  ├─ 기본정보: ABC 모터, L20241108-001, 500
  ├─ 검사항목: 외관검사, 치수측정
  └─ 참조: 생산계획서 PLAN-045
```

**특징**:
- ✓ 고유 식별자 (문서번호)
- ✓ 작성자/작성일시
- ✓ 상태 (작성중/제출/승인)
- ✓ 이력 추적

**생명주기**:
```
작성 → 제출 → 검토 → 승인 → 완료
  ↓      ↓      ↓      ↓      ↓
 초안   확정   보류   승인   보관
```

### L3: Record (레코드)

**정의**: Document를 데이터베이스에 저장한 데이터

**관점**: 시스템의 관점 (처리하고, 검색하고, 분석하는 단위)

**저장 구조**:
```
Document: "QC-REQ-001"
  │
  ├─ persist (저장)
  ↓
Records (데이터베이스):

├─ Main Record (메인 엔티티)
│   qc_requests 테이블
│   ├─ id: 1
│   ├─ document_no: "QC-REQ-001"
│   ├─ product_name: "ABC 모터"
│   ├─ lot_number: "L20241108-001"
│   ├─ quantity: 500
│   └─ plan_id: 45 ← Reference (FK)
│
├─ Child Records (자식, 1:N)
│   qc_items 테이블
│   ├─ {id: 1, request_id: 1, item: "외관검사"}
│   └─ {id: 2, request_id: 1, item: "치수측정"}
│
├─ Reference Section 처리
│   └─ plan_id: 45 (FK)
│       → production_plans 테이블 참조
│       → 원본 수정 시 동반 업데이트 (살아있는 링크)
│
└─ Attachment Section 처리
    attachments 테이블
    ├─ {id: 1, request_id: 1, file: "도면.pdf"}
    └─ 파일 복제본 저장 (스냅샷)
        → 원본 변경과 무관 (그 시점의 증거)
```

**특징**:
- ✓ 정규화된 데이터
- ✓ 관계형 저장 (FK, 1:N, M:N)
- ✓ 쿼리/분석 최적화
- ✓ 트랜잭션 보장

---

## 차원 2: Structural Decomposition (구조 분해)

### Section → Field

Form과 Document는 **구조적으로 분해**됩니다:

```
Form / Document
  │
  ├─ 구조 분해
  ↓
Section (섹션)
  │ = 의미적 그룹
  │ = 엔티티 경계 결정
  │ = Reference/Attachment 유형 결정
  │
  ├─ 구조 분해
  ↓
Field (필드)
  = 원자적 데이터 단위
  = 엔티티의 속성 (column)
```

### Section (섹션)

**정의**: 문서 내 의미적 그룹, 엔티티 경계를 결정하는 단위

**역할**: 데이터베이스 테이블 구조 결정

**3가지 유형**:

**1. Main Section (메인 섹션)**
```
섹션: "기본 정보"
  │
  └─ 엔티티: qc_requests (메인 테이블)
      ├─ Field: document_no
      ├─ Field: product_name
      ├─ Field: lot_number
      └─ Field: quantity
```

**2. Child Section (자식 섹션)**
```
섹션: "검사 항목" (체크박스 여러 개)
  │
  └─ 엔티티: qc_items (1:N 자식 테이블)
      ├─ Field: request_id (FK)
      ├─ Field: item_name
      └─ Field: is_checked
```

**3. Reference/Attachment Section**
```
섹션: "참조 문서"
  ├─ Reference: production_plan_id (FK)
  │   → 살아있는 링크, 동반 업데이트
  │
  └─ Attachment: files (1:N 파일 테이블)
      → 복제 저장, 스냅샷
```

**Section 인식 패턴**:
```
워크숍에서 Section 발견:

"이 부분은 여러 개 입력해요" → Child Section (1:N)
"다른 문서를 참조해요" → Reference Section
"파일을 첨부해요" → Attachment Section
"기본 정보예요" → Main Section
```

### Field (필드)

**정의**: 원자적 데이터 단위, 엔티티의 속성

**역할**: 데이터베이스 Column으로 매핑

**타입**:
```
Text Field:
  - 단일 행: VARCHAR
  - 여러 행: TEXT

Numeric Field:
  - 정수: INTEGER
  - 실수: DECIMAL

Date/Time Field:
  - 날짜: DATE
  - 시간: TIME
  - 날짜시간: TIMESTAMP

Selection Field:
  - 단일 선택: FK (참조키)
  - 다중 선택: M:N (연결 테이블)

Boolean Field:
  - 체크박스: BOOLEAN
```

**예시**:
```
Section: "기본 정보"
  ├─ Field: 제품명 (Text) → product_name VARCHAR
  ├─ Field: 수량 (Numeric) → quantity INTEGER
  ├─ Field: 검사일자 (Date) → inspection_date DATE
  ├─ Field: 제품 (Selection) → product_id FK
  └─ Field: 긴급여부 (Boolean) → is_urgent BOOLEAN
```

---

## Reference vs Attachment 상세

### Reference (참조)

**정의**: 다른 문서/데이터를 가리키는 살아있는 링크

**저장 방식**: Foreign Key (FK)

**특징**:
- ✓ 원본 수정 시 자동 반영
- ✓ 최신 상태 유지
- ✓ 저장 공간 효율적
- ✓ 일관성 유지

**사용 시나리오**:
- ✓ 현재 규정 참조
- ✓ 마스터 데이터 참조
- ✓ 진행 중인 프로젝트 참조

**예시**:
```
품질검사의뢰서 (Document)
  └─ 참조: 생산계획서 PLAN-045

Record 저장:
  qc_requests.plan_id = 45 (FK)

  → production_plans 테이블의 id=45 참조
  → 생산계획서 수정 시 자동 반영
  → 최신 정보 유지
```

### Attachment (붙임)

**정의**: 그 시점의 복사본을 저장하는 스냅샷

**저장 방식**: 파일 복제 + 메타데이터 테이블

**특징**:
- ✓ 원본 변경과 무관
- ✓ 그 시점의 증거 보존
- ✓ 법적 효력
- ✓ 감사 대응

**사용 시나리오**:
- ✓ 승인 시점 자료 (증빙)
- ✓ 계약 증빙 자료
- ✓ 감사 대응 자료

**예시**:
```
품질검사보고서 (Document)
  └─ 붙임: 도면_v1.3.pdf

Record 저장:
  attachments 테이블
  ├─ id: 1
  ├─ request_id: 1
  ├─ file_name: "도면_v1.3.pdf"
  ├─ file_path: "/uploads/2024/11/..."
  ├─ snapshot: [파일 복제본]
  └─ uploaded_at: 2024-11-08

  → 원본 도면이 v2.0으로 변경되어도
  → 이 보고서는 v1.3 유지
  → 그 시점의 증거 보존
```

### 선택 기준

| 기준 | Reference | Attachment |
|------|-----------|------------|
| **변경 반영** | 자동 반영 | 변경 무관 |
| **최신성** | 항상 최신 | 시점 고정 |
| **증거 능력** | 약함 | 강함 |
| **법적 효력** | 약함 | 강함 |
| **저장 공간** | 효율적 | 중복 허용 |
| **포멀리티** | 레벨 0-3 | 레벨 4-5 |

---

## 완전한 플로우 다이어그램

```
┌─────────────────────────────────────────────────────┐
│ FormType: "품질검사의뢰서" (양식 유형/서식)          │
│                                                     │
│ 필수 본질 정의:                                      │
│ ├─ Section: 기본정보 (Main)                         │
│ │   ├─ Field: 제품명 ✓                            │
│ │   └─ Field: 수량 ✓                              │
│ ├─ Section: 검사항목 (Child, 1:N)                  │
│ └─ Section: 참조정보 (Reference)                    │
└─────────────────────────────────────────────────────┘
                    │
                    ├─ realization (구체화, 1:N)
                    ↓
┌─────────────────────────────────────────────────────┐
│ Form: Web 입력 양식                                  │
│                                                     │
│ = 필수 Section/Field (상속)                         │
│ + 부가 Section/Field (확장)                         │
│ + Decoration (최적화)                              │
│                                                     │
│ ├─ [필수] 기본정보, 검사항목, 참조정보              │
│ ├─ [부가] 의뢰자정보, 특이사항                      │
│ └─ [Decoration] 자동완성, 유효성검사               │
└─────────────────────────────────────────────────────┘
                    │
                    ├─ fill (작성)
                    ↓
┌─────────────────────────────────────────────────────┐
│ Document: "QC-REQ-001"                              │
│                                                     │
│ 제품명: ABC 모터                                     │
│ 수량: 500                                           │
│ 검사항목: 외관검사, 치수측정                         │
│ 참조: 생산계획서 PLAN-045                            │
└─────────────────────────────────────────────────────┘
                    │
                    ├─ persist (저장)
                    ↓
┌─────────────────────────────────────────────────────┐
│ Records (데이터베이스)                               │
│                                                     │
│ qc_requests (메인)                                  │
│ ├─ id: 1                                           │
│ ├─ product_name: "ABC 모터"                        │
│ ├─ quantity: 500                                   │
│ └─ plan_id: 45 (FK) ← Reference                   │
│                                                     │
│ qc_items (자식, 1:N)                               │
│ ├─ {id:1, item:"외관검사"}                         │
│ └─ {id:2, item:"치수측정"}                         │
│                                                     │
│ attachments (붙임, 복제)                            │
│ └─ {file:"도면.pdf", snapshot...}                  │
└─────────────────────────────────────────────────────┘
```

---

## 포멀리티와 계층의 관계

### 포멀리티 레벨별 특성

| 레벨 | 포멀리티 | FormType | Form | Document | Record |
|------|---------|----------|------|----------|--------|
| **0** | 암묵적 | 불필요 | 간소 | 선택적 | 최소 |
| **1** | 낮음 | 간단 | 단순 | 권장 | 기본 |
| **2** | 중간 | 표준 | 표준 | 필수 | 표준 |
| **3** | 높음 | 엄격 | 엄격 | 필수 | 완전 |
| **4** | 매우높음 | 법적 | 법적 | 필수 | 법적 |
| **5** | 법적 | 법정 | 법정 | 필수 | 감사 |

### 시스템 요구사항

**레벨 0-1: 편의성 우선**
- FormType: 암묵적 또는 간소화
- Form: 빠른 입력 UI
- Document: 간단한 저장
- Record: 최소 정보만 저장

**레벨 2-3: 균형**
- FormType: 명확한 정의
- Form: 표준 양식 준수
- Document: 승인 워크플로우
- Record: 이력 관리

**레벨 4-5: 엄격함 우선**
- FormType: 법적 요건 준수
- Form: 엄격한 양식
- Document: 다단계 승인
- Record: 감사 추적, 증거 보존

---

## 실전 예시

### 포멀리티 진화 (Progressive Formalization)

```
레벨 0: 전화 통화 (암묵적)
  ↓ (문제 발생 시 공식화 필요)
레벨 1: 통화기록 메모
  ↓ (중요도 상승)
레벨 2: 업무연락서 작성
  ↓ (승인 필요)
레벨 3: 공식 의뢰서
  ↓ (법적 분쟁)
레벨 5: 법정 증거 자료

실제 사례: "제품 납기 변경"
1. 전화로 협의 (레벨 0)
2. 이메일 확인 (레벨 2)
3. 변경요청서 작성 (레벨 3)
4. 변경계약서 체결 (레벨 5)
```

---

## 다음 단계

문서 구조를 이해했다면:

- **[문서의 종류와 본질](document-types.md)**: 서/지/표/록 분류와 포멀리티
- **[데이터 모델 도출](data-modeling.md)**: Section-Field에서 ERD로
- **[문서 관계](document-relationships.md)**: Reference와 Attachment 상세
- **[워크숍 진행](../04-workshop/facilitation-guide.md)**: 실전 적용

---

**핵심**: Formology는 두 개의 독립적 차원으로 문서를 이해합니다
- **인스턴스 차원**: FormType → Form → Document → Record
- **구조 차원**: Section → Field
