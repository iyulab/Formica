# Formology 용어 표준 (Terminology Standard)

> **Formology 방법론의 공식 용어 정의 및 표준 문서**
>
> 모든 Formology 문서는 이 표준을 따릅니다.

---

## 문서 목적

이 문서는 Formology 방법론에서 사용하는 모든 핵심 용어의 **공식 정의**와 **사용 규칙**을 제공합니다.

**역할**:
- ✅ 용어 해석의 단일 기준점
- ✅ 문서 간 일관성 보장
- ✅ 현업-개발자 간 소통 표준

**범위**: 개념 정의와 사상 규칙 (구현 코드 제외)

---

## 핵심 개념 체계

Formology는 **2차원 개념 체계**를 사용합니다:

```
[Dimension 1: 인스턴스 계층 - Vertical]
FormType → Form → Document → Record

[Dimension 2: 구조 분해 - Horizontal]
Form = Section[] + Field[]
```

### 관계도

```
FormType (서식 개념)
  ├─ 추상적 정의
  └─ "품질검사의뢰서", "작업지시서" 등

↓ 구체화

Form (양식 구조)
  ├─ Section[] (구조 단위)
  │   ├─ Main Section
  │   ├─ Child Section (1:N)
  │   ├─ Reference Section (FK)
  │   └─ Attachment Section (snapshot)
  └─ Field[] (입력 단위)
      ├─ Text, Numeric, Date
      ├─ Selection, Checklist
      └─ FileUpload, TextArea 등

↓ 인스턴스화

Document (문서 인스턴스)
  ├─ 유저가 작성한 구체적 문서
  └─ "품질검사의뢰서 #QC-2024-001"

↓ 저장

Record (레코드)
  ├─ 데이터베이스에 저장된 데이터
  └─ qc_requests 테이블의 row
```

---

## Dimension 1: 인스턴스 계층 (Instance Hierarchy)

### L0: FormType (서식 개념)

**정의**: 업무에서 사용하는 **문서의 추상적 개념**

**특징**:
- 모양 없는 순수 개념 (shapeless concept)
- 현업이 "우리는 이런 서식 씁니다"라고 부르는 이름
- 1개 FormType → N개 Form 가능 (1:N)

**예시**:
```
- "작업지시서" (FormType)
- "품질검사의뢰서" (FormType)
- "설비점검일지" (FormType)
- "월별생산현황표" (FormType)
```

**영문 표기**: FormType (대문자 시작, 띄어쓰기 없음)

**한글 표기**: 서식, 서식 개념

---

### L1: Form (양식 구조)

**정의**: FormType의 **구체적 Section/Field 구조**

**특징**:
- 모양이 있는 구체적 레이아웃 (concrete layout)
- Section과 Field로 구성
- 1개 FormType → N개 Form (예: 동일 FormType의 버전별/부서별 양식)

**구조**:
```
Form = Section[] + Field[]

Form 예시:
┌─────────────────────────────┐
│ [Main Section] 헤더         │
├─────────────────────────────┤
│ 문서번호: [자동채번] Field  │
│ 작성자: [자동] Field        │
├─────────────────────────────┤
│ [Main Section] 본문         │
│ 제품명: [Selection] Field   │ ← Reference Section
│ 수량: [Numeric] Field       │
├─────────────────────────────┤
│ [Child Section] 상세 (1:N)  │
│ 항목: [Text] Field          │
│ 수량: [Numeric] Field       │
└─────────────────────────────┘
```

**영문 표기**: Form (대문자 시작)

**한글 표기**: 양식, 양식 구조

---

### L2: Document (문서 인스턴스)

**정의**: 유저가 **실제로 작성한 Form의 인스턴스**

**특징**:
- Form의 Field에 데이터가 채워진 상태
- 문서 ID와 고유 상태를 가짐 (초안/제출/승인 등)
- 1개 Form → N개 Document

**예시**:
```
FormType: "작업지시서"
→ Form: 작업지시서 양식 V1.2
  → Document: 작업지시서 #WO-2024-001 (초안)
  → Document: 작업지시서 #WO-2024-002 (승인됨)
  → Document: 작업지시서 #WO-2024-003 (완료)
```

**상태 흐름**:
```
Document 생명주기:
생성 → 초안 → 제출 → 승인 → 완료 → 보관
```

**영문 표기**: Document (대문자 시작)

**한글 표기**: 문서, 문서 인스턴스

---

### L3: Record (레코드)

**정의**: Document가 **데이터베이스에 저장된 형태**

**특징**:
- Section → Entity (테이블)
- Field → Attribute (컬럼)
- 데이터베이스 row로 구현

**사상 규칙**:
```
Form Structure → Database Schema

[Main Section]              → main_table
  Field: 문서번호          → column: document_id
  Field: 제품명 (FK)       → column: product_id (FK)
  Field: 수량              → column: quantity

[Child Section] (1:N)       → child_table
  Field: 항목              → column: item_name
  Field: 수량              → column: item_quantity
```

**영문 표기**: Record (대문자 시작)

**한글 표기**: 레코드

---

## Dimension 2: 구조 분해 (Structural Decomposition)

### Section (섹션)

**정의**: Form을 구성하는 **의미 단위 그룹**

**역할**:
- Form의 논리적 구역
- Section → Entity (테이블) 사상 기준
- 관계 정의의 기본 단위

**Section 유형 4가지**:

#### 1. Main Section (주 섹션)

**정의**: Document의 **핵심 정보를 담는 메인 섹션**

**특징**:
- 1개 Form → 1개 이상의 Main Section
- Main Section → Main Entity (주 테이블)

**예시**:
```
FormType: "작업지시서"
Form 구조:
[Main Section] 지시 헤더
  - 지시번호: Field
  - 작성일: Field

[Main Section] 지시 본문
  - 제품명: Field (FK)
  - 수량: Field
  - 납기일: Field

→ Record: work_orders 테이블 (Main Entity)
```

---

#### 2. Child Section (자식 섹션)

**정의**: Main Section에 종속된 **1:N 관계 섹션**

**특징**:
- 1개 Main Section → N개 Child Section
- Child Section → Child Entity (자식 테이블)
- 부모 테이블의 FK를 가짐

**예시**:
```
FormType: "작업지시서"
Form 구조:
[Main Section] 지시 정보
  - 지시번호: Field

[Child Section] 자재 내역 (1:N)
  - 원료명: Field (FK)
  - 필요량: Field
  - + 추가 버튼 (N개 입력 가능)

→ Record:
  work_orders (Main Entity)
    ↓ 1:N
  work_order_materials (Child Entity)
```

**구현 특징**:
- 화면: "추가" 버튼으로 N개 입력
- DB: 별도 자식 테이블
- FK: parent_id (부모 테이블 참조)

---

#### 3. Reference Section (참조 섹션)

**정의**: 다른 FormType의 Document를 **실시간 링크(FK)로 참조**

**특징**:
- 살아있는 링크 (living link)
- 원본 수정 시 자동 반영
- Reference Section Field → FK (Foreign Key)

**예시**:
```
FormType: "품질검사보고서"
Form 구조:
[Main Section] 보고 정보
  - 보고번호: Field

[Reference Section] 의뢰 정보
  - 의뢰서: Field (FK → qc_requests)
    ↓
    원본 "품질검사의뢰서" Document를 실시간 참조

→ Record:
  qc_reports.request_id (FK) → qc_requests.id
```

**동작**:
```
원본 의뢰서 수정:
qc_requests.product_id = A → B

↓ 자동 반영

보고서에서 조회:
qc_reports.request_id → qc_requests.product_id = B (최신 값)
```

**사용 시나리오**:
- 승인/처리 문서가 의뢰 문서 참조
- 하위 프로세스가 상위 프로세스 참조
- 최신 정보 동기화 필요

---

#### 4. Attachment Section (첨부 섹션)

**정의**: 다른 FormType의 Document를 **과거 시점 스냅샷으로 복제**

**특징**:
- 고정된 사본 (frozen copy)
- 원본 수정과 무관
- Attachment Section → 복제 테이블 or 파일

**예시**:
```
FormType: "출하확인서"
Form 구조:
[Main Section] 출하 정보
  - 출하번호: Field

[Attachment Section] 의뢰서 정보 (스냅샷)
  - 제품명: Field (복제)
  - 수량: Field (복제)
  - 납기일: Field (복제)
    ↓
    출하 시점의 의뢰서 데이터를 그대로 복제

→ Record:
  shipments 테이블
  ├─ product_name (복제)
  ├─ quantity (복제)
  └─ due_date (복제)

또는
  shipments 테이블
  └─ request_snapshot (JSON 복제)
```

**동작**:
```
원본 의뢰서 수정:
work_orders.quantity = 100 → 200

↓ 영향 없음

출하확인서 조회:
shipments.quantity = 100 (출하 시점 고정)
```

**사용 시나리오**:
- 증빙 보존 필요 (세무, 계약)
- 과거 시점 재현 필요
- 원본 변경과 무관하게 보존

**구현 방법 2가지**:
1. **복제 테이블**: 필요한 컬럼만 복제
2. **파일/JSON**: 전체 Document를 스냅샷

---

### Section 비교 요약

| 구분 | Reference Section | Attachment Section |
|------|-------------------|-------------------|
| **개념** | 살아있는 링크 | 고정된 사본 |
| **원본 수정** | 자동 반영 | 영향 없음 |
| **DB 구현** | FK | 복제 테이블/JSON |
| **사용 예시** | 승인/처리 문서 | 증빙/계약 문서 |
| **시점** | 실시간 (현재) | 고정 (과거) |

**선택 기준**:
```
실시간 동기화 필요? → Reference Section
과거 시점 고정 필요? → Attachment Section
```

---

### Field (필드)

**정의**: Section 내 **개별 입력 단위**

**역할**:
- 유저 입력 최소 단위
- Field → Attribute (컬럼) 사상

**Field 유형**:

#### 기본 입력 Field

| Field 유형 | 설명 | 예시 | DB 타입 |
|-----------|------|------|---------|
| **Text** | 단행 텍스트 | 문서번호, 이름 | VARCHAR |
| **TextArea** | 다행 텍스트 | 비고, 설명 | TEXT |
| **Numeric** | 숫자 | 수량, 금액 | INTEGER, DECIMAL |
| **Date** | 날짜 | 작성일, 납기일 | DATE |
| **Time** | 시간 | 시작시간, 종료시간 | TIME, TIMESTAMP |
| **Checkbox** | 단일 체크 | 동의 여부 | BOOLEAN |

#### 선택 입력 Field

| Field 유형 | 설명 | 예시 | DB 타입 |
|-----------|------|------|---------|
| **Selection** | 단일 선택 (드롭다운) | 제품명, 담당자 | VARCHAR (FK) |
| **Radio** | 단일 선택 (라디오) | 승인/반려 | ENUM |
| **Checklist** | 다중 선택 | 검사항목 | JSON, M:N 테이블 |

#### 특수 Field

| Field 유형 | 설명 | 예시 | DB 타입 |
|-----------|------|------|---------|
| **FileUpload** | 파일 첨부 | 사진, PDF | VARCHAR (경로) |
| **자동채번** | 시스템 자동 생성 | 문서번호 | VARCHAR (자동) |
| **자동** | 로그인 유저 등 | 작성자, 작성일 | 자동 입력 |

---

#### Reference Section의 Selection Field = FK

**규칙**:
```
Reference Section 내의 Selection Field
  → FK (Foreign Key) 사상
```

**예시**:
```
FormType: "품질검사의뢰서"
Form 구조:
[Main Section] 의뢰 정보
  제품명: [Selection ▼] ← Reference Section Field
    ↓
  Record: qc_requests.product_id (FK → products.id)

[Main Section] 의뢰 정보
  수량: [Numeric] ← Main Section Field
    ↓
  Record: qc_requests.quantity (일반 컬럼)
```

**구분**:
- **Reference Section의 Selection** → FK (다른 테이블 참조)
- **Main Section의 Selection** → ENUM 또는 일반 컬럼

---

## 사상 규칙 (Mapping Rules)

### 규칙 1: Section → Entity

```
Section 유형 → Entity (테이블)

Main Section      → Main Entity (주 테이블)
Child Section     → Child Entity (자식 테이블, 1:N)
Reference Section → FK (참조, 실시간)
Attachment Section → 복제 테이블 또는 JSON (과거 고정)
```

**예시**:
```
FormType: "작업지시서"
Form 구조:
[Main Section] 지시 정보
[Child Section] 자재 내역
[Reference Section] 작업표준서

→ Record:
work_orders (Main Entity)
  ├─ work_order_materials (Child Entity, 1:N)
  └─ standard_id (FK to work_standards)
```

---

### 규칙 2: Field → Attribute

```
Field 유형 → Attribute (컬럼)

Text          → VARCHAR
Numeric       → INTEGER, DECIMAL
Date          → DATE
Selection (FK)→ VARCHAR (FK)
Selection     → ENUM, VARCHAR
Checklist     → JSON, M:N 테이블
FileUpload    → VARCHAR (파일 경로)
```

**예시**:
```
Form Field:
  제품명: [Selection ▼] (Reference Section)
  수량: [Numeric]
  납기일: [Date]

→ Record Attributes:
  product_id: VARCHAR (FK)
  quantity: INTEGER
  due_date: DATE
```

---

### 규칙 3: Child Section → 1:N 관계

```
Parent Section (Main)
  ↓ 1:N
Child Section
  ↓
Parent Entity
  ↓ FK
Child Entity (parent_id)
```

**예시**:
```
FormType: "작업지시서"
Form:
[Main Section] 지시 정보 (1)
  ↓
[Child Section] 자재 내역 (N)
  - 원료: Field
  - 수량: Field

→ Record:
work_orders (id, ...)
  ↓ 1:N
work_order_materials (id, order_id FK, material_id, quantity)
```

---

### 규칙 4: Reference Section → FK

```
Reference Section Field (Selection)
  ↓
FK to Referenced Entity
  ↓
실시간 동기화 (원본 수정 시 자동 반영)
```

**예시**:
```
FormType: "품질검사보고서"
Form:
[Reference Section] 의뢰 정보
  의뢰서: [참조 버튼]

→ Record:
qc_reports.request_id (FK) → qc_requests.id
```

---

### 규칙 5: Attachment Section → 복제

```
Attachment Section
  ↓
복제 테이블 or JSON
  ↓
과거 시점 고정 (원본 수정과 무관)
```

**예시**:
```
FormType: "출하확인서"
Form:
[Attachment Section] 의뢰서 스냅샷
  제품명: Field (복제)
  수량: Field (복제)

→ Record:
shipments 테이블
  ├─ product_name (복제)
  ├─ quantity (복제)
```

---

## 명명 규칙 (Naming Conventions)

### FormType 명명

**원칙**: 현업이 부르는 이름 그대로 (한글)

**패턴**:
```
[업무영역][대상][접미사]

접미사별 의미:
-서 (書): 공식 요청/의뢰 문서 (작업지시서, 품질검사의뢰서)
-지 (紙): 일상 기록 (작업일지, 점검일지)
-표 (表): 집계/목록 (월별생산현황표, 재고현황표)
-록 (錄): 시간순 로그 (불량이력록, 변경이력록)
```

**예시**:
```
✅ 좋은 예:
- 작업지시서
- 품질검사의뢰서
- 설비점검일지
- 월별생산현황표

❌ 나쁜 예:
- 작업지시 (접미사 누락)
- WorkOrder (영문 사용)
- 작업_지시_서 (언더스코어 사용)
```

---

### Form 명명

**원칙**: FormType + "양식" + 버전/구분

**패턴**:
```
[FormType명] 양식 [V버전] [구분]

예시:
- 작업지시서 양식 V1.2
- 품질검사의뢰서 양식 (생산부)
- 설비점검일지 양식 V2.0
```

---

### Document 명명

**원칙**: FormType + "#" + 문서번호

**패턴**:
```
[FormType명] #[문서번호]

예시:
- 작업지시서 #WO-2024-001
- 품질검사의뢰서 #QC-2024-045
- 설비점검일지 #INSP-2024-12-15
```

---

### Record 명명

**원칙**: 영문 snake_case (데이터베이스 표준)

**패턴**:
```
FormType → 테이블명 (복수형, snake_case)

작업지시서 → work_orders
품질검사의뢰서 → qc_requests
설비점검일지 → equipment_inspections
```

**컬럼명**:
```
Field → 컬럼명 (snake_case)

문서번호 → document_id
제품명 → product_id (FK인 경우)
수량 → quantity
작성일 → created_at
```

---

### 구현 레이어별 명명

| 레이어 | 규칙 | 예시 |
|--------|------|------|
| **현업/화면** | 한글 원문 | 작업지시서, 품질검사의뢰서 |
| **API Endpoint** | kebab-case | `/api/v1/work-orders` |
| **Database** | snake_case | `work_orders`, `qc_requests` |
| **Code (변수)** | camelCase | `workOrder`, `qcRequest` |
| **Code (클래스)** | PascalCase | `WorkOrder`, `QcRequest` |

---

## 문서 분류 체계

### 서/지/표/록 분류

**접미사별 특성**:

#### -서 (書): 공식 문서

**특징**:
- 공식 요청/의뢰/승인
- 워크플로우 (초안→제출→승인)
- 법적/계약적 효력

**예시**:
- 작업지시서
- 품질검사의뢰서
- 자재요청서
- 승인요청서

**Document 흐름**:
```
생성 → 초안 → 제출 → 승인 → 완료
```

---

#### -지 (紙): 일상 기록

**특징**:
- 일상 업무 기록
- 시간순 정렬 (날짜별)
- 승인 없이 즉시 작성

**예시**:
- 작업일지
- 설비점검일지
- 생산일지
- 근무일지

**Document 흐름**:
```
생성 → 작성 → 저장 (승인 없음)
```

---

#### -표 (表): 집계/현황

**특징**:
- 데이터 집계/통계
- 시스템 자동 생성 가능
- 읽기 전용 (조회용)

**예시**:
- 월별생산현황표
- 재고현황표
- 불량현황표
- 가동률현황표

**Document 특징**:
```
자동 생성 (조회 시점)
읽기 전용
PDF/Excel 출력
```

---

#### -록 (錄): 이력/로그

**특징**:
- 변경 이력 추적
- 시간순 누적 기록
- 삭제/수정 불가 (Append-only)

**예시**:
- 불량이력록
- 변경이력록
- 승인이력록
- 접근이력록

**Document 특징**:
```
Append-only (추가만 가능)
시간순 정렬
삭제/수정 불가
```

---

## 용어 사용 가이드

### ✅ 올바른 용어 사용

```
✅ "품질검사의뢰서 FormType을 정의합니다"
✅ "작업지시서 Form의 Section 구조를 스케치합니다"
✅ "Main Section은 work_orders 테이블로 사상됩니다"
✅ "Reference Section은 FK로 구현됩니다"
✅ "Child Section은 1:N 관계입니다"
✅ "Document는 초안→제출→승인 흐름을 따릅니다"
```

---

### ❌ 피해야 할 용어 사용

```
❌ "품질검사의뢰서 엔티티" → ✅ "품질검사의뢰서 FormType"
❌ "양식을 정의합니다" → ✅ "FormType을 정의하고 Form 구조를 스케치합니다"
❌ "템플릿" → ✅ "Form" 또는 "양식"
❌ "폼" → ✅ "Form" (영문) 또는 "양식" (한글)
❌ "데이터 모델" → ✅ "Record 구조" 또는 "Entity 모델"
❌ "화면" → ✅ "Form 레이아웃" 또는 "Document 입력 화면"
```

---

### 상황별 용어 선택

#### 현업과 대화 시

```
✅ 사용: FormType, 양식, 문서, 섹션, 항목
❌ 회피: Entity, Record, Table, FK, Schema
```

**예시**:
```
"작업지시서라는 서식(FormType)이 있고요,
그 양식(Form)은 지시 정보 섹션(Main Section)과
자재 내역 섹션(Child Section)으로 나뉩니다.
각 섹션에는 제품명, 수량 같은 항목(Field)이 있어요."
```

---

#### 개발자와 대화 시

```
✅ 사용: FormType, Form, Section, Field, Record, Entity, FK
✅ 병행: FormType (= 서식 개념), Form (= 양식 구조)
```

**예시**:
```
"작업지시서 FormType의 Form 구조에서
Main Section은 work_orders 테이블(Main Entity)로 사상되고,
Child Section은 work_order_materials 테이블로 1:N 관계를 형성합니다.
Reference Section의 Selection Field는 FK로 구현됩니다."
```

---

#### 문서 작성 시

```
✅ 원칙: 개념 도입 시 한글+영문 병기
✅ 반복 시: 영문 우선 (FormType, Form, Section, Field)
```

**예시**:
```
"업무에서 사용하는 서식 개념을 **FormType**이라 합니다.
FormType은 구체적인 양식 구조인 **Form**으로 구체화됩니다.
Form은 **Section**(섹션)과 **Field**(필드)로 구성됩니다.

각 FormType은..."
```

---

## 개념 흐름도 (Concept Flow)

### 전체 흐름

```
1. 현업 워크숍
   ↓
   FormType 목록 도출 (22개 서식 브레인스토밍)

2. 양식 설계
   ↓
   Form 구조 스케치 (Section/Field)

3. 관계 정의
   ↓
   Section 간 관계 파악 (Reference/Child/Attachment)

4. 데이터 사상
   ↓
   Section → Entity, Field → Attribute

5. 구현
   ↓
   Database, API, Frontend
```

---

### 개념 변환 흐름

```
[현업 언어]
"우리는 작업지시서를 씁니다"
  ↓
[FormType 정의]
FormType: "작업지시서"
  ↓
[Form 구조 설계]
Form: 작업지시서 양식
  [Main Section] 지시 정보
    - 지시번호: Field
    - 제품명: Field (FK)
  [Child Section] 자재 내역 (1:N)
    - 원료: Field
    - 수량: Field
  ↓
[Record 사상]
Entity: work_orders
  - id (PK)
  - product_id (FK)

Entity: work_order_materials (1:N)
  - id (PK)
  - order_id (FK)
  - material_id (FK)
  - quantity
  ↓
[구현]
Database: PostgreSQL 테이블
API: /api/v1/work-orders
Frontend: React Form Component
```

---

## 체크리스트

### FormType 정의 체크리스트

```
FormType 정의 시:
□ 현업이 실제로 사용하는 이름인가?
□ 접미사(-서/-지/-표/-록)가 명확한가?
□ 업무 흐름에서 역할이 명확한가?
□ 다른 FormType과 중복되지 않는가?
```

---

### Form 설계 체크리스트

```
Form 구조 설계 시:
□ Section 유형이 명확한가? (Main/Child/Reference/Attachment)
□ Main Section이 1개 이상 존재하는가?
□ Child Section의 1:N 관계가 명확한가?
□ Reference Section은 실시간 동기화가 필요한가?
□ Attachment Section은 과거 고정이 필요한가?
□ 각 Field의 유형이 적절한가?
□ Reference Section의 Selection Field = FK인가?
```

---

### 관계 정의 체크리스트

```
Section 관계 정의 시:
□ Reference vs Attachment 선택이 적절한가?
  - 실시간 동기화 필요 → Reference
  - 과거 시점 고정 필요 → Attachment
□ Child Section의 부모가 명확한가?
□ FK 방향이 올바른가?
□ M:N 관계가 필요한가? (Checklist Field)
```

---

### Record 사상 체크리스트

```
Record 사상 시:
□ Section → Entity 사상이 명확한가?
□ Field → Attribute 사상이 명확한가?
□ FK 제약조건이 정의되었는가?
□ 1:N 관계의 parent_id FK가 있는가?
□ 테이블명/컬럼명이 snake_case인가?
□ 필수/선택 제약조건이 정의되었는가?
```

---

## 관련 문서

**핵심 방법론**:
- [문서 구조](../02-methodology/document-structure.md) - FormType/Form/Document/Record 상세
- [문서 유형](../02-methodology/document-types.md) - 서/지/표/록 분류
- [데이터 모델링](../02-methodology/data-modeling.md) - Section → Entity 사상
- [문서 관계](../02-methodology/document-relationships.md) - Reference/Attachment Section

**실전 가이드**:
- [워크숍 진행](../04-workshop/facilitation-guide.md) - FormType 도출 워크숍
- [데이터베이스 설계](../05-realization/database-design.md) - Record 구조 설계
- [API 설계](../05-realization/api-design.md) - API 엔드포인트 설계

**사례 연구**:
- [제조 MES 사례](../06-case-studies/manufacturing-mes.md) - 실전 적용 사례

---

## 버전 이력

| 버전 | 날짜 | 변경 내용 |
|------|------|----------|
| 1.0 | 2024-01-XX | 초안 작성 (2차원 개념 체계 확립) |

---

**이 문서는 Formology 방법론의 공식 용어 표준입니다.**

모든 Formology 관련 문서와 대화는 이 표준을 따라야 합니다.
