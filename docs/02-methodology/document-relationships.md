# 문서 관계론

## 개요

Document는 독립적으로 존재하지 않습니다. 업무 흐름 속에서 다른 Document들과 **관계**를 맺으며 의미를 형성합니다.

> **핵심**: Form의 Section 구조가 Document 간 관계를 정의하며, 이는 Record 간 관계로 구현됩니다.

**개념 체계**:
```
FormType 정의 → Section 유형 정의 (Reference/Attachment)
  ↓
Form 구조 → Section 배치 및 Field 정의
  ↓
Document 작성 → Section 기반 관계 형성
  ↓
Record 저장 → FK 또는 복제 테이블로 구현
```

---

## Section 기반 관계의 2가지 유형

### 관계 분류

```
Section 유형 (Form 구조)
├─ Reference Section → FK (Foreign Key)
│   - 살아있는 링크
│   - 원본 수정 시 자동 반영
│   - 실시간 동기화
│
└─ Attachment Section → 복제 테이블 또는 파일
    - 스냅샷 (과거 시점 고정)
    - 원본 변경과 무관
    - 증빙 보존
```

### 핵심 차이점

| 구분 | Reference Section | Attachment Section |
|------|-------------------|-------------------|
| **시점** | 현재 (living) | 과거 (frozen) |
| **동기화** | O | X |
| **원본 변경 반영** | O | X |
| **독립성** | 낮음 | 높음 |
| **DB 구현** | FK | 복제 테이블 or 파일 |
| **용도** | 실시간 조회 | 증빙 보존 |

**중요**: Snapshot은 Attachment Section의 특수한 형태로, 문서 이력 관리에 사용됩니다.

---

## 1. Reference Section (참조 섹션)

### 정의

**Reference Section**은 Form 구조에서 다른 Document나 마스터 데이터를 **현재 시점**에서 가리키는 Section입니다.

**구조적 이해**:
```
Form: 품질검사의뢰서
├─ [Main Section] 기본 정보
│   ├─ Field: 의뢰번호
│   └─ Field: 작성자
│
└─ [Reference Section] 생산 정보 참조
    └─ Field: 생산계획서 ID (FK)
        ↓ (살아있는 링크)
      [생산계획서 Document/Record]
```

**Field 레벨 참조**:
```
Form: 품질검사의뢰서
[Main Section]
  └─ Field: 제품명 [선택 ▼]  ← Selection Field → FK
       ↓
     [제품 마스터 Record]
       - ABC-100
       - DEF-200
       - GHI-300
```

### 특징

**1. 실시간 동기화**
```
시나리오:
1. 의뢰서 작성 시 제품 "ABC-100" 선택
2. 나중에 제품 마스터에서 "ABC-100" 이름 변경
   "ABC-100" → "ABC-100 (개선형)"
3. 의뢰서 조회 시 자동으로 변경된 이름 표시
```

**2. 데이터 일관성**
```
하나의 원천 (Single Source of Truth)
제품 정보는 제품 마스터에만 존재
→ 모든 문서는 제품 마스터를 참조
→ 제품 정보 변경 시 자동 반영
```

**3. 저장 효율**
```
❌ 참조 없이:
의뢰서1: "ABC-100", "샤프트", "10kg"
의뢰서2: "ABC-100", "샤프트", "10kg"
의뢰서3: "ABC-100", "샤프트", "10kg"
→ 데이터 중복, 불일치 위험

✅ 참조 사용:
의뢰서1: 제품ID = P001
의뢰서2: 제품ID = P001
의뢰서3: 제품ID = P001
→ 제품 마스터: P001 = "ABC-100", "샤프트", "10kg"
→ 데이터 일관성, 저장 효율
```

### Reference Section의 종류

**1. Field 레벨 참조 (가장 일반적)**
```
Form: 품질검사의뢰서
[Main Section]
  └─ Field: 제품명 [Selection] → FK to 제품 마스터
  └─ Field: 담당자 [Selection] → FK to 직원 마스터
  └─ Field: 고객 [Selection] → FK to 고객 마스터
```

**Record 변환**:
```sql
CREATE TABLE qc_requests (
  id INT PRIMARY KEY,
  product_id INT,  -- FK to products
  employee_id INT, -- FK to employees
  customer_id INT, -- FK to customers
  FOREIGN KEY (product_id) REFERENCES products(id),
  FOREIGN KEY (employee_id) REFERENCES employees(id),
  FOREIGN KEY (customer_id) REFERENCES customers(id)
);
```

**2. Section 레벨 참조 (복합 참조)**
```
Form: 품질검사보고서
├─ [Main Section] 검사 정보
│   └─ Field: 검사번호
│
└─ [Reference Section] 의뢰 정보
    └─ Field: 의뢰서 ID (FK)
        ↓ (살아있는 링크)
      [품질검사의뢰서 Document]
```

**Record 변환**:
```sql
CREATE TABLE qc_reports (
  id INT PRIMARY KEY,
  inspection_no VARCHAR(50),
  request_id INT, -- FK to qc_requests
  FOREIGN KEY (request_id) REFERENCES qc_requests(id)
);
```

### Reference Section 표현 방법

**Form (양식)에서**:
```
Form: 품질검사의뢰서
┌─────────────────────────┐
│ [Main Section]          │
│ 제품명: [선택 ▼]       │ ← Field FK
│   ABC-100 (샤프트)     │
│   DEF-200 (베어링)     │
│   GHI-300 (기어)       │
│                         │
│ LOT번호: [입력]        │
│ 수량: [숫자]           │
└─────────────────────────┘
```

**Document (작성된 문서)에서**:
```
품질검사의뢰서 #QC-2024-001
- 제품명: ABC-100 (샤프트, 10kg) ← 제품 마스터에서 실시간 조회
- LOT번호: L20240115-01
- 수량: 100개
```

**Record 관계도**:
```
[products (제품 마스터)]
         ↑ FK
         │
[qc_requests (의뢰서)]
```

### 참조 끊김 처리

**문제 상황**:
```
의뢰서가 제품 "ABC-100" 참조
→ 제품 마스터에서 "ABC-100" 삭제
→ 의뢰서 조회 시 오류?
```

**해결 방법**:

**옵션 1: 삭제 방지**
```
제품 삭제 시도
→ "이 제품을 참조하는 문서가 5건 있습니다"
→ 삭제 불가
```

**옵션 2: 소프트 삭제**
```
제품 삭제 시
→ deleted = true 플래그
→ 신규 문서에는 표시 안 됨
→ 기존 문서 조회 가능
```

**옵션 3: 스냅샷 전환**
```
참조 끊김 감지
→ 자동으로 마지막 상태 복사
→ 참조 → 스냅샷으로 전환
```

---

## 2. Attachment Section (붙임 섹션)

### 정의

**Attachment Section**은 Form 구조에서 다른 Document의 **과거 시점 스냅샷**을 복사하여 보존하는 Section입니다.

**구조적 이해**:
```
Form: 품질검사보고서
├─ [Main Section] 검사 정보
│   ├─ Field: 보고서번호
│   └─ Field: 검사일자
│
├─ [Reference Section] 의뢰 정보 (FK)
│   └─ Field: 의뢰서 ID → [품질검사의뢰서]
│
└─ [Attachment Section] 첨부 정보 (스냅샷)
    ├─ 문서 첨부 (복제 테이블)
    │   └─ 검사기준서 v2.1 스냅샷
    │
    └─ 파일 첨부 (파일 저장)
        ├─ 불량사진.jpg
        ├─ 측정데이터.xlsx
        └─ 분석보고서.pdf
```

### 특징

**1. 독립성**
```
원본 변경되어도 붙임은 변경되지 않음

시나리오:
1. 검사보고서에 검사기준서 붙임
2. 나중에 검사기준서 내용 변경
3. 검사보고서의 붙임은 원래 버전 유지
```

**2. 불변성**
```
붙임은 첨부 시점에 고정됨
→ 이후 수정 불가
→ 증빙력 확보
```

**3. 완전성**
```
보고서만 봐도 모든 정보 파악 가능
→ 붙임 문서들이 모든 맥락 제공
→ 독립적 이해 가능
```

### Attachment Section의 종류

**1. 문서 스냅샷 (Document Snapshot Attachment)**

**Form 구조**:
```
Form: 품질검사보고서
└─ [Attachment Section] 의뢰서 스냅샷
    └─ Field: 의뢰서 스냅샷 ID → 복제 테이블
```

**Record 변환 (복제 테이블 방식)**:
```sql
-- 원본 테이블
CREATE TABLE qc_requests (
  id INT PRIMARY KEY,
  product_name VARCHAR(100),
  lot_no VARCHAR(50),
  quantity INT
);

-- 스냅샷 복제 테이블
CREATE TABLE qc_requests_snapshots (
  snapshot_id INT PRIMARY KEY,
  original_request_id INT,
  snapshot_at TIMESTAMP,
  -- 원본 필드 전체 복제
  product_name VARCHAR(100),
  lot_no VARCHAR(50),
  quantity INT
);

-- 보고서 테이블
CREATE TABLE qc_reports (
  id INT PRIMARY KEY,
  inspection_no VARCHAR(50),
  request_snapshot_id INT, -- Attachment Section → FK to snapshot table
  FOREIGN KEY (request_snapshot_id) REFERENCES qc_requests_snapshots(snapshot_id)
);
```

**2. 파일 첨부 (File Attachment)**

**Form 구조**:
```
Form: 불량보고서
└─ [Attachment Section] 증빙 파일
    └─ Field: 첨부파일 목록 [FileUpload]
```

**Record 변환 (파일 메타데이터 테이블)**:
```sql
CREATE TABLE defect_reports (
  id INT PRIMARY KEY,
  report_no VARCHAR(50)
);

CREATE TABLE report_attachments (
  id INT PRIMARY KEY,
  report_id INT,
  file_name VARCHAR(255),
  file_path VARCHAR(500),
  file_size INT,
  uploaded_at TIMESTAMP,
  FOREIGN KEY (report_id) REFERENCES defect_reports(id)
);
```

**3. 증빙 스냅샷 (Evidence Snapshot)**

**Form 구조**:
```
Form: 시정조치서
└─ [Attachment Section] 증빙 자료
    ├─ Field: 원인분석서 스냅샷 ID
    ├─ Field: 개선계획서 스냅샷 ID
    └─ Field: 검증결과서 스냅샷 ID
```

### Reference Section vs Attachment Section 선택 기준

**Reference Section을 사용하는 경우 (FK)**:
```
Form 설계 시:
□ 최신 정보가 중요한 Field
□ 마스터 데이터 참조 Field
□ 실시간 동기화 필요한 Section
□ 원본 변경 시 반영이 필요한 관계

예시 Form Field:
- 제품명 [Selection] → 제품 마스터
- 고객명 [Selection] → 고객 마스터
- 담당자 [Selection] → 직원 마스터
- 상위 문서 ID [Reference] → 선행 문서

Record 구현: Foreign Key (FK)
```

**Attachment Section을 사용하는 경우 (스냅샷/복제)**:
```
Form 설계 시:
□ 특정 시점 정보 고정이 필요한 Section
□ 증빙 자료 보관용 Section
□ 원본 변경과 무관해야 하는 관계
□ 법적/감사 목적 보존이 필요한 첨부

예시 Form Section:
- [Attachment Section] 계약서 스냅샷
- [Attachment Section] 승인서 스냅샷
- [Attachment Section] 검사기준서 v2.1
- [Attachment Section] 증빙 파일

Record 구현: 복제 테이블 또는 파일 저장
```

### 실전 예시

**시나리오: 품질검사 프로세스**

**1. 품질검사의뢰서 Form (Reference Section 사용)**
```
Form: 품질검사의뢰서
├─ [Main Section] 의뢰 정보
│   ├─ Field: 의뢰번호
│   └─ Field: 의뢰일자
│
└─ [Reference Section] 제품 정보
    └─ Field: 제품명 [Selection] → FK to 제품 마스터
```

**Document 예시**:
```
품질검사의뢰서 #QC-2024-001
- 제품명: ABC-100 (샤프트, 10kg) ← 실시간 조회
- LOT번호: L20240115-01
```

**이유**: 최신 제품 정보 필요 (사양 변경 시 자동 반영)

---

**2. 품질검사보고서 Form (Attachment Section 사용)**
```
Form: 품질검사보고서
├─ [Main Section] 검사 정보
│   ├─ Field: 보고서번호
│   └─ Field: 검사일자
│
├─ [Reference Section] 의뢰 정보 (FK)
│   └─ Field: 의뢰서 ID → [품질검사의뢰서]
│
└─ [Attachment Section] 증빙 자료
    ├─ Field: 검사기준서 스냅샷 ID → 복제 테이블
    └─ Field: 불량사진 [FileUpload] → 파일 저장
```

**Document 예시**:
```
품질검사보고서 #QC-REP-2024-001
- 의뢰서: QC-2024-001 (FK - 살아있는 링크)
- [첨부] 검사기준서 v2.1 (스냅샷 - 검사 당시 기준)
- [첨부] 불량사진_01.jpg (파일)
```

**이유**:
- 의뢰서 정보는 실시간 조회 (Reference Section → FK)
- 검사기준서는 당시 버전 고정 필요 (Attachment Section → 스냅샷)
- 증빙 파일은 법적 보존 (Attachment Section → 파일)

---

## 3. 스냅샷 (Snapshot) - Attachment Section의 특수 형태

### 정의

**스냅샷**은 Attachment Section의 특수한 형태로, Document의 **특정 버전 상태**를 복사하여 이력 관리에 활용하는 방식입니다.

> **핵심**: 스냅샷은 독립적인 관계 유형이 아니라, Attachment Section이 버전 관리 목적으로 사용될 때의 명칭입니다.

**Form 구조 관점**:
```
Form: 품질검사보고서
└─ [Attachment Section] 의뢰서 버전 기록
    └─ Field: 의뢰서 스냅샷 ID (v3.0) → 복제 테이블

원본 Document 버전 이력:
품질검사의뢰서 #QC-2024-001
  v1.0 (작성중) → v2.0 (제출됨) → v3.0 (승인됨)

보고서가 참조하는 스냅샷:
  → v3.0 스냅샷 (승인된 시점의 정확한 상태)
```

### 특징

**1. 시점 고정**
```
특정 버전의 문서 상태를 복사
→ 이후 원본 변경되어도 영향 없음
```

**2. 이력 추적**
```
스냅샷으로 과거 상태 재현 가능
→ "그때 문서가 어땠는지" 정확히 알 수 있음
```

**3. 버전 관리**
```
의뢰서 v1.0 → 보고서 A
의뢰서 v2.0 → 보고서 B
의뢰서 v3.0 → 보고서 C

각 보고서는 해당 버전의 스냅샷 보유
```

### Reference Section vs Attachment Section (스냅샷 포함)

**비교 시나리오**: 품질검사의뢰서 정보를 검사보고서에서 사용

**Reference Section (FK) - 실시간 연결**:
```
Form: 검사보고서
└─ [Reference Section]
    └─ Field: 의뢰서 ID (FK) → 의뢰서 원본

Record 구현: FOREIGN KEY (request_id) REFERENCES qc_requests(id)

장점: 항상 최신 정보 조회
단점: 원본 수정 시 과거 데이터도 영향
적합: 진행 중인 업무, 마스터 데이터 조회
```

**Attachment Section (스냅샷) - 버전 고정**:
```
Form: 검사보고서
└─ [Attachment Section]
    └─ Field: 의뢰서 스냅샷 ID (v3.0) → 복제 테이블

Record 구현:
  FK (snapshot_id) REFERENCES qc_requests_snapshots(snapshot_id)
  스냅샷 테이블이 원본 필드 전체 복제

장점: 승인 당시 정확한 상태 보존, 이력 관리
단점: 저장 공간 필요, 복제 테이블 관리
적합: 완료된 업무, 버전 관리, 법적 증빙
```

**Attachment Section (파일) - 단순 첨부**:
```
Form: 검사보고서
└─ [Attachment Section]
    └─ Field: 의뢰서 PDF [FileUpload] → 파일 저장

Record 구현: 파일 경로 또는 BLOB 저장

장점: 완전히 독립적, 원본 시스템 무관
단점: 구조적 관계 없음, 쿼리 불가
적합: 외부 시스템 문서, 단순 증빙 첨부
```

### 스냅샷 활용 패턴

**패턴 1: 승인 시 스냅샷**
```
문서 승인 시 → 현재 상태 스냅샷 저장
→ 승인 후 원본 수정되어도
→ 승인 당시 상태 보존
```

**패턴 2: 주기적 스냅샷**
```
매월 말일 → 모든 진행 중 문서 스냅샷
→ 월별 상태 추적 가능
```

**패턴 3: 이벤트 기반 스냅샷**
```
중요 이벤트 발생 시 → 스냅샷
- 계약 체결 시
- 감사 시작 시
- 분쟁 발생 시
```

---

## 문서 관계 설계 원칙

### 원칙 1: Form 구조로 관계 정의

```
Form 설계 시 질문: "이 Section의 목적은?"

실시간 정보 조회 → Reference Section (FK)
과거 시점 고정 → Attachment Section (스냅샷)
증빙 파일 첨부 → Attachment Section (파일)
```

### 원칙 2: 업무 흐름을 Section으로 반영

```
FormType: 작업지시서
  [Main Section] 지시 정보
  [Reference Section] 제품 정보 (FK - 진행 중)

FormType: 작업일지
  [Main Section] 작업 내역
  [Reference Section] 지시서 (FK - 기록 중)

FormType: 작업완료보고서
  [Main Section] 완료 정보
  [Attachment Section] 지시서 스냅샷 (완료 시점 고정)
  [Attachment Section] 증빙 사진 (파일)
```

### 원칙 3: 마스터 데이터는 Reference Section (FK)

```
Form 설계 원칙:
마스터 데이터 조회 Field → Reference Section에 배치 → FK 구현

예시:
- 제품명 [Selection] → FK to products
- 고객명 [Selection] → FK to customers
- 담당자 [Selection] → FK to employees
- 설비 [Selection] → FK to equipments

이유:
- 정보 변경 시 일괄 반영
- 중복 데이터 최소화
- 데이터 일관성 유지
```

### 원칙 4: 법적 증빙은 Attachment Section (스냅샷)

```
Form 설계 원칙:
법적 증빙 Section → Attachment Section 배치 → 복제 테이블 구현

예시:
- [Attachment Section] 계약서 v1.2 스냅샷
- [Attachment Section] 승인서 스냅샷
- [Attachment Section] 검사성적서 스냅샷

이유:
- 원본 변조 방지 (과거 시점 고정)
- 당시 상태 정확히 증명
- 분쟁 시 증거 확보
```

---

## 실전 가이드

### 워크숍에서 Form 관계 파악하기

**질문 1: "이 정보를 어디서 선택하나요?"**
```
현업: "제품 목록에서 선택해요"

Form 설계:
→ [Reference Section] 또는 Field에 Selection 타입
→ Record: FK to products

예시:
Form: 검사의뢰서
  [Main Section]
    └─ Field: 제품명 [Selection] → FK
```

**질문 2: "이 문서를 첨부하나요?"**
```
현업: "검사의뢰서를 붙여요"

Form 설계:
→ [Attachment Section] 정의
→ Record: 복제 테이블 또는 파일

예시:
Form: 검사보고서
  [Attachment Section] 의뢰서 첨부
    └─ Field: 의뢰서 스냅샷 ID
```

**질문 3: "나중에 원본이 변경되면?"**
```
현업: "변경되어도 보고서는 그대로여야 해요"

Form 설계:
→ [Attachment Section] (시점 고정 필요)
→ Record: 스냅샷 복제 테이블

현업: "최신 정보가 보여야 해요"
→ [Reference Section] (실시간 조회)
→ Record: FK
```

### Form 관계 다이어그램 작성

**표기법 (Form-Record 매핑)**:
```
[FormA]
  [Reference Section] ──FK──→ [FormB]

[FormA]
  [Attachment Section] ····스냅샷··→ [FormB]

[FormA]
  [Attachment Section] ····파일···→ [파일]
```

**예시**:
```
[제품 마스터]
       ↑ FK (Reference Section)
       │
[품질검사의뢰서]
       ↓ 스냅샷 (Attachment Section)
[품질검사보고서]
       ↓ FK (Reference Section)
[품질관리대장]
```

### Form 관계 검증 체크리스트

```
Form 설계 검증:
□ Reference Section이 필요한 마스터 데이터가 식별되었는가?
□ Attachment Section이 필요한 증빙 자료가 명확한가?
□ 각 Section의 Field 타입이 정의되었는가? (Selection/Text/FileUpload)
□ Reference Section의 FK 끊김 처리 방안이 있는가?
□ Attachment Section의 스냅샷 시점이 정의되었는가?
□ 파일 첨부 Section의 크기 제한이 정의되었는가?
□ 스냅샷 보관 기간이 정해졌는가?

Record 구현 검증:
□ Reference Section → FK 매핑이 명확한가?
□ Attachment Section → 복제 테이블 또는 파일 저장 전략이 명확한가?
□ 각 Section → Entity 변환 규칙이 일관되는가?
```

---

## 고급 패턴

### 패턴 1: Reference Section + Attachment Section 혼용

```
Form: 작업일지
├─ [Main Section] 작업 정보
│
├─ [Reference Section] 지시 정보 (진행 중)
│   └─ Field: 작업지시서 ID (FK)
│       → 실시간 최신 지시사항 반영
│
└─ [Attachment Section] 완료 증빙 (완료 시)
    └─ Field: 지시서 스냅샷 ID
        → 완료 시점 지시 내용 고정

Record 변환:
CREATE TABLE work_logs (
  id INT PRIMARY KEY,
  work_order_id INT,  -- Reference Section → FK (진행 중)
  work_order_snapshot_id INT,  -- Attachment Section → 스냅샷 (완료 시)
  FOREIGN KEY (work_order_id) REFERENCES work_orders(id),
  FOREIGN KEY (work_order_snapshot_id) REFERENCES work_orders_snapshots(snapshot_id)
);
```

### 패턴 2: 다중 Reference Section

```
Form: 품질검사보고서
├─ [Main Section] 검사 정보
│
└─ [Reference Section] 다중 참조
    ├─ Field: 제품 [Selection] → FK to products
    ├─ Field: 고객 [Selection] → FK to customers
    ├─ Field: 검사자 [Selection] → FK to employees
    └─ Field: 검사기준서 ID → FK to inspection_standards

Record 변환:
CREATE TABLE qc_reports (
  id INT PRIMARY KEY,
  product_id INT,
  customer_id INT,
  inspector_id INT,
  standard_id INT,
  FOREIGN KEY (product_id) REFERENCES products(id),
  FOREIGN KEY (customer_id) REFERENCES customers(id),
  FOREIGN KEY (inspector_id) REFERENCES employees(id),
  FOREIGN KEY (standard_id) REFERENCES inspection_standards(id)
);
```

### 패턴 3: 계층적 Attachment Section

```
Form: 최종보고서
├─ [Main Section] 보고서 정보
│
└─ [Attachment Section] 계층적 증빙
    ├─ Field: 상세보고서 스냅샷 ID
    │   → 복제 테이블 (상세보고서 전체 복제)
    │       └─ 상세보고서의 Attachment Section:
    │           └─ Field: 데이터시트 파일
    │               └─ 데이터시트의 Attachment:
    │                   └─ Field: 측정기록 파일
    │
    └─ Field: 증빙사진 [FileUpload]

Record 변환: 재귀적 스냅샷 또는 중첩 파일 참조
```

---

## 구현 매핑 참고

**Form Section → Record 구현 매핑** (상세는 구현 가이드 참조):

### Reference Section → Foreign Key
```
Form 설계:
[Reference Section] 제품 정보
  └─ Field: 제품명 [Selection]

Record 구현:
  → Database: FOREIGN KEY (product_id) REFERENCES products(id)
  → API: GET /products, POST /qc-requests { product_id: 123 }
  → Frontend: <select> 또는 autocomplete 컴포넌트
```

### Attachment Section (스냅샷) → 복제 테이블
```
Form 설계:
[Attachment Section] 의뢰서 스냅샷
  └─ Field: 의뢰서 스냅샷 ID

Record 구현:
  → Database:
      CREATE TABLE qc_requests_snapshots (...);
      FOREIGN KEY (snapshot_id) REFERENCES qc_requests_snapshots(snapshot_id)
  → API:
      POST /qc-requests/{id}/snapshot → 스냅샷 생성
      GET /qc-requests/snapshots/{snapshot_id} → 스냅샷 조회
  → Frontend: 버전 이력 조회 UI, 타임라인 컴포넌트
```

### Attachment Section (파일) → 파일 저장
```
Form 설계:
[Attachment Section] 증빙 파일
  └─ Field: 첨부파일 [FileUpload]

Record 구현:
  → Database:
      파일 메타데이터 테이블 (file_name, file_path, file_size)
  → API:
      POST /attachments (multipart/form-data)
      GET /attachments/{id}/download
  → Frontend: <input type="file"> 또는 드래그앤드롭 컴포넌트
```

---

## 다음 단계

Form 기반 문서 관계를 이해했다면:

- **[문서 구조론](document-structure.md)**: FormType→Form→Document→Record 개념 체계
- **[데이터 모델 도출](data-modeling.md)**: Section→Entity, Field→Attribute 사상 규칙
- **[문서의 종류](document-types.md)**: 서/지/표/록 분류 체계
- **[워크숍 진행 가이드](../04-workshop/facilitation-guide.md)**: Form 설계 실전 적용

---

**핵심 기억**:
- **Reference Section** (FK) → 실시간, 살아있는 링크
- **Attachment Section** (스냅샷/파일) → 과거 시점 고정, 증빙 보존
- **Form 구조**가 Section 유형을 정의하고, Section이 Record 관계를 결정
