# 사례 연구: 플라스틱 사출 공장 MES 구축

## 프로젝트 개요

| 항목 | 내용 |
|------|------|
| **산업** | 플라스틱 사출 제조 |
| **규모** | 직원 30명, 사출기 8대 |
| **기간** | 4개월 (2023.03 ~ 2023.06) |
| **예산** | ₩35,000,000 |
| **팀** | 개발 2명, 현업 5명 |

### 도입 배경

**문제점**:
- 📄 종이 작업지시서 (분실, 오기재)
- 📊 Excel 품질관리 (실시간 불가)
- 📞 구두 지시 (누락, 오해)
- 불량률 3.2%, 정보 검색 60분/일

**목표**:
- 실시간 생산 현황 파악
- 품질 데이터 자동 집계
- 불량률 1% 이하 달성

---

## Formology 적용 과정

### Phase 1: 워크숍 (1일, 2시간)

#### FormType 도출

**Step 1: FormType 브레인스토밍 (30분)**

진행자: "눈손이가 되었다고 상상하세요. 오직 문서로만 일해야 합니다."

```
도출된 22개 FormType:
작업지시서, 자재요청서, 입고확인서, 일일생산일지,
품질검사의뢰서, 검사기록지, 합격확인서, 불량기록지,
금형정비의뢰서, 정비일지, 시정조치서, 예방조치서,
출하지시서, 출하확인서, 월별생산현황표, 불량이력록,
재고현황표, 작업표준서, 검사기준서, 설비가동일지,
가동률현황표, 재작업의뢰서
```

**Step 2: FormType 그룹핑 (20분)**

```
[요청/의뢰 FormType] 6개: 작업지시서, 자재요청서, 품질검사의뢰서...
[기록 FormType] 8개: 일일생산일지, 검사기록지, 불량기록지...
[보고/명세 FormType] 2개: 시정조치서, 예방조치서
[표/현황 FormType] 3개: 월별생산현황표, 재고현황표...
[록/이력 FormType] 1개: 불량이력록
[기준/표준 FormType] 2개: 작업표준서, 검사기준서
```

**Step 3: 우선순위 (15분)**

```
MVP (1순위) - 7개 FormType:
1. 작업지시서
2. 일일생산일지
3. 품질검사의뢰서
4. 검사기록지
5. 불량기록지
6. 합격확인서
7. 월별생산현황표
```

**Step 4: 주요 Form 구조 스케치 (40분)**

**FormType: "작업지시서" → Form 구조**:
```
┌──────────────────────────────┐
│ [Main Section] 지시정보      │
├──────────────────────────────┤
│ 지시번호: [자동] Field       │
│ 제품명: [Selection] Field    │ ← Reference Section (products FK)
│ 수량: [Numeric] Field        │
│ 사출기: [Selection] Field    │ ← Reference Section (machines FK)
│ 금형: [Selection] Field      │ ← Reference Section (molds FK)
├──────────────────────────────┤
│ [Child Section] 자재목록:    │ ← 1:N
│ - 원료 [Selection] [수량]    │
│ + 추가                       │
├──────────────────────────────┤
│ [Reference Section] 작업기준 │
│ - 작업표준서: [참조] Field   │ ← 최신 버전 (FK)
│ - 검사기준서: [참조] Field   │
└──────────────────────────────┘
```

**FormType: "일일생산일지" → Form 구조**:
```
┌──────────────────────────────┐
│ [Main Section] 생산정보      │
├──────────────────────────────┤
│ 작업지시서: [Reference]      │ ← Reference Section (work_orders FK)
│ 작업자: [Selection] Field    │
│ 시간: [Time] ~ [Time] Field  │
├──────────────────────────────┤
│ [Main Section] 생산실적      │
│ - 양품: [Numeric] Field      │
│ - 불량: [Numeric] Field      │
├──────────────────────────────┤
│ [Child Section] 불량유형:    │ ← M:N
│   [Checklist] Fields         │
├──────────────────────────────┤
│ [Main Section] 비고          │
│ 특이사항: [TextArea] Field   │
│ 첨부: [FileUpload] Field     │ ← 1:N files
└──────────────────────────────┘
```

**Section → Entity 사상 자동 도출**:
```
FormType: "작업지시서" → Main Section → work_orders
  ├─ Child Section → work_order_materials (1:N)
  ├─ Reference Section Field → products (FK)
  ├─ Reference Section Field → machines (FK)
  └─ Reference Section Field → work_standards (FK, latest)

FormType: "일일생산일지" → Main Section → production_logs
  ├─ Reference Section → work_orders (FK)
  ├─ Child Section → defect_occurrences (M:N)
  └─ FileUpload Field → production_attachments (1:N)

FormType: "품질검사의뢰서" → Main Section → qc_requests
  ├─ Reference Section → production_logs (FK)
  ├─ Reference Section → inspection_standards (FK)
  └─ Document Flow → inspection_records (1:1)
```

**워크숍 성과**:
- ✅ 22개 FormType 도출 완료
- ✅ 7개 MVP FormType 선정
- ✅ 4개 주요 Form 구조 (Section/Field) 스케치
- ✅ Section → Entity 사상으로 ERD 초안 자동 도출
- ✅ **소요 시간: 2시간 15분**

---

### Phase 2: 개발 (3개월)

#### Sprint 1-2 (4주): 기본 기능

**구현**:
```typescript
// 기술 스택
Backend: Node.js + TypeScript + Express
Database: PostgreSQL 14
Frontend: React + TypeScript + Material-UI

// 기본 CRUD + 관계
interface WorkOrder {
  id: string;
  productId: string;      // FK
  machineId: string;      // FK
  standardId: string;     // FK (latest)
}

interface ProductionLog {
  id: string;
  workOrderId: string;    // FK 필수
  productName: string;    // copied from work_order
}

// 워크플로우 자동화
async function completeProductionLog(logId: string) {
  const log = await ProductionLog.findById(logId);
  log.status = 'COMPLETED';

  // 품질검사의뢰서 자동 생성
  if (log.requiresQC) {
    await QCRequest.create({
      productionLogId: log.id,
      lotNumber: generateLotNumber(),
      status: 'PENDING'
    });
    await notify('quality-team');
  }
}
```

**결과**:
- 7개 문서 CRUD 완성
- 문서 간 관계 구현
- 기본 워크플로우 자동화

#### Sprint 3 (2주): 패턴 발견

**개발자가 자연스럽게 발견한 패턴**:

```typescript
// 패턴 1: "의뢰서" = 승인 필요
abstract class RequestDocument {
  status: 'DRAFT' | 'SUBMITTED' | 'APPROVED';
  async submit() { ... }
  async approve() { ... }
}

class WorkOrder extends RequestDocument { }
class QCRequest extends RequestDocument { }

// 패턴 2: "기록지" = 이력 추적
abstract class RecordDocument {
  history: AuditLog[];
  async save() { ... }
}

// 패턴 3: "일지" = 시간순 정렬
abstract class DailyDocument {
  date: Date;
  static async findByDate() { ... }
}

// 패턴 4: "표" = 집계/통계
abstract class SummaryDocument {
  period: { start: Date; end: Date };
  abstract async generate();
}
```

**효과**:
- 코드 재사용성 80% 향상
- 새 문서 추가 70% 단축
- 일관된 UX

**중요**: 현업은 이런 패턴을 전혀 모름. 그냥 "의뢰서", "기록지"일 뿐.

#### Sprint 4-6 (6주): 2차 기능

- 작업표준서, 검사기준서 (버전 관리)
- 통계 화면 및 대시보드
- 모바일 최적화
- 바코드 스캔 기능

---

### Phase 3: 운영 (2023.07~)

**도입 과정**:
```
Week 1: 관리자 교육 (2시간)
Week 2: 현장 직원 교육 (3시간 × 2회)
Week 3: 병행 운영 (종이 + 시스템)
Week 4: 완전 전환
```

**피드백**:
- ✅ "우리 용어 그대로라 이해 빠름"
- ✅ "작업지시서 찾는 시간 90% 단축"
- ⚠️ "모바일 입력 불편" → 즉시 개선
- ⚠️ "바코드 스캔 필요" → 2주 만에 추가

---

## 성과 측정

### 정량적 성과

| 지표 | 도입 전 | 도입 후 | 개선 |
|------|---------|---------|------|
| **불량률** | 3.2% | 0.8% | **75% ↓** |
| **정보 검색** | 60분/일 | 5분/일 | **92% ↓** |
| **현황 파악** | 4시간 | 실시간 | **100% ↑** |
| **문서 작성** | 30분/건 | 5분/건 | **83% ↓** |
| **데이터 정확도** | 70% | 95%+ | **36% ↑** |

**ROI**:
```
투자: ₩35,000,000

연간 효과:
- 불량 감소: ₩45,000,000
- 시간 절약: ₩18,000,000
- 재고 최적화: ₩8,000,000

총: ₩71,000,000/년
ROI: 203% (첫 해)
회수 기간: 6개월
```

### 정성적 성과

**현업**:
> "우리가 쓰던 용어 그대로라 적응이 빨랐어요." - 생산 관리자

> "품질검사 결과를 실시간으로 보니 즉시 대응 가능해요." - 품질 담당자

> "최신 작업표준서를 자동으로 보여주니 구버전 실수가 없어졌어요." - 현장 반장

**경영진**:
> "실시간 전체 현황이 이렇게 강력할 줄 몰랐습니다." - 공장장

**개발팀**:
> "Formology 덕분에 현업과 소통이 쉬웠어요. Entity 설명 안 해도 되니까." - 개발자

---

## 성공 요인

### 1. 현업 주도 설계

```
전통 방식:
개발자: "Entity 모델링..."
현업: "???"
→ 3개월 뒤 이해 못함

Formology:
현업: "우리는 이런 문서 씁니다"
개발자: "네, 그대로 만들겠습니다"
→ 2시간 만에 합의
```

### 2. 점진적 복잡도

```
Phase 1: 단순 CRUD → 현업 즉시 이해
Phase 2: 자동화 추가 → 편리함만 느낌
Phase 3: 패턴 적용 → 현업은 모름, 개발자만 재사용
```

### 3. 즉시 검증

```
워크숍 종료:
"이 문서 목록 맞아요?" → "네!" (100% 합의)

1개월 후:
"화면이 우리 양식이랑 똑같네요" → 추가 설명 불필요
```

### 4. 자연스러운 진화

```
초기: 22개 문서
1개월: +2개 (현업 요청)
3개월: +3개 (자동화)
→ 강요 없이 필요 시 추가
```

---

## 교훈

### 잘된 점 ✅

1. **워크숍 충실 이행**
   - 2시간 → 3개월 단축

2. **MVP 우선순위 엄수**
   - 7개만 1차 개발

3. **패턴 내부화**
   - 현업에 노출 안 함

4. **즉시 피드백 반영**
   - 모바일 → 2주 해결
   - 바코드 → 1주 POC

### 아쉬운 점 ⚠️

1. **모바일 초기 미고려**
   - 현장은 태블릿 선호

2. **바코드 늦은 도입**
   - 워크숍에서 명시 질문 필요

3. **통계 우선순위 낮춤**
   - 경영진에게 매우 중요

### 개선 사항 🔧

**워크숍 추가 질문**:
- "어떤 기기로 사용?" (PC/모바일/태블릿)
- "입력 방식?" (타이핑/스캔/음성)
- "누가 주로 봄?" (현장/관리자/경영진)

---

## 결론

### 핵심 성과

1. ✅ **현업 언어 그대로** → 이해도 100%
2. ✅ **2시간 워크숍** → 3개월 단축
3. ✅ **점진적 진화** → 지속 개선
4. ✅ **패턴 내재화** → 재사용성 80% ↑

### 측정 성과

- **불량률 75% 감소** (3.2% → 0.8%)
- **검색 92% 단축** (60분 → 5분)
- **ROI 203%** (첫 해)
- **회수 6개월**

이것은 **이론이 아닌 실전 결과**입니다.

Formology는 **제조업 디지털 전환의 실용적 해법**임을 증명했습니다.

---

## 프로젝트 타임라인

```
2023.03.10  워크숍 (2시간)
2023.03.13  개발 시작
2023.04.10  Sprint 1-2 완료
2023.05.08  Sprint 3-4 완료
2023.06.12  Sprint 5-6 완료
2023.06.19  교육 및 전환
2023.07.01  정식 운영
2023.09.30  성과 측정

결과: 모든 목표 달성 ✅
```

---

## 더 읽을거리

- [워크숍 진행 가이드](../04-workshop/facilitation-guide.md) - 실전 워크숍
- [데이터 모델 도출](../02-methodology/data-modeling.md) - ERD 변환
- [개념적 사상 원리](../05-realization/conceptual-mapping.md) - 구현 원리
