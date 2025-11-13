# 템플릿 모음

## 개요

Formology 워크숍 및 구현 과정에서 바로 사용할 수 있는 문서 양식, 코드 템플릿, 설정 파일 모음입니다.

### 구성

```
01. 워크숍 준비 템플릿
02. 문서 양식 템플릿 (서/지/표/록)
03. 데이터베이스 스키마 템플릿
04. API 코드 템플릿
05. 프로젝트 구조 템플릿
```

---

## 01. 워크숍 준비 템플릿

### 1.1 초대 이메일 템플릿

```markdown
제목: [Formology 워크숍] {시스템명} 문서 정의 세션 안내

안녕하세요,

{시스템명} 개발을 위한 Formology 워크숍을 아래와 같이 진행합니다.

■ 일시: {YYYY.MM.DD} ({요일}) {HH:MM}~{HH:MM} ({N}시간)
■ 장소: {장소명}
■ 참석자: {팀명} {N}명, {팀명} {N}명, 개발팀 {N}명
■ 목적: 업무 문서 목록 도출 및 양식 정의

■ 준비사항:
- 현재 사용 중인 문서 양식 (종이, Excel, PDF 등) 가져오기
- 업무 흐름에 대한 이해
- 특별한 IT 지식은 불필요합니다

■ 진행 방식:
1. 여러분이 실제로 사용하는 "문서"를 나열합니다
2. 문서의 양식을 간단히 스케치합니다
3. 문서 간 흐름을 정리합니다

※ Entity, Domain Model 등 어려운 용어는 사용하지 않습니다
※ 여러분의 업무 용어 그대로 사용합니다

궁금한 점은 언제든 문의해주세요.

감사합니다.

---
{담당자명}
{연락처}
```

---

### 1.2 워크숍 체크리스트

```markdown
# Formology 워크숍 체크리스트

## 1주 전
- [ ] 참석자 선정 (현업 3~5명, 개발 1~2명)
- [ ] 회의실 예약 (6~8인용, 2~3시간)
- [ ] 준비물 구매
  - [ ] 포스트잇 5색 (각 100장)
  - [ ] 마커펜 굵은 것 (5색)
  - [ ] A4 용지 30장
- [ ] 초대 메일 발송
- [ ] 참석 확인

## 1일 전
- [ ] 참석자 리마인더
- [ ] 준비물 최종 점검
- [ ] 회의실 재확인
- [ ] 진행 시나리오 리허설

## 당일 30분 전
- [ ] 회의실 환기
- [ ] 화이트보드 지우기
- [ ] 포스트잇, 마커 테이블 배치
- [ ] 간식, 음료 준비
- [ ] 카메라 준비
- [ ] 휴대폰 진동 모드

## 워크숍 종료 직후
- [ ] 화이트보드 사진 촬영 (최소 5장)
- [ ] 포스트잇 정리 (버리지 말 것!)
- [ ] 참석자 감사 메일
- [ ] 산출물 디지털화 시작
```

---

### 1.3 워크숍 타임테이블 템플릿

```markdown
# {시스템명} Formology 워크숍 일정

**일시**: {YYYY.MM.DD} ({요일}) {HH:MM}~{HH:MM}
**장소**: {장소명}
**참석자**: {참석자 목록}

| 시간 | 단계 | 활동 | 산출물 | 진행자 |
|------|------|------|--------|--------|
| {HH:MM}-{HH:MM} | 오프닝 | 눈손이 소개, 규칙 설명 | 참여자 이해 | {이름} |
| {HH:MM}-{HH:MM} | 업무 선정 | 대상 업무 선택 및 범위 | 업무 범위 | {이름} |
| {HH:MM}-{HH:MM} | 브레인스토밍 | 문서 목록 도출 | 15~30개 문서 | {이름} |
| {HH:MM}-{HH:MM} | 그룹핑 | 비슷한 문서 묶기 | 4~6개 그룹 | {이름} |
| {HH:MM}-{HH:MM} | 우선순위 | 중요도×빈도 매트릭스 | MVP 범위 | {이름} |
| {HH:MM}-{HH:MM} | **휴식** | 15분 휴식 | - | - |
| {HH:MM}-{HH:MM} | 흐름 그리기 | 문서 간 화살표 연결 | 워크플로우 | {이름} |
| {HH:MM}-{HH:MM} | 양식 스케치 | 주요 문서 3~5개 상세화 | 양식 초안 | {이름} |
| {HH:MM}-{HH:MM} | 데이터 관계 | 입력 방식 확인 | ERD 초안 | {이름} |
| {HH:MM}-{HH:MM} | 확인 및 정리 | 체크리스트 확인 | 최종 산출물 | {이름} |
| {HH:MM}-{HH:MM} | 마무리 | 다음 단계 안내 | 액션 아이템 | {이름} |
```

---

## 02. 문서 양식 템플릿 (서/지/표/록)

### 2.1 요청서 (-서) 템플릿

```markdown
# {문서명} (-서)

## 기본 정보
- **문서번호**: [자동생성]
- **작성일**: [자동입력]
- **작성자**: [자동입력]
- **상태**: 임시저장 / 제출 / 승인 / 반려

---

## 요청 내용

### 1. {항목명}
- **타입**: [직접입력] / [선택 ▼] / [체크박스]
- **필수여부**: 필수 / 선택
- **설명**: {항목 설명}

### 2. {항목명}
- **타입**: [직접입력]
- **필수여부**: 필수
- **설명**: {항목 설명}

### 3. {항목명}
- **타입**: [선택 ▼]
- **옵션**: 옵션1, 옵션2, 옵션3
- **필수여부**: 필수
- **설명**: {항목 설명}

---

## 첨부 파일
- **붙임**: {파일명} (최대 {N}개, {N}MB 이하)

---

## 승인 정보
- **제출일시**: [자동입력]
- **승인자**: [자동지정]
- **승인일시**: [자동입력]
- **승인의견**: [텍스트]

---

## 이력
| 일시 | 작업자 | 작업 | 내용 |
|------|--------|------|------|
| {YYYY-MM-DD HH:MM} | {이름} | 작성 | 최초 작성 |
| {YYYY-MM-DD HH:MM} | {이름} | 제출 | 승인 요청 |
| {YYYY-MM-DD HH:MM} | {이름} | 승인 | 승인 완료 |
```

**Excel 템플릿** (작업지시서 예시):

| 항목 | 타입 | 필수 | 예시 |
|------|------|------|------|
| 지시서번호 | 자동생성 | ○ | WO20241105001 |
| 제품 | 선택 ▼ | ○ | 제품A |
| 설비 | 선택 ▼ | ○ | 설비1호기 |
| 목표수량 | 직접입력 | ○ | 1000 |
| 마감일 | 날짜선택 | ○ | 2024-11-20 |
| 우선순위 | 선택 ▼ | ○ | 높음 / 보통 / 낮음 |
| 비고 | 직접입력 | | 긴급 생산 |
| 작성일 | 자동입력 | ○ | 2024-11-05 10:00 |
| 작성자 | 자동입력 | ○ | 홍길동 |
| 상태 | 자동관리 | ○ | 임시저장 / 제출 / 승인 / 반려 |

---

### 2.2 기록지 (-지) 템플릿

```markdown
# {문서명} (-지)

## 기본 정보
- **문서번호**: [자동생성]
- **기록일**: {YYYY-MM-DD}
- **기록자**: [자동입력]

---

## 기록 내용

### 1. 참조 문서
- **참조**: [선택 ▼] (최신 버전 자동 연결)
- **예시**: 작업지시서 WO20241105001

### 2. {기록 항목}
- **타입**: [직접입력] / [측정값]
- **단위**: {단위}
- **측정값**: _______

### 3. {체크리스트}
- [ ] {항목1}
- [ ] {항목2}
- [ ] {항목3}

### 4. 특이사항
- **내용**: [텍스트]

---

## 첨부 파일
- **붙임**: 사진, 문서 등

---

## 이력 추적 (자동 기록)

| 버전 | 일시 | 작업자 | 변경 내용 |
|------|------|--------|----------|
| 1 | {YYYY-MM-DD HH:MM} | {이름} | 최초 작성 |
| 2 | {YYYY-MM-DD HH:MM} | {이름} | {항목} 수정 |
| 3 | {YYYY-MM-DD HH:MM} | {이름} | {항목} 추가 |
```

**Excel 템플릿** (품질검사보고서 예시):

| 항목 | 타입 | 필수 | 예시 |
|------|------|------|------|
| 보고서번호 | 자동생성 | ○ | QR20241105001 |
| 참조: 작업지시서 | 선택 ▼ | ○ | WO20241105001 (최신) |
| 검사일 | 날짜선택 | ○ | 2024-11-05 |
| 검사자 | 직접입력 | ○ | 검사자A |
| **검사 항목** | | | |
| 1. 치수 검사 | | | |
| - 규격 | 직접입력 | ○ | 100±0.5mm |
| - 측정값 | 직접입력 | ○ | 100.2mm |
| - 결과 | 선택 ▼ | ○ | OK / NG |
| 2. 표면 검사 | | | |
| - 규격 | 직접입력 | ○ | 긁힘 없음 |
| - 측정값 | 직접입력 | ○ | 양호 |
| - 결과 | 선택 ▼ | ○ | OK / NG |
| 종합 결과 | 선택 ▼ | ○ | OK / NG |
| NG 조치 | 직접입력 | | 재작업 지시 |

---

### 2.3 현황표 (-표) 템플릿

```markdown
# {문서명} (-표)

## 조회 조건
- **기준일**: {YYYY-MM-DD}
- **기간**: {YYYY-MM-DD} ~ {YYYY-MM-DD}
- **구분**: 전체 / {분류1} / {분류2}

---

## 요약

| 지표 | 값 | 단위 |
|------|------|------|
| {지표1} | {값} | {단위} |
| {지표2} | {값} | {단위} |
| {지표3} | {값} | {단위} |
| {비율} | {값} | % |

---

## 상세 현황

### 1. {분류명}별 현황

| {분류} | {지표1} | {지표2} | {지표3} | {비율} |
|--------|---------|---------|---------|--------|
| {항목1} | {값} | {값} | {값} | {값}% |
| {항목2} | {값} | {값} | {값} | {값}% |
| {항목3} | {값} | {값} | {값} | {값}% |
| **합계** | **{값}** | **{값}** | **{값}** | **{값}%** |

### 2. {분류명}별 현황

| {분류} | {지표1} | {지표2} | {지표3} |
|--------|---------|---------|---------|
| {항목1} | {값} | {값} | {값} |
| {항목2} | {값} | {값} | {값} |
| **합계** | **{값}** | **{값}** | **{값}** |

---

## 내보내기
- **Excel**: [다운로드]
- **PDF**: [다운로드]
- **CSV**: [다운로드]
```

**Excel 템플릿** (일일 생산현황 예시):

```
# 일일 생산 현황표

기준일: 2024-11-05

## 요약
총 작업지시: 12건
완료: 8건 (66.7%)
진행중: 4건 (33.3%)
총 생산량: 8,500개
총 불량: 120개 (1.41%)

## 설비별 현황
| 설비명 | 지시건수 | 생산량 | 불량 | 불량률 | 가동률 |
|--------|----------|--------|------|--------|--------|
| 설비1호기 | 3 | 2,500 | 30 | 1.20% | 85.5% |
| 설비2호기 | 5 | 4,000 | 60 | 1.50% | 92.3% |
| 설비3호기 | 4 | 2,000 | 30 | 1.50% | 78.2% |
| **합계** | **12** | **8,500** | **120** | **1.41%** | **85.3%** |

## 제품별 현황
| 제품명 | 생산량 | 불량 | 불량률 |
|--------|--------|------|--------|
| 제품A | 3,000 | 40 | 1.33% |
| 제품B | 4,000 | 60 | 1.50% |
| 제품C | 1,500 | 20 | 1.33% |
| **합계** | **8,500** | **120** | **1.41%** |
```

---

### 2.4 대장 (-록) 템플릿

```markdown
# {문서명} (-록)

## 검색 조건
- **검색어**: [텍스트]
- **분류**: 전체 / {분류1} / {분류2}
- **상태**: 전체 / {상태1} / {상태2}
- **기간**: {YYYY-MM-DD} ~ {YYYY-MM-DD}

---

## 목록

| 번호 | {항목1} | {항목2} | {항목3} | {항목4} | 등록일 | 상태 | 작업 |
|------|---------|---------|---------|---------|--------|------|------|
| 1 | {값} | {값} | {값} | {값} | {YYYY-MM-DD} | {상태} | [수정] [삭제] |
| 2 | {값} | {값} | {값} | {값} | {YYYY-MM-DD} | {상태} | [수정] [삭제] |
| 3 | {값} | {값} | {값} | {값} | {YYYY-MM-DD} | {상태} | [수정] [삭제] |

---

## 페이징
[◀ 이전] 1 2 3 4 5 [다음 ▶]

총 {N}건 | 페이지 {M}/{P}

---

## 작업
- [신규 등록]
- [일괄 다운로드]
- [Excel 업로드]
```

**Excel 템플릿** (제품 대장 예시):

| 제품코드 | 제품명 | 분류 | 단위 | 표준단가 | 안전재고 | 상태 | 등록일 |
|----------|--------|------|------|----------|----------|------|--------|
| P001 | 제품A | 완제품 | EA | 10,000 | 100 | 사용중 | 2024-01-01 |
| P002 | 제품B | 완제품 | EA | 15,000 | 150 | 사용중 | 2024-01-05 |
| P003 | 제품C | 완제품 | EA | 8,000 | 80 | 단종 | 2024-01-10 |

---

## 03. 데이터베이스 스키마 템플릿

### 3.1 요청서 (-서) 테이블 템플릿

```sql
-- ===== {테이블명} (-서 패턴) =====
CREATE TABLE {table_name} (
  -- 기본키
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  -- 문서번호 (자동생성)
  {document_number} VARCHAR(20) UNIQUE NOT NULL,

  -- 업무 필드 (워크숍에서 도출)
  {field1_name} {data_type} NOT NULL,
  {field2_name} {data_type},
  {field3_id} UUID NOT NULL, -- [선택 ▼] = FK

  -- 승인 워크플로우 (-서 패턴 필수)
  status VARCHAR(20) DEFAULT 'DRAFT'
    CHECK (status IN ('DRAFT', 'SUBMITTED', 'APPROVED', 'REJECTED')),
  submitted_at TIMESTAMP,
  submitted_by UUID,
  approved_at TIMESTAMP,
  approved_by UUID,
  approval_comment TEXT,
  rejected_at TIMESTAMP,
  rejected_by UUID,
  rejection_reason TEXT,

  -- 메타데이터
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  created_by UUID NOT NULL,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_by UUID,

  -- 외래키
  FOREIGN KEY ({field3_id}) REFERENCES {related_table}(id),
  FOREIGN KEY (created_by) REFERENCES users(id),
  FOREIGN KEY (submitted_by) REFERENCES users(id),
  FOREIGN KEY (approved_by) REFERENCES users(id),
  FOREIGN KEY (rejected_by) REFERENCES users(id)
);

-- 인덱스
CREATE INDEX idx_{table_name}_status ON {table_name}(status);
CREATE INDEX idx_{table_name}_created_at ON {table_name}(created_at);
CREATE INDEX idx_{table_name}_{field3_id} ON {table_name}({field3_id});

-- 문서번호 자동생성 시퀀스
CREATE SEQUENCE {table_name}_seq;

-- 문서번호 자동생성 트리거
CREATE OR REPLACE FUNCTION generate_{table_name}_number()
RETURNS TRIGGER AS $$
BEGIN
  NEW.{document_number} = '{PREFIX}' ||
    TO_CHAR(CURRENT_DATE, 'YYYYMMDD') ||
    LPAD(NEXTVAL('{table_name}_seq')::TEXT, 4, '0');
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER {table_name}_number_trigger
  BEFORE INSERT ON {table_name}
  FOR EACH ROW
  EXECUTE FUNCTION generate_{table_name}_number();

-- updated_at 자동 갱신 트리거
CREATE TRIGGER {table_name}_updated_at_trigger
  BEFORE UPDATE ON {table_name}
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();
```

---

### 3.2 기록지 (-지) 테이블 템플릿

```sql
-- ===== {테이블명} (-지 패턴) =====
CREATE TABLE {table_name} (
  -- 기본키
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),

  -- 문서번호
  {document_number} VARCHAR(20) UNIQUE NOT NULL,

  -- 참조 문서 ("참조:" 패턴)
  {referenced_document_id} UUID NOT NULL,

  -- 업무 필드
  {field1_name} {data_type} NOT NULL,
  {field2_name} {data_type},

  -- 이력 추적 (-지 패턴 필수)
  history JSONB DEFAULT '[]'::JSONB,

  -- 메타데이터
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  created_by UUID NOT NULL,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_by UUID,

  -- 외래키
  FOREIGN KEY ({referenced_document_id}) REFERENCES {referenced_table}(id),
  FOREIGN KEY (created_by) REFERENCES users(id)
);

-- 인덱스
CREATE INDEX idx_{table_name}_{referenced_document_id}
  ON {table_name}({referenced_document_id});
CREATE INDEX idx_{table_name}_created_at ON {table_name}(created_at);

-- 이력 추적 트리거
CREATE OR REPLACE FUNCTION track_{table_name}_history()
RETURNS TRIGGER AS $$
BEGIN
  NEW.history = NEW.history || jsonb_build_object(
    'timestamp', CURRENT_TIMESTAMP,
    'user_id', NEW.updated_by,
    'changes', jsonb_build_object(
      {field_tracking_logic}
    )
  );
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER {table_name}_history_trigger
  BEFORE UPDATE ON {table_name}
  FOR EACH ROW
  EXECUTE FUNCTION track_{table_name}_history();
```

---

### 3.3 1:N 관계 테이블 템플릿

```sql
-- ===== {테이블명}_items (1:N 관계) =====
-- "여러 개 입력" = 1:N 관계 테이블

CREATE TABLE {parent_table}_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  {parent_table}_id UUID NOT NULL,

  -- 항목 필드
  {field1_name} {data_type} NOT NULL,
  {field2_name} {data_type},
  {field3_name} {data_type},

  -- 순서 (UI 표시용)
  display_order INTEGER DEFAULT 0,

  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,

  FOREIGN KEY ({parent_table}_id) REFERENCES {parent_table}(id) ON DELETE CASCADE
);

-- 인덱스
CREATE INDEX idx_{parent_table}_items_{parent_table}_id
  ON {parent_table}_items({parent_table}_id);
```

---

## 04. API 코드 템플릿

### 4.1 Express 라우터 템플릿

```typescript
// routes/{resource}.ts
import { Router } from 'express';
import { authenticate, authorize } from '../middleware/auth';
import * as controller from '../controllers/{resource}';
import { validate } from '../middleware/validation';
import { create{Resource}Schema, update{Resource}Schema } from '../schemas/{resource}';

const router = Router();

// ===== CRUD =====
router.get('/', authenticate, controller.list);
router.get('/:id', authenticate, controller.get);
router.post('/', authenticate, validate(create{Resource}Schema), controller.create);
router.put('/:id', authenticate, validate(update{Resource}Schema), controller.update);
router.delete('/:id', authenticate, authorize('ADMIN'), controller.remove);

// ===== -서 패턴: 승인 워크플로우 =====
router.post('/:id/submit', authenticate, controller.submit);
router.post('/:id/approve', authenticate, authorize('MANAGER', 'ADMIN'), controller.approve);
router.post('/:id/reject', authenticate, authorize('MANAGER', 'ADMIN'), controller.reject);

// ===== -지 패턴: 이력 조회 =====
router.get('/:id/history', authenticate, controller.getHistory);

// ===== 파일 첨부 =====
router.post('/:id/attachments', authenticate, controller.uploadAttachment);
router.get('/:id/attachments', authenticate, controller.listAttachments);

export { router as {resource}Router };
```

---

### 4.2 컨트롤러 템플릿

```typescript
// controllers/{resource}.ts
import { Request, Response, NextFunction } from 'express';
import { {Resource}Service } from '../services/{resource}-service';

const service = new {Resource}Service();

export const list = async (req: Request, res: Response, next: NextFunction) => {
  try {
    const { status, page = 1, limit = 20 } = req.query;

    const result = await service.list({
      status: status as string,
      page: Number(page),
      limit: Number(limit)
    });

    res.json(result);
  } catch (error) {
    next(error);
  }
};

export const get = async (req: Request, res: Response, next: NextFunction) => {
  try {
    const { id } = req.params;
    const item = await service.findById(id);

    if (!item) {
      return res.status(404).json({ error: 'Not found' });
    }

    res.json(item);
  } catch (error) {
    next(error);
  }
};

export const create = async (req: Request, res: Response, next: NextFunction) => {
  try {
    const item = await service.create({
      ...req.body,
      createdBy: req.user.userId
    });

    res.status(201).json(item);
  } catch (error) {
    next(error);
  }
};

export const update = async (req: Request, res: Response, next: NextFunction) => {
  try {
    const { id } = req.params;
    const item = await service.update(id, {
      ...req.body,
      updatedBy: req.user.userId
    });

    res.json(item);
  } catch (error) {
    next(error);
  }
};

export const submit = async (req: Request, res: Response, next: NextFunction) => {
  try {
    const { id } = req.params;
    const item = await service.submit(id, req.user.userId);

    res.json(item);
  } catch (error) {
    next(error);
  }
};

export const approve = async (req: Request, res: Response, next: NextFunction) => {
  try {
    const { id } = req.params;
    const { comment } = req.body;

    const item = await service.approve(id, req.user.userId, comment);

    res.json(item);
  } catch (error) {
    next(error);
  }
};
```

---

### 4.3 서비스 레이어 템플릿

```typescript
// services/{resource}-service.ts
import { {Resource}, {Resource}Status } from '../models/{resource}';
import { db } from '../database';
import { NotFoundError, BadRequestError } from '../errors';

export class {Resource}Service {
  async create(data: Create{Resource}DTO): Promise<{Resource}> {
    const item = await db.{resources}.create({
      ...data,
      status: {Resource}Status.DRAFT,
      createdAt: new Date()
    });

    return item;
  }

  async findById(id: string): Promise<{Resource} | null> {
    return await db.{resources}.findById(id);
  }

  async list(filters: ListFilters): Promise<ListResult<{Resource}>> {
    const { status, page, limit } = filters;

    const query = db.{resources}.query();

    if (status) {
      query.where('status', status);
    }

    const total = await query.count();
    const data = await query
      .skip((page - 1) * limit)
      .limit(limit)
      .exec();

    return {
      data,
      pagination: {
        page,
        limit,
        total,
        totalPages: Math.ceil(total / limit)
      }
    };
  }

  async update(id: string, data: Update{Resource}DTO): Promise<{Resource}> {
    const item = await this.findById(id);

    if (!item) {
      throw new NotFoundError('{Resource} not found');
    }

    Object.assign(item, data);
    item.updatedAt = new Date();

    await item.save();

    return item;
  }

  async submit(id: string, userId: string): Promise<{Resource}> {
    const item = await this.findById(id);

    if (!item) {
      throw new NotFoundError('{Resource} not found');
    }

    if (item.status !== {Resource}Status.DRAFT) {
      throw new BadRequestError('Only draft items can be submitted');
    }

    item.status = {Resource}Status.SUBMITTED;
    item.submittedAt = new Date();
    item.submittedBy = userId;

    await item.save();

    // 알림
    await this.notifyApprover(item);

    return item;
  }

  async approve(id: string, userId: string, comment?: string): Promise<{Resource}> {
    const item = await this.findById(id);

    if (!item) {
      throw new NotFoundError('{Resource} not found');
    }

    if (item.status !== {Resource}Status.SUBMITTED) {
      throw new BadRequestError('Only submitted items can be approved');
    }

    item.status = {Resource}Status.APPROVED;
    item.approvedAt = new Date();
    item.approvedBy = userId;
    item.approvalComment = comment;

    await item.save();

    // 알림
    await this.notifyCreator(item);

    return item;
  }

  private async notifyApprover(item: {Resource}): Promise<void> {
    // 알림 로직
  }

  private async notifyCreator(item: {Resource}): Promise<void> {
    // 알림 로직
  }
}
```

---

## 05. 프로젝트 구조 템플릿

### 5.1 전체 디렉터리 구조

```
{project-name}/
├── src/
│   ├── config/
│   │   ├── database.ts
│   │   ├── jwt.ts
│   │   └── app.ts
│   ├── models/
│   │   ├── user.ts
│   │   ├── {resource1}.ts
│   │   └── {resource2}.ts
│   ├── controllers/
│   │   ├── auth.ts
│   │   ├── {resource1}.ts
│   │   └── {resource2}.ts
│   ├── services/
│   │   ├── {resource1}-service.ts
│   │   └── {resource2}-service.ts
│   ├── routes/
│   │   ├── auth.ts
│   │   ├── {resource1}.ts
│   │   └── {resource2}.ts
│   ├── middleware/
│   │   ├── auth.ts
│   │   ├── validation.ts
│   │   └── error-handler.ts
│   ├── schemas/
│   │   ├── {resource1}.ts
│   │   └── {resource2}.ts
│   ├── database/
│   │   ├── migrations/
│   │   │   ├── 001_create_users.sql
│   │   │   ├── 002_create_{resource1}.sql
│   │   │   └── 003_create_{resource2}.sql
│   │   └── seeds/
│   │       └── 001_initial_data.sql
│   ├── utils/
│   │   ├── logger.ts
│   │   └── errors.ts
│   └── server.ts
├── tests/
│   ├── unit/
│   │   └── services/
│   └── integration/
│       └── api/
├── docs/
│   ├── api.md
│   ├── database.md
│   └── deployment.md
├── .env.example
├── .gitignore
├── package.json
├── tsconfig.json
└── README.md
```

---

### 5.2 package.json 템플릿

```json
{
  "name": "{project-name}",
  "version": "1.0.0",
  "description": "{project description}",
  "main": "dist/server.js",
  "scripts": {
    "dev": "nodemon src/server.ts",
    "build": "tsc",
    "start": "node dist/server.js",
    "test": "jest",
    "test:watch": "jest --watch",
    "lint": "eslint src/**/*.ts",
    "lint:fix": "eslint src/**/*.ts --fix",
    "migrate": "node scripts/migrate.js",
    "seed": "node scripts/seed.js"
  },
  "dependencies": {
    "express": "^4.18.0",
    "pg": "^8.11.0",
    "jsonwebtoken": "^9.0.0",
    "bcryptjs": "^2.4.3",
    "cors": "^2.8.5",
    "helmet": "^7.0.0",
    "dotenv": "^16.0.0",
    "joi": "^17.9.0"
  },
  "devDependencies": {
    "@types/express": "^4.17.17",
    "@types/node": "^20.0.0",
    "@types/pg": "^8.10.0",
    "@types/jsonwebtoken": "^9.0.2",
    "@types/bcryptjs": "^2.4.2",
    "@types/cors": "^2.8.13",
    "@typescript-eslint/eslint-plugin": "^5.60.0",
    "@typescript-eslint/parser": "^5.60.0",
    "eslint": "^8.43.0",
    "jest": "^29.5.0",
    "nodemon": "^2.0.22",
    "supertest": "^6.3.3",
    "ts-jest": "^29.1.0",
    "ts-node": "^10.9.1",
    "typescript": "^5.1.3"
  }
}
```

---

### 5.3 .env.example 템플릿

```bash
# Application
NODE_ENV=development
PORT=3000

# Database
DB_HOST=localhost
DB_PORT=5432
DB_NAME={database_name}
DB_USER={username}
DB_PASSWORD={password}

# JWT
JWT_SECRET={your_secret_key}
JWT_EXPIRES_IN=1h
JWT_REFRESH_EXPIRES_IN=7d

# File Upload
UPLOAD_DIR=./uploads
MAX_FILE_SIZE=10485760  # 10MB in bytes

# CORS
CORS_ORIGIN=http://localhost:3000

# Logging
LOG_LEVEL=info
```

---

### 5.4 tsconfig.json 템플릿

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "moduleResolution": "node",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "tests"]
}
```

---

## 사용 가이드

### 워크숍 템플릿 사용법

1. **워크숍 준비**:
   - 초대 이메일 템플릿 복사 → 프로젝트 정보 입력 → 발송
   - 체크리스트 출력 → 진행 상황 체크
   - 타임테이블 복사 → 시간 조정 → 공유

2. **문서 양식 활용**:
   - 워크숍에서 도출된 문서명으로 템플릿 선택
   - 항목명, 타입, 필수여부 워크숍 결과로 대체
   - Excel 템플릿으로 현업 검증

3. **데이터베이스 스키마**:
   - 문서 접미사에 맞는 테이블 템플릿 선택
   - `{table_name}`, `{field_name}` 등을 실제 값으로 대체
   - `{PREFIX}`를 문서번호 접두어로 변경 (예: WO, QR)

4. **API 코드**:
   - `{resource}`, `{Resource}` 등을 실제 리소스명으로 대체
   - 필요한 엔드포인트만 선택적으로 구현
   - 접미사별 패턴 확인 후 적용

5. **프로젝트 구조**:
   - 디렉터리 구조 템플릿 복사
   - `{project-name}`, `{resource1}` 등 실제 이름으로 변경
   - package.json, tsconfig.json 설정 조정

---

**관련 문서**:
- [워크숍 준비 가이드](../04-workshop/preparation.md)
- [워크숍 산출물 관리](../04-workshop/outputs.md)
- [데이터베이스 설계](database-design.md)
- [API 설계 가이드](api-design.md)
