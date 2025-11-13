# 데이터베이스 설계 가이드

## 개요

Formology에서 데이터베이스 설계는 **Form 구조에서 자연스럽게 도출**됩니다. 사전 설계가 아니라 **발견**입니다.

### 핵심 원칙

```
FormType → Form → Document → Record
  ↓
Section → Entity (테이블)
Field → Attribute (컬럼)
Reference Section → FK
Child Section → 1:N 관계
```

**전통 방식과의 차이**:

| 단계 | 전통 ERD 모델링 | Formology 접근 |
|------|----------------|--------------|
| 시작 | Entity 정의 | FormType 나열 |
| 분석 | 속성 도출 | Form 구조 스케치 (Section/Field) |
| 관계 | Cardinality 분석 | Section 유형 관찰 (Reference/Child) |
| 검증 | 정규화 이론 | 현업 확인 |
| 시간 | 2-4주 | 2시간 |

---

## 패턴 1: Form Section/Field → 테이블 구조

### 규칙 1: Selection Field in Reference Section = FK

**워크숍 Form 구조**:
```
FormType: "품질검사의뢰서"
↓
Form: 품질검사의뢰서 양식

┌─────────────────────────┐
│ [Main Section] 의뢰정보 │
├─────────────────────────┤
│ 제품명: [Selection ▼]  │ ← Reference Section Field
│ LOT번호: [Text]         │ ← Main Section Field
│ 수량: [Numeric]         │ ← Main Section Field
└─────────────────────────┘
```

**질문**:
```
개발자: "제품명은 어디서 선택하나요?"
현업: "제품 목록에서요"
개발자: "그 목록은 어디 있나요?"
현업: "제품 마스터에 있죠"
```

**Section → Entity 사상**:
```sql
-- 제품 마스터 (참조 엔티티)
CREATE TABLE products (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(100) NOT NULL,
  code VARCHAR(50) UNIQUE NOT NULL,
  spec TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- FormType: "품질검사의뢰서" → Main Section → qc_requests 테이블
CREATE TABLE qc_requests (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  request_number VARCHAR(20) UNIQUE NOT NULL,  -- Field: 의뢰번호
  product_id UUID NOT NULL,                     -- Reference Section Field → FK
  lot_number VARCHAR(50) NOT NULL,              -- Field: LOT번호 [Text]
  quantity INTEGER NOT NULL,                    -- Field: 수량 [Numeric]

  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  created_by UUID NOT NULL,

  FOREIGN KEY (product_id) REFERENCES products(id)
);
```

**패턴**: `Reference Section + Selection Field` → FK 관계

---

### 규칙 2: Text/Numeric/Date Fields = 컬럼

**워크숍 Form 구조**:
```
FormType: "작업일지"
↓
Form: 작업일지 양식

┌─────────────────────────┐
│ [Main Section] 작업정보 │
├─────────────────────────┤
│ 작업자: [Text]          │ ← Main Section Field
│ 시작시간: [Time]        │ ← Main Section Field
│ 종료시간: [Time]        │ ← Main Section Field
│ 특이사항: [TextArea]    │ ← Main Section Field
└─────────────────────────┘
```

**Section → Entity 사상**:
```sql
-- FormType: "작업일지" → Main Section → work_logs 테이블
CREATE TABLE work_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  worker_name VARCHAR(50) NOT NULL,      -- Field: 작업자 [Text]
  start_time TIME NOT NULL,              -- Field: 시작시간 [Time]
  end_time TIME NOT NULL,                -- Field: 종료시간 [Time]
  notes TEXT,                            -- Field: 특이사항 [TextArea]

  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**패턴**: `Main Section Field (Text/Numeric/Date)` → VARCHAR/INTEGER/DATE 컬럼

---

### 규칙 3: Child Section = 1:N 테이블

**워크숍 Form 구조**:
```
FormType: "품질검사의뢰서"
↓
Form: 품질검사의뢰서 양식

┌─────────────────────────┐
│ [Main Section] 의뢰정보 │
├─────────────────────────┤
│ 제품명: [Selection]     │
│ LOT번호: [Text]         │
├─────────────────────────┤
│ [Child Section] 검사항목│ ← 1:N 관계
│ ☑ 외관검사             │
│ ☑ 치수검사             │
│ ☐ 중량검사             │
│ ☑ 강도검사             │
└─────────────────────────┘
```

**질문**:
```
개발자: "검사항목은 몇 개까지 선택할 수 있나요?"
현업: "여러 개요. 제품마다 다르고요"
```

**Section → Entity 사상 - Child Section**:
```sql
-- 검사항목 마스터 (참조 엔티티)
CREATE TABLE inspection_types (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(50) NOT NULL,     -- 외관검사, 치수검사...
  description TEXT
);

-- FormType: "품질검사의뢰서" → Child Section → qc_request_items 테이블 (1:N)
CREATE TABLE qc_request_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  request_id UUID NOT NULL,                  -- Parent: Main Section FK
  inspection_type_id UUID NOT NULL,          -- Reference Section Field
  is_required BOOLEAN DEFAULT false,         -- Field

  FOREIGN KEY (request_id) REFERENCES qc_requests(id) ON DELETE CASCADE,
  FOREIGN KEY (inspection_type_id) REFERENCES inspection_types(id)
);
```

**패턴**: `Child Section` → 1:N 관계 테이블 (Parent FK 포함)

---

### 규칙 4: "참조:" = FK (최신 버전)

**워크숍 양식**:
```
┌─────────────────────┐
│ 작업지시서          │
├─────────────────────┤
│ 작업표준서: [참조]  │ ← "최신 버전 보기"
│ 검사기준서: [참조]  │
└─────────────────────┘
```

**질문**:
```
개발자: "작업표준서를 어떻게 찾나요?"
현업: "제품별로 있고, 최신 버전을 봐야 해요"
개발자: "구버전도 봐야 하나요?"
현업: "아니요, 무조건 최신만요"
```

**자동 도출**:
```sql
-- 작업표준서 (버전 관리)
CREATE TABLE work_standards (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  product_id UUID NOT NULL,
  version VARCHAR(20) NOT NULL,      -- v1.0, v1.1, v2.0
  content TEXT NOT NULL,
  is_latest BOOLEAN DEFAULT false,   -- 최신 버전 플래그
  effective_date DATE NOT NULL,
  supersedes_id UUID,                -- 이전 버전 참조

  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  created_by UUID NOT NULL,

  FOREIGN KEY (product_id) REFERENCES products(id),
  FOREIGN KEY (supersedes_id) REFERENCES work_standards(id),

  UNIQUE (product_id, version)
);

-- 최신 버전만 조회하는 뷰
CREATE VIEW latest_work_standards AS
SELECT * FROM work_standards
WHERE is_latest = true;

-- 작업지시서
CREATE TABLE work_orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  product_id UUID NOT NULL,
  standard_id UUID NOT NULL,         -- 작업표준서 참조

  FOREIGN KEY (standard_id) REFERENCES work_standards(id),

  -- 최신 버전 제약조건 (트리거로 검증)
  CONSTRAINT fk_latest_standard CHECK (
    standard_id IN (SELECT id FROM latest_work_standards)
  )
);
```

**패턴**: `참조:` → FK (최신 버전 관리)

---

### 규칙 5: "붙임:" = 1:N (파일)

**워크숍 양식**:
```
┌─────────────────────┐
│ 불량기록지          │
├─────────────────────┤
│ 불량 내용: [텍스트] │
│ 붙임: [파일들]      │ ← 사진 여러 장
└─────────────────────┘
```

**질문**:
```
개발자: "파일은 몇 개까지 첨부할 수 있나요?"
현업: "여러 개요. 불량 부위 사진을 다 찍으니까"
```

**자동 도출**:
```sql
-- 불량기록지
CREATE TABLE defect_records (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  description TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 첨부파일 (1:N)
CREATE TABLE defect_attachments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  record_id UUID NOT NULL,
  file_name VARCHAR(255) NOT NULL,
  file_path VARCHAR(500) NOT NULL,
  file_size INTEGER,
  mime_type VARCHAR(100),
  uploaded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

  FOREIGN KEY (record_id) REFERENCES defect_records(id) ON DELETE CASCADE
);

-- 인덱스
CREATE INDEX idx_defect_attachments_record
ON defect_attachments(record_id);
```

**패턴**: `붙임:` → 1:N 파일 테이블

---

## 패턴 2: Document 관계 → FK/계층

### Document 간 Reference Section 관계

**워크숍 Document 흐름도**:
```
FormType: "작업지시서" → [Document #WO001]
    ↓ (Reference Section)
FormType: "작업일지" → [Document #LOG001, #LOG002, ...] (여러 개)
    ↓ (Reference Section)
FormType: "품질검사의뢰서" → [Document #QC001] (선택)
```

**질문**:
```
개발자: "작업지시서 하나에 작업일지가 몇 개나요?"
현업: "여러 개죠. 하루에 하나씩 쓰니까"
개발자: "작업일지 없이 작업지시서만 있을 수 있나요?"
현업: "네, 아직 작업 시작 안 했으면요"
```

**Section → Entity 사상 - Document 간 관계**:
```sql
-- FormType: "작업지시서" → Main Section → work_orders
CREATE TABLE work_orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_number VARCHAR(20) UNIQUE NOT NULL,
  product_id UUID NOT NULL,
  quantity INTEGER NOT NULL,
  status VARCHAR(20) DEFAULT 'PENDING',  -- PENDING, IN_PROGRESS, COMPLETED

  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- FormType: "작업일지" → Main Section with Reference Section → work_logs
CREATE TABLE work_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  work_order_id UUID NOT NULL,          -- Reference Section → Parent Document FK
  log_date DATE NOT NULL,
  worker_name VARCHAR(50),
  good_quantity INTEGER DEFAULT 0,
  defect_quantity INTEGER DEFAULT 0,

  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

  FOREIGN KEY (work_order_id) REFERENCES work_orders(id) ON DELETE CASCADE
);

-- Document 상태 자동 업데이트 트리거
CREATE OR REPLACE FUNCTION update_work_order_status()
RETURNS TRIGGER AS $$
BEGIN
  -- 작업일지 Document가 추가되면 작업지시서 Document 상태 변경
  UPDATE work_orders
  SET status = 'IN_PROGRESS'
  WHERE id = NEW.work_order_id AND status = 'PENDING';

  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_work_log_insert
AFTER INSERT ON work_logs
FOR EACH ROW
EXECUTE FUNCTION update_work_order_status();
```

**패턴**: Document 흐름 `A → B` → B의 Reference Section이 A를 FK로 참조 (1:N)

---

### Attachment Section 관계 (Snapshot)

**워크숍 대화**:
```
현업: "작업지시서에 제품명이 있는데, 작업일지에도 제품명을 적어요"
개발자: "작업지시서에서 참조하면 되지 않나요?"
현업: "아니요, 그때그때 기록해야 해요. 나중에 작업지시서가 바뀔 수도 있으니까"
```

**Section → Entity 사상 - Attachment Section (과거 시점 고정)**:
```sql
-- FormType: "작업일지" with Attachment Section
CREATE TABLE work_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  work_order_id UUID NOT NULL,          -- Reference Section (living link)

  -- Attachment Section Fields: 작업지시서 Document로부터 스냅샷 복사
  product_name VARCHAR(100) NOT NULL,   -- 그 시점의 제품명 (frozen)
  product_code VARCHAR(50) NOT NULL,    -- 그 시점의 제품 코드 (frozen)
  target_quantity INTEGER NOT NULL,     -- 그 시점의 목표 수량 (frozen)

  -- Main Section Fields: 작업일지 고유 데이터
  log_date DATE NOT NULL,
  actual_quantity INTEGER,

  FOREIGN KEY (work_order_id) REFERENCES work_orders(id)
);

-- Attachment Section 자동 스냅샷 트리거
CREATE OR REPLACE FUNCTION snapshot_work_order_data()
RETURNS TRIGGER AS $$
BEGIN
  -- Attachment Section: Document 생성 시점 데이터 복사 (과거 시점 고정)
  SELECT
    p.name,
    p.code,
    wo.quantity
  INTO
    NEW.product_name,
    NEW.product_code,
    NEW.target_quantity
  FROM work_orders wo
  JOIN products p ON wo.product_id = p.id
  WHERE wo.id = NEW.work_order_id;

  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_snapshot_before_insert
BEFORE INSERT ON work_logs
FOR EACH ROW
EXECUTE FUNCTION snapshot_work_order_data();
```

**패턴**: `Attachment Section` → 스냅샷 복사 (과거 시점 고정, 원본 변경과 무관)

---

## 패턴 3: 문서 접미어 → 테이블 특성

### -서(書): 공식 문서 = 승인 워크플로우

```sql
-- 모든 "-서" 문서의 공통 패턴
CREATE TABLE approval_documents (
  id UUID PRIMARY KEY,
  document_type VARCHAR(50) NOT NULL,   -- 작업지시서, 품질검사의뢰서...

  -- 승인 워크플로우
  status VARCHAR(20) DEFAULT 'DRAFT',   -- DRAFT, SUBMITTED, APPROVED, REJECTED
  submitted_at TIMESTAMP,
  submitted_by UUID,
  approved_at TIMESTAMP,
  approved_by UUID,
  approval_comments TEXT,

  -- 법적 효력
  is_official BOOLEAN DEFAULT true,
  signature_required BOOLEAN DEFAULT true,

  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  created_by UUID NOT NULL
);

-- 작업지시서 (상속 패턴)
CREATE TABLE work_orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  -- approval_documents 패턴 포함
  status VARCHAR(20) DEFAULT 'DRAFT',
  submitted_at TIMESTAMP,
  approved_at TIMESTAMP,
  approved_by UUID,

  -- 작업지시서 고유 데이터
  order_number VARCHAR(20) UNIQUE NOT NULL,
  product_id UUID NOT NULL,
  quantity INTEGER NOT NULL
);

-- 승인 프로세스 함수
CREATE OR REPLACE FUNCTION submit_for_approval(doc_id UUID, doc_type VARCHAR)
RETURNS VOID AS $$
BEGIN
  -- 문서 타입에 따라 동적 처리
  EXECUTE format('
    UPDATE %I
    SET status = ''SUBMITTED'',
        submitted_at = CURRENT_TIMESTAMP,
        submitted_by = current_user_id()
    WHERE id = $1
  ', doc_type || 's')
  USING doc_id;

  -- 승인자에게 알림
  INSERT INTO notifications (user_id, message, link)
  SELECT
    approver_id,
    format(''%s 승인 요청'', doc_type),
    format(''/%s/%s'', doc_type, doc_id)
  FROM approvers
  WHERE document_type = doc_type;
END;
$$ LANGUAGE plpgsql;
```

**패턴**: `-서` → 승인 워크플로우 필드 추가

---

### -지(紙): 단순 기록 = 이력 관리

```sql
-- 모든 "-지" 문서의 공통 패턴
CREATE TABLE record_documents (
  id UUID PRIMARY KEY,

  -- 이력 추적
  history JSONB DEFAULT '[]'::jsonb,   -- 변경 이력
  version INTEGER DEFAULT 1,

  -- 승인 불필요
  status VARCHAR(20) DEFAULT 'ACTIVE',  -- ACTIVE, ARCHIVED

  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  created_by UUID NOT NULL,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_by UUID
);

-- 검사기록지
CREATE TABLE inspection_records (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  -- record_documents 패턴
  history JSONB DEFAULT '[]'::jsonb,
  version INTEGER DEFAULT 1,

  -- 검사기록지 고유 데이터
  qc_request_id UUID NOT NULL,
  inspector_name VARCHAR(50),
  result VARCHAR(20),  -- PASS, FAIL

  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 자동 이력 기록 트리거
CREATE OR REPLACE FUNCTION track_record_history()
RETURNS TRIGGER AS $$
BEGIN
  NEW.history = NEW.history || jsonb_build_object(
    'version', NEW.version,
    'updated_at', CURRENT_TIMESTAMP,
    'updated_by', current_user_id(),
    'changes', jsonb_build_object(
      'old', to_jsonb(OLD),
      'new', to_jsonb(NEW)
    )
  );

  NEW.version = NEW.version + 1;
  NEW.updated_at = CURRENT_TIMESTAMP;

  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_inspection_record_history
BEFORE UPDATE ON inspection_records
FOR EACH ROW
EXECUTE FUNCTION track_record_history();
```

**패턴**: `-지` → 이력 추적 JSONB 필드

---

### -표(表): 집계/통계 = 집계 뷰

```sql
-- "-표"는 실제 테이블이 아닌 뷰로 구현
CREATE VIEW monthly_production_summary AS
SELECT
  DATE_TRUNC('month', log_date) as month,
  product_id,
  p.name as product_name,
  SUM(good_quantity) as total_good,
  SUM(defect_quantity) as total_defect,
  ROUND(
    100.0 * SUM(good_quantity) / NULLIF(SUM(good_quantity + defect_quantity), 0),
    2
  ) as yield_rate,
  COUNT(DISTINCT work_order_id) as order_count
FROM work_logs wl
JOIN work_orders wo ON wl.work_order_id = wo.id
JOIN products p ON wo.product_id = p.id
GROUP BY
  DATE_TRUNC('month', log_date),
  product_id,
  p.name;

-- 필요시 Materialized View로 성능 향상
CREATE MATERIALIZED VIEW monthly_production_summary_mat AS
SELECT * FROM monthly_production_summary;

-- 매일 자정 갱신
CREATE OR REPLACE FUNCTION refresh_production_summary()
RETURNS VOID AS $$
BEGIN
  REFRESH MATERIALIZED VIEW monthly_production_summary_mat;
END;
$$ LANGUAGE plpgsql;

-- 스케줄링 (pg_cron 확장 사용)
SELECT cron.schedule(
  'refresh-production-summary',
  '0 0 * * *',  -- 매일 자정
  'SELECT refresh_production_summary();'
);
```

**패턴**: `-표` → 집계 뷰 또는 Materialized View

---

### -록(錄): 이력 = 시계열 테이블

```sql
-- "-록"은 시계열 데이터 + 검색 최적화
CREATE TABLE defect_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  -- 시계열 필드
  occurred_at TIMESTAMP NOT NULL,
  recorded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

  -- 이력 데이터
  product_id UUID NOT NULL,
  defect_type_id UUID NOT NULL,
  quantity INTEGER NOT NULL,
  cost DECIMAL(10, 2),

  -- 메타데이터
  work_order_id UUID,
  worker_name VARCHAR(50),
  shift VARCHAR(20),

  -- 검색 최적화
  search_text TSVECTOR
);

-- 시계열 인덱스 (최근 데이터 조회 최적화)
CREATE INDEX idx_defect_history_occurred
ON defect_history(occurred_at DESC);

-- 파티셔닝 (월별)
CREATE TABLE defect_history_2024_01
PARTITION OF defect_history
FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE defect_history_2024_02
PARTITION OF defect_history
FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');

-- 전문 검색 (PostgreSQL Full-Text Search)
CREATE INDEX idx_defect_history_search
ON defect_history USING GIN(search_text);

-- 검색 텍스트 자동 생성
CREATE OR REPLACE FUNCTION generate_search_text()
RETURNS TRIGGER AS $$
BEGIN
  NEW.search_text = to_tsvector('korean',
    COALESCE(NEW.worker_name, '') || ' ' ||
    COALESCE(NEW.shift, '') || ' ' ||
    (SELECT name FROM products WHERE id = NEW.product_id) || ' ' ||
    (SELECT name FROM defect_types WHERE id = NEW.defect_type_id)
  );
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_defect_history_search
BEFORE INSERT OR UPDATE ON defect_history
FOR EACH ROW
EXECUTE FUNCTION generate_search_text();
```

**패턴**: `-록` → 시계열 테이블 + 파티셔닝 + 전문검색

---

## 완전한 예제: 제조 MES

### 워크숍 결과 → ERD

**도출된 문서**:
1. 작업지시서 (서)
2. 작업일지 (지)
3. 품질검사의뢰서 (서)
4. 검사기록지 (지)
5. 월별생산현황표 (표)
6. 불량이력록 (록)

**ERD 다이어그램** (텍스트):

```
┌─────────────┐       ┌─────────────┐
│  products   │───┐   │  machines   │
│ (제품마스터) │   │   │  (설비)     │
└─────────────┘   │   └─────────────┘
                  │           │
                  ↓           ↓
            ┌──────────────────┐
            │  work_orders     │ ← -서 (승인)
            │  (작업지시서)    │
            └──────────────────┘
                     │ 1:N
                     ↓
            ┌──────────────────┐
            │  work_logs       │ ← -지 (이력)
            │  (작업일지)      │
            └──────────────────┘
                     │ 1:N
                     ↓
            ┌──────────────────┐
            │  qc_requests     │ ← -서 (승인)
            │  (검사의뢰서)    │
            └──────────────────┘
                     │ 1:1
                     ↓
            ┌──────────────────┐
            │  inspection_     │ ← -지 (이력)
            │  records         │
            │  (검사기록지)    │
            └──────────────────┘
                     │
                ┌────┴────┐
                ↓         ↓
        ┌───────────┐ ┌───────────┐
        │ pass_     │ │ defect_   │
        │ records   │ │ records   │
        └───────────┘ └───────────┘
                             │
                             ↓
                    ┌──────────────┐
                    │ defect_      │ ← -록 (시계열)
                    │ history      │
                    └──────────────┘

VIEW: monthly_production_summary ← -표 (집계)
```

### 완전한 DDL

```sql
-- ============================================
-- 마스터 테이블
-- ============================================

-- 제품
CREATE TABLE products (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  code VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  spec TEXT,
  unit VARCHAR(20) DEFAULT 'EA',

  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  created_by UUID NOT NULL
);

-- 설비
CREATE TABLE machines (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  code VARCHAR(50) UNIQUE NOT NULL,
  name VARCHAR(100) NOT NULL,
  type VARCHAR(50),
  status VARCHAR(20) DEFAULT 'ACTIVE',

  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 불량 유형
CREATE TABLE defect_types (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  code VARCHAR(20) UNIQUE NOT NULL,
  name VARCHAR(50) NOT NULL,
  category VARCHAR(50),
  severity VARCHAR(20)  -- LOW, MEDIUM, HIGH, CRITICAL
);

-- ============================================
-- 문서 테이블: -서 (작업지시서)
-- ============================================

CREATE TABLE work_orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_number VARCHAR(20) UNIQUE NOT NULL,

  -- 문서 내용
  product_id UUID NOT NULL,
  machine_id UUID NOT NULL,
  target_quantity INTEGER NOT NULL,
  due_date DATE NOT NULL,
  priority VARCHAR(20) DEFAULT 'NORMAL',

  -- 승인 워크플로우 (-서 패턴)
  status VARCHAR(20) DEFAULT 'DRAFT',
  submitted_at TIMESTAMP,
  submitted_by UUID,
  approved_at TIMESTAMP,
  approved_by UUID,

  -- 메타
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  created_by UUID NOT NULL,

  FOREIGN KEY (product_id) REFERENCES products(id),
  FOREIGN KEY (machine_id) REFERENCES machines(id)
);

CREATE INDEX idx_work_orders_status ON work_orders(status);
CREATE INDEX idx_work_orders_due_date ON work_orders(due_date);

-- ============================================
-- 문서 테이블: -지 (작업일지)
-- ============================================

CREATE TABLE work_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  work_order_id UUID NOT NULL,

  -- 스냅샷 데이터
  product_name VARCHAR(100) NOT NULL,
  machine_name VARCHAR(100) NOT NULL,
  target_quantity INTEGER NOT NULL,

  -- 작업일지 내용
  log_date DATE NOT NULL,
  worker_name VARCHAR(50) NOT NULL,
  shift VARCHAR(20),
  start_time TIME,
  end_time TIME,
  good_quantity INTEGER DEFAULT 0,
  defect_quantity INTEGER DEFAULT 0,
  rework_quantity INTEGER DEFAULT 0,
  notes TEXT,

  -- 이력 관리 (-지 패턴)
  history JSONB DEFAULT '[]'::jsonb,
  version INTEGER DEFAULT 1,

  -- 메타
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  created_by UUID NOT NULL,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

  FOREIGN KEY (work_order_id) REFERENCES work_orders(id) ON DELETE CASCADE
);

CREATE INDEX idx_work_logs_order ON work_logs(work_order_id);
CREATE INDEX idx_work_logs_date ON work_logs(log_date DESC);

-- ============================================
-- 문서 테이블: -서 (품질검사의뢰서)
-- ============================================

CREATE TABLE qc_requests (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  request_number VARCHAR(20) UNIQUE NOT NULL,

  -- 관계
  work_log_id UUID NOT NULL,
  product_id UUID NOT NULL,

  -- 내용
  lot_number VARCHAR(50) NOT NULL,
  quantity INTEGER NOT NULL,
  requested_date DATE NOT NULL,

  -- 승인 워크플로우 (-서 패턴)
  status VARCHAR(20) DEFAULT 'PENDING',
  inspected_at TIMESTAMP,
  inspected_by UUID,

  -- 메타
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  created_by UUID NOT NULL,

  FOREIGN KEY (work_log_id) REFERENCES work_logs(id),
  FOREIGN KEY (product_id) REFERENCES products(id)
);

-- ============================================
-- 문서 테이블: -지 (검사기록지)
-- ============================================

CREATE TABLE inspection_records (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  qc_request_id UUID UNIQUE NOT NULL,

  -- 검사 내용
  inspector_name VARCHAR(50) NOT NULL,
  inspection_date DATE NOT NULL,
  result VARCHAR(20) NOT NULL,  -- PASS, FAIL

  -- 검사 항목 (JSONB로 유연하게)
  inspection_items JSONB NOT NULL,
  /*
  {
    "appearance": { "result": "PASS", "notes": "" },
    "dimension": { "result": "PASS", "value": 10.5, "spec": "10.0±0.5" },
    "weight": { "result": "FAIL", "value": 95, "spec": "100±5" }
  }
  */

  -- 이력 관리 (-지 패턴)
  history JSONB DEFAULT '[]'::jsonb,
  version INTEGER DEFAULT 1,

  -- 메타
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  created_by UUID NOT NULL,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

  FOREIGN KEY (qc_request_id) REFERENCES qc_requests(id)
);

-- ============================================
-- 문서 테이블: 불량기록지 (검사 결과가 FAIL일 때)
-- ============================================

CREATE TABLE defect_records (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  inspection_record_id UUID NOT NULL,

  -- 불량 상세
  defect_type_id UUID NOT NULL,
  quantity INTEGER NOT NULL,
  severity VARCHAR(20),
  description TEXT,
  root_cause TEXT,

  -- 첨부파일
  attachments JSONB DEFAULT '[]'::jsonb,

  -- 메타
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  created_by UUID NOT NULL,

  FOREIGN KEY (inspection_record_id) REFERENCES inspection_records(id),
  FOREIGN KEY (defect_type_id) REFERENCES defect_types(id)
);

-- ============================================
-- 문서 테이블: -록 (불량이력록)
-- ============================================

CREATE TABLE defect_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  -- 시계열 필드 (-록 패턴)
  occurred_at TIMESTAMP NOT NULL,
  recorded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

  -- 관계
  product_id UUID NOT NULL,
  defect_type_id UUID NOT NULL,
  defect_record_id UUID,
  work_order_id UUID,

  -- 이력 데이터
  quantity INTEGER NOT NULL,
  estimated_cost DECIMAL(10, 2),
  shift VARCHAR(20),
  worker_name VARCHAR(50),

  -- 검색 최적화
  search_text TSVECTOR,

  FOREIGN KEY (product_id) REFERENCES products(id),
  FOREIGN KEY (defect_type_id) REFERENCES defect_types(id),
  FOREIGN KEY (defect_record_id) REFERENCES defect_records(id)
) PARTITION BY RANGE (occurred_at);

-- 파티션 생성 (월별)
CREATE TABLE defect_history_2024_01 PARTITION OF defect_history
FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE INDEX idx_defect_history_occurred
ON defect_history(occurred_at DESC);

CREATE INDEX idx_defect_history_search
ON defect_history USING GIN(search_text);

-- ============================================
-- 뷰: -표 (월별생산현황표)
-- ============================================

CREATE VIEW monthly_production_summary AS
SELECT
  DATE_TRUNC('month', wl.log_date) as month,
  p.id as product_id,
  p.code as product_code,
  p.name as product_name,
  COUNT(DISTINCT wo.id) as order_count,
  COUNT(DISTINCT wl.id) as log_count,
  SUM(wl.good_quantity) as total_good,
  SUM(wl.defect_quantity) as total_defect,
  SUM(wl.rework_quantity) as total_rework,
  ROUND(
    100.0 * SUM(wl.good_quantity) /
    NULLIF(SUM(wl.good_quantity + wl.defect_quantity), 0),
    2
  ) as yield_rate,
  SUM(wo.target_quantity) as total_target,
  ROUND(
    100.0 * SUM(wl.good_quantity) / NULLIF(SUM(wo.target_quantity), 0),
    2
  ) as achievement_rate
FROM work_logs wl
JOIN work_orders wo ON wl.work_order_id = wo.id
JOIN products p ON wo.product_id = p.id
WHERE wo.status = 'APPROVED'
GROUP BY
  DATE_TRUNC('month', wl.log_date),
  p.id, p.code, p.name
ORDER BY month DESC, p.code;

-- Materialized View로 성능 향상 (선택)
CREATE MATERIALIZED VIEW monthly_production_summary_mat AS
SELECT * FROM monthly_production_summary;

CREATE INDEX idx_monthly_summary_month
ON monthly_production_summary_mat(month DESC);

-- ============================================
-- 트리거 및 함수
-- ============================================

-- 1. 작업지시서 번호 자동 생성
CREATE OR REPLACE FUNCTION generate_work_order_number()
RETURNS TRIGGER AS $$
BEGIN
  NEW.order_number = 'WO' ||
    TO_CHAR(CURRENT_DATE, 'YYYYMMDD') ||
    LPAD(NEXTVAL('work_order_seq')::TEXT, 4, '0');
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE SEQUENCE work_order_seq;

CREATE TRIGGER trg_work_order_number
BEFORE INSERT ON work_orders
FOR EACH ROW
WHEN (NEW.order_number IS NULL)
EXECUTE FUNCTION generate_work_order_number();

-- 2. 작업일지 스냅샷 생성
CREATE OR REPLACE FUNCTION snapshot_work_order_to_log()
RETURNS TRIGGER AS $$
BEGIN
  SELECT
    p.name,
    m.name,
    wo.target_quantity
  INTO
    NEW.product_name,
    NEW.machine_name,
    NEW.target_quantity
  FROM work_orders wo
  JOIN products p ON wo.product_id = p.id
  JOIN machines m ON wo.machine_id = m.id
  WHERE wo.id = NEW.work_order_id;

  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_work_log_snapshot
BEFORE INSERT ON work_logs
FOR EACH ROW
EXECUTE FUNCTION snapshot_work_order_to_log();

-- 3. 불량이력록 자동 생성
CREATE OR REPLACE FUNCTION create_defect_history()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO defect_history (
    occurred_at,
    product_id,
    defect_type_id,
    defect_record_id,
    work_order_id,
    quantity,
    shift,
    worker_name
  )
  SELECT
    ir.inspection_date,
    qc.product_id,
    NEW.defect_type_id,
    NEW.id,
    wl.work_order_id,
    NEW.quantity,
    wl.shift,
    wl.worker_name
  FROM inspection_records ir
  JOIN qc_requests qc ON ir.qc_request_id = qc.id
  JOIN work_logs wl ON qc.work_log_id = wl.id
  WHERE ir.id = NEW.inspection_record_id;

  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_defect_history_auto
AFTER INSERT ON defect_records
FOR EACH ROW
EXECUTE FUNCTION create_defect_history();

-- 4. 이력 자동 추적
CREATE OR REPLACE FUNCTION track_document_history()
RETURNS TRIGGER AS $$
BEGIN
  NEW.history = NEW.history || jsonb_build_object(
    'version', NEW.version,
    'timestamp', CURRENT_TIMESTAMP,
    'user_id', current_user_id(),
    'changes', jsonb_build_object(
      'old', row_to_json(OLD),
      'new', row_to_json(NEW)
    )
  );
  NEW.version = NEW.version + 1;
  NEW.updated_at = CURRENT_TIMESTAMP;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trg_work_log_history
BEFORE UPDATE ON work_logs
FOR EACH ROW
EXECUTE FUNCTION track_document_history();

CREATE TRIGGER trg_inspection_record_history
BEFORE UPDATE ON inspection_records
FOR EACH ROW
EXECUTE FUNCTION track_document_history();
```

---

## 성능 최적화

### 인덱스 전략

```sql
-- 1. FK 인덱스 (자동 생성 권장)
CREATE INDEX idx_work_logs_order ON work_logs(work_order_id);
CREATE INDEX idx_qc_requests_log ON qc_requests(work_log_id);

-- 2. 검색 인덱스
CREATE INDEX idx_work_orders_status_date ON work_orders(status, due_date);
CREATE INDEX idx_work_logs_date_product ON work_logs(log_date, product_name);

-- 3. 복합 인덱스 (자주 함께 검색)
CREATE INDEX idx_defect_history_product_date
ON defect_history(product_id, occurred_at DESC);

-- 4. 부분 인덱스 (활성 데이터만)
CREATE INDEX idx_active_work_orders
ON work_orders(due_date)
WHERE status IN ('PENDING', 'IN_PROGRESS');

-- 5. 전문 검색 인덱스
CREATE INDEX idx_work_logs_notes_search
ON work_logs USING GIN(to_tsvector('korean', notes));
```

### 파티셔닝 전략

```sql
-- 시계열 데이터는 월별 파티셔닝
CREATE TABLE defect_history (
  -- ...
) PARTITION BY RANGE (occurred_at);

-- 자동 파티션 생성 함수
CREATE OR REPLACE FUNCTION create_monthly_partition(
  base_table TEXT,
  partition_date DATE
)
RETURNS VOID AS $$
DECLARE
  partition_name TEXT;
  start_date DATE;
  end_date DATE;
BEGIN
  partition_name = base_table || '_' || TO_CHAR(partition_date, 'YYYY_MM');
  start_date = DATE_TRUNC('month', partition_date);
  end_date = start_date + INTERVAL '1 month';

  EXECUTE format('
    CREATE TABLE IF NOT EXISTS %I PARTITION OF %I
    FOR VALUES FROM (%L) TO (%L)
  ', partition_name, base_table, start_date, end_date);
END;
$$ LANGUAGE plpgsql;

-- 매월 자동 생성 (pg_cron)
SELECT cron.schedule(
  'create-next-month-partition',
  '0 0 1 * *',  -- 매월 1일
  'SELECT create_monthly_partition(''defect_history'', CURRENT_DATE + INTERVAL ''1 month'')'
);
```

---

## 마이그레이션 전략

### 버전 관리

```sql
-- 스키마 버전 테이블
CREATE TABLE schema_versions (
  version VARCHAR(20) PRIMARY KEY,
  description TEXT,
  applied_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  applied_by VARCHAR(50)
);

-- 마이그레이션 스크립트 예시
-- V1.0__initial_schema.sql
-- V1.1__add_work_logs.sql
-- V1.2__add_qc_module.sql
```

### 점진적 적용

```
Phase 1: 작업지시서 + 작업일지
  → work_orders, work_logs 테이블만

Phase 2: 품질검사 모듈 추가
  → qc_requests, inspection_records 추가

Phase 3: 통계 및 이력
  → 뷰, 파티션, 이력 테이블 추가
```

---

## 요약: 워크숍 → DDL 변환 프로세스

```
1. 워크숍: 문서 양식 스케치
   ↓
2. 패턴 인식:
   - [선택] → FK
   - [입력] → 컬럼
   - 체크박스 → 1:N
   - 참조 → FK (최신)
   - 붙임 → 파일 테이블
   ↓
3. 관계 도출:
   - A → B → 부모-자식
   - "그때" → 스냅샷
   ↓
4. 접미어 패턴:
   - -서 → 승인 워크플로우
   - -지 → 이력 관리
   - -표 → 집계 뷰
   - -록 → 시계열 파티션
   ↓
5. DDL 생성:
   - CREATE TABLE
   - CREATE INDEX
   - CREATE TRIGGER
   - CREATE VIEW
   ↓
6. 현업 검증:
   - "이 테이블이 OO 문서죠?"
   - "맞아요!" ✅

소요 시간: 워크숍 2시간 + DDL 작성 4시간 = 6시간
```

**전통 ERD 방식**: 2-4주
**Formology 방식**: 6시간
**시간 절감**: 95%

---

**관련 문서**:
- [데이터 모델링 개념](../02-methodology/data-modeling.md)
- [API 설계](api-design.md)
- [제조 MES 사례](../06-case-studies/manufacturing-mes.md)
