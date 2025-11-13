# 데이터 모델 도출

## 핵심 원리

> **"Form의 Section-Field 구조를 보면 데이터 모델이 보인다"**

추상화 없이도, Form의 구조만 있으면 ERD가 자연스럽게 도출됩니다.

**개념 체계 이해**:
```
FormType (서식) → 필수 Section/Field 정의 (본질)
  ↓
Form (양식) → Section/Field 구체화 (레이아웃)
  ↓
Document (문서) → Section/Field 작성 (데이터 입력)
  ↓
Record (레코드) → Entity/Attribute 저장 (DB)
```

---

## 도출 규칙 (Dimension 2: Structural Decomposition → DB Schema)

### 기본 사상 (Mapping)

| Form 구조 | 데이터 모델 | 설명 |
|-----------|-------------|------|
| **Section** → **Entity** | 테이블 | Section이 Entity 경계 결정 |
| **Field** → **Attribute** | 컬럼 | Field가 Entity 속성이 됨 |
| **Main Section** | 메인 테이블 | 품질검사의뢰서 Main → qc_requests |
| **Child Section** | 자식 테이블 (1:N) | 검사항목 Child → qc_items |
| **Reference Section** | FK (참조키) | 참조: 계획서 → plan_id FK |
| **Attachment Section** | 파일 테이블 (1:N) | 붙임: 첨부파일 → attachments |

### Field 타입별 구현

| Field 타입 | DB 컬럼 타입 | 예시 |
|-----------|-------------|------|
| Text (Single-line) | VARCHAR | 의뢰번호 → document_no VARCHAR(50) |
| Text (Multi-line) | TEXT | 특이사항 → notes TEXT |
| Numeric | INTEGER, DECIMAL | 수량 → quantity INTEGER |
| Date/Time | DATE, TIMESTAMP | 작성일 → created_at TIMESTAMP |
| Selection (Single) | VARCHAR + FK | 제품 → product_id VARCHAR(50) FK |
| Selection (Multiple) | 1:N 테이블 | 검사항목 → qc_items |
| Boolean | BOOLEAN | 긴급여부 → is_urgent BOOLEAN |
| File | VARCHAR (경로) | 파일경로 → file_path VARCHAR(500) |

### 도출 프로세스

```
1. FormType 확인 → 필수 Section/Field 파악
  ↓
2. Form 구조 분석
   ├─ Section 식별 → Entity 경계 결정
   └─ Field 타입 파악 → Attribute 타입 결정
  ↓
3. Section 유형별 변환
   ├─ Main Section → 메인 테이블
   ├─ Child Section → 자식 테이블 (1:N)
   ├─ Reference Section → FK
   └─ Attachment Section → 파일 테이블
  ↓
4. Field → Attribute 변환
   ├─ Text → VARCHAR/TEXT
   ├─ Numeric → INTEGER/DECIMAL
   ├─ Date/Time → DATE/TIMESTAMP
   ├─ Selection → FK or 1:N
   ├─ Boolean → BOOLEAN
   └─ File → VARCHAR (경로)
  ↓
5. ERD 완성
   └─ Record (레코드) 구조 확정
```

---

## 실전 예시: 품질검사의뢰서

### FormType (서식) 정의

```
품질검사의뢰서 FormType:

필수 Section (본질):
1. Main Section - 의뢰 기본정보
   필수 Field: 의뢰번호, 작성자, 제품, 수량
2. Reference Section - 생산계획 참조
   필수 Field: 생산계획서 ID
```

### Form (양식) 구조

```
┌─────────────────────────┐
│ 품질검사의뢰서 (Form)    │
├─────────────────────────┤
│ [Main Section]          │
│ 의뢰번호: QC-REQ-001    │  ← Field: Text (auto)
│ 작성자: 김철수          │  ← Field: Selection (User)
├─────────────────────────┤
│ 제품명: ABC-모터        │  ← Field: Selection (Product)
│ LOT번호: L20241104-001  │  ← Field: Selection (Lot)
│ 수량: 500 EA            │  ← Field: Numeric
├─────────────────────────┤
│ [Child Section]         │
│ 검사항목:               │  ← Multiple Selection
│ ☑ 외관 검사            │  ← Child Field
│ ☑ 치수 측정            │  ← Child Field
│ □ 성능 시험            │  ← Child Field
├─────────────────────────┤
│ [Reference Section]     │
│ 참조: 생산계획서 PLAN-045│ ← FK (living link)
├─────────────────────────┤
│ [Attachment Section]    │
│ 붙임: 📎 도면.pdf      │  ← File (snapshot)
└─────────────────────────┘
```

### 도출되는 ERD (Record 구조)

```sql
-- Main Section → 메인 Entity (qc_requests)
CREATE TABLE qc_requests (
  id VARCHAR(50) PRIMARY KEY,

  -- Main Section Fields → Attributes
  document_no VARCHAR(50),       -- Field: Text (auto)
  created_by VARCHAR(50),        -- Field: Selection → FK
  product_id VARCHAR(50),        -- Field: Selection → FK
  lot_id VARCHAR(50),            -- Field: Selection → FK
  quantity INTEGER,              -- Field: Numeric

  -- Reference Section → FK (living link)
  production_plan_id VARCHAR(50), -- Reference Section → FK

  -- 생명주기 Fields
  created_at TIMESTAMP,
  updated_at TIMESTAMP,
  status VARCHAR(20),

  -- 제약조건
  FOREIGN KEY (created_by) REFERENCES users(id),
  FOREIGN KEY (product_id) REFERENCES products(id),
  FOREIGN KEY (lot_id) REFERENCES lots(id),
  FOREIGN KEY (production_plan_id) REFERENCES production_plans(id)
);

-- Child Section → 자식 Entity (qc_request_items)
CREATE TABLE qc_request_items (
  id VARCHAR(50) PRIMARY KEY,
  request_id VARCHAR(50) NOT NULL, -- 부모 참조

  -- Child Section Fields → Attributes
  item_name VARCHAR(100),          -- Field: Text
  is_checked BOOLEAN DEFAULT FALSE, -- Field: Boolean

  FOREIGN KEY (request_id) REFERENCES qc_requests(id) ON DELETE CASCADE
);

-- Attachment Section → 파일 Entity (qc_request_attachments)
CREATE TABLE qc_request_attachments (
  id VARCHAR(50) PRIMARY KEY,
  request_id VARCHAR(50) NOT NULL, -- 부모 참조

  -- Attachment Section Fields → Attributes
  file_name VARCHAR(200),          -- Field: Text
  file_path VARCHAR(500),          -- Field: File Path
  file_size BIGINT,                -- Field: Numeric
  uploaded_at TIMESTAMP,           -- Field: Date/Time

  FOREIGN KEY (request_id) REFERENCES qc_requests(id) ON DELETE CASCADE
);
```

**Section → Entity 사상 결과**:
- **Main Section** → `qc_requests` (메인 테이블)
- **Child Section** → `qc_request_items` (1:N 자식 테이블)
- **Reference Section** → `production_plan_id` FK (살아있는 링크)
- **Attachment Section** → `qc_request_attachments` (1:N 파일 테이블, 스냅샷)

---

## 패턴 인식 가이드 (Section/Field → Entity/Attribute)

### 패턴 1: Main Section → 메인 Entity

```
Form 구조:
  [Main Section] 품질검사의뢰서
    ├─ Field: 의뢰번호
    ├─ Field: 작성자
    └─ Field: 수량
  ↓
Record 구조:
  CREATE TABLE qc_requests (
    id PK,
    document_no VARCHAR,
    created_by VARCHAR FK,
    quantity INTEGER
  )
```

### 패턴 2: Selection Field → FK

```
Form 구조:
  Field: 제품명 [선택 ▼]
  ↓
Record 구조:
  product_id VARCHAR(50)
  FOREIGN KEY (product_id) REFERENCES products(id)
```

### 패턴 3: Text/Numeric Field → 일반 Attribute

```
Form 구조:
  Field: 수량 [숫자 입력]
  ↓
Record 구조:
  quantity INTEGER
```

### 패턴 4: Child Section → 자식 Entity (1:N)

```
Form 구조:
  [Child Section] 검사항목
    ├─ Field: 항목명
    └─ Field: 선택여부
  ↓
Record 구조:
  CREATE TABLE qc_request_items (
    id PK,
    request_id FK,
    item_name VARCHAR,
    is_checked BOOLEAN
  )
```

### 패턴 5: Reference Section → FK (살아있는 링크)

```
Form 구조:
  [Reference Section]
    └─ Field: 생산계획서 참조
  ↓
Record 구조:
  production_plan_id VARCHAR(50)
  FOREIGN KEY → production_plans(id)

  -- 원본 수정 시 자동 반영
```

### 패턴 6: Attachment Section → 파일 Entity (스냅샷)

```
Form 구조:
  [Attachment Section]
    ├─ Field: 파일명
    └─ Field: 파일경로
  ↓
Record 구조:
  CREATE TABLE qc_request_attachments (
    id PK,
    request_id FK,
    file_name VARCHAR,
    file_path VARCHAR
  )

  -- 원본 변경과 무관, 복제본 유지
```

---

## 관계 타입 도출

### Reference Section → FK (살아있는 링크)

```
Form의 Reference Section
  → Record: FK, 최신 정보 자동 반영
  → JOIN으로 실시간 조회

예: qc_requests.production_plan_id FK
   → production_plans.id (최신 버전)

   production_plans의 수정이 qc_requests 조회에 즉시 반영됨
```

### Document 간 부모-자식 관계

```
Document 흐름상 생성 관계
  → Record: FK, 생성 시점 고정

예: 검사의뢰서 Document → 검사기록지 Document 생성
   inspection_records.request_id FK
   → qc_requests.id
```

### Attachment Section → 파일 Entity (스냅샷)

```
Form의 Attachment Section
  → Record: 1:N 파일 테이블, 복제본 유지

예: qc_request_attachments.request_id FK
   → qc_requests.id

   원본 파일 변경과 무관, 당시 버전 보존
```

---

## 정규화 자동 적용

### 1NF: Section 분리로 자동 달성

```
Child Section 발견 (복수 항목)
  → 자동으로 별도 Entity로 분리
  → 1NF (원자성) 자동 달성

예: [Child Section] 검사항목 (여러개)
   → qc_request_items 테이블 (1:N)
```

### 2NF: Selection Field로 자연스럽게

```
Selection Field 발견
  → FK로 마스터 Entity 참조
  → 중복 제거
  → 2NF (부분 종속 제거) 자동 달성

예: Field: 제품 [선택]
   → product_id FK → products(id)
```

### 3NF: Reference Section으로 암묵적 적용

```
Reference Section 발견
  → FK로 다른 Entity 연결
  → 이행 종속 제거
  → 3NF 자동 달성

예: [Reference Section] 생산계획서 참조
   → production_plan_id FK → production_plans(id)
```

**핵심**: Section/Field 구조가 명확하면 정규화는 자연스럽게 따라옵니다.

---

## 전통 방식 vs Formology

### 전통 데이터 모델링

```
1. 업무 인터뷰 (1주)
2. Entity 추상화 (1주)
3. 속성 정의 (1주)
4. 명시적 정규화 (1주)
5. ERD 완성 및 현업 검증 (1주)

소요: 5주
현업 이해: 40%
과정: Entity → Attribute → 관계 → 검증
```

### Formology 방식

```
1. FormType 정의 (30분)
   - 필수 Section/Field 파악
2. Form 구조 스케치 (30분)
   - Section/Field 구체화
3. Section → Entity 사상 (30분)
   - Main/Child/Reference/Attachment 식별
4. Field → Attribute 사상 (30분)
   - 타입별 변환 규칙 적용
5. ERD 도출 및 현업 검증 (30분)

소요: 2시간 30분
현업 이해: 95%
과정: Form → Section/Field → Entity/Attribute → Record
```

**핵심 차이**:
- 전통: 추상화부터 시작 (Top-down)
- Formology: 구체적 Form부터 시작 (Bottom-up)

---

## 검증 방법

### 현업 검증 (Form 기반)

```
개발자: "품질검사의뢰서 Form에 제품 정보 Section 있죠?"
현업: "네, 제품명이랑 LOT번호 Field요"

개발자: "제품은 Selection Field죠? [선택 ▼]"
현업: "네, 목록에서 선택해요"

개발자: "그럼 products Entity가 별도로 필요하겠네요?"
현업: "당연하죠!"

개발자: "검사항목은 Child Section이고요?"
현업: "네, 여러 개 선택할 수 있어요"

→ Form/Section/Field 용어로 자연스러운 검증 완료
```

### ERD 역검증 (Record → Form)

```
Record 구조를 Form 구조로 역변환:
1. 테이블 → Section
   qc_requests → [Main Section]
   qc_items → [Child Section]
2. FK → Selection Field 또는 Reference Section
   product_id FK → Field: 제품 [선택]
   plan_id FK → [Reference Section] 생산계획서
3. 일반 컬럼 → Field
   quantity INTEGER → Field: 수량 [숫자]
4. 1:N 테이블 → Child Section 또는 Attachment Section
   qc_items → [Child Section] 검사항목
   attachments → [Attachment Section] 첨부파일

현업: "맞아요, 이게 우리 Form이에요!"
```

---

## 실전 워크플로우

### 1. FormType 정의 (30분)

```
현업과 함께:
- 필수 Section/Field 정의 (본질 요소)
- "이것이 없으면 이 서식이 아니다"
```

### 2. Form 구조 스케치 (30분)

```
현업과 함께:
- Section 구조 그리기 (Main/Child/Reference/Attachment)
- Field 타입 정의 (Text/Numeric/Selection/Boolean/Date/File)
- 입력 방식 결정 ([선택 ▼], [직접 입력], [체크박스])
```

### 3. Section → Entity 사상 (15분)

```
Form 구조를 보며:
- Main Section → 메인 테이블 식별
- Child Section → 자식 테이블 (1:N) 식별
- Reference Section → FK 필요 식별
- Attachment Section → 파일 테이블 식별
```

### 4. Field → Attribute 사상 (15분)

```
Field 타입별 변환:
- Text → VARCHAR/TEXT
- Numeric → INTEGER/DECIMAL
- Date/Time → DATE/TIMESTAMP
- Selection (Single) → VARCHAR + FK
- Selection (Multiple) → 1:N 테이블
- Boolean → BOOLEAN
- File → VARCHAR (경로)
```

### 5. ERD 작성 (30분)

```
사상 규칙 적용:
- CREATE TABLE 문 작성
- FK 관계 정의
- 제약조건 추가
- 인덱스 설계
```

### 6. 현업 검증 (15분)

```
Record → Form 역변환으로 검증:
"이 테이블이 이 Section 맞죠?"
"이 FK가 이 Field 맞죠?"
"빠진 Section/Field 없나요?"
```

---

## 고급 패턴

### 문서 버전 관리

```
양식: 작업표준서 v2.1 (개정일: ...)
  ↓
모델: CREATE TABLE work_standards (
       id PK,
       version VARCHAR(10),
       revision_date DATE,
       supersedes_id FK  -- 이전 버전
     )
```

### 승인 워크플로우

```
양식: 승인: 작성자 → 검토자 → 승인자
  ↓
모델: CREATE TABLE approvals (
       id PK,
       document_id FK,
       approver_id FK,
       approval_level INT,
       approved_at TIMESTAMP
     )
```

### 문서 상태

```
양식: 상태 [작성중/검토중/승인완료]
  ↓
모델: status ENUM('draft', 'in_review', 'approved')
```

---

## 핵심 정리

**사상 규칙 (Form → Record)**:
1. **Section → Entity**:
   - Main Section → 메인 테이블
   - Child Section → 자식 테이블 (1:N)
   - Reference Section → FK
   - Attachment Section → 파일 테이블
2. **Field → Attribute**:
   - Text → VARCHAR/TEXT
   - Numeric → INTEGER/DECIMAL
   - Selection → FK 또는 1:N
   - Date/Time → DATE/TIMESTAMP
   - Boolean → BOOLEAN
   - File → VARCHAR (경로)

**장점**:
- **추상화 불필요**: Form 구조에서 직접 도출
- **정규화 자동 적용**: Section/Field 구조가 정규형 보장
- **현업 즉시 이해**: Form ↔ Record 양방향 변환 가능
- **검증 간단**: Section/Field 용어로 소통

**체크리스트**:
- [ ] 모든 Section이 Entity로 사상되었는가?
- [ ] Main/Child/Reference/Attachment Section 구분이 명확한가?
- [ ] 모든 Field가 Attribute로 변환되었는가?
- [ ] Selection Field가 FK로 올바르게 매핑되었는가?
- [ ] Child Section이 1:N 테이블로 분리되었는가?
- [ ] Reference/Attachment Section이 올바르게 구현되었는가?
- [ ] 현업이 Record → Form 역변환을 이해하는가?

---

## 더 읽을거리

### 개념 이해
- **[문서 구조론](document-structure.md)** - FormType→Form→Document→Record 흐름, Section-Field 구조
- **[문서 관계론](document-relationships.md)** - Reference/Attachment Section 구현 상세
- **[문서 종류](document-types.md)** - 서/지/표/록 분류와 포멀리티

### 구현 원리
- **[개념적 사상 원리](../05-realization/conceptual-mapping.md)** - Form → Record 구현 변환
- **[인터페이스 설계 이론](../05-realization/interface-design-theory.md)** - Form → UI 구현

### 실전 적용
- **[워크숍 진행 가이드](../04-workshop/facilitation-guide.md)** - FormType/Form 도출 실습
- **[사례 연구](../06-case-studies/)** - 실제 Section→Entity 사상 사례

---

**핵심**: Form의 Section-Field 구조에 모든 데이터 모델이 있다. 그저 사상 규칙을 적용하면 된다.
