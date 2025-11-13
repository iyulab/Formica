# API 설계 가이드

## 개요

Formology 워크숍에서 도출된 FormType 목록을 RESTful API 또는 GraphQL API로 변환하는 실전 가이드입니다.

### 핵심 원칙

```
FormType (서식) = API 리소스 (Resource)
Form Section/Field = API 스키마 (Schema)
Document (문서) = 리소스 인스턴스 (Resource Instance)
Document 흐름 = API 엔드포인트 조합 (Endpoint Composition)
FormType 접미사 = API 동작 패턴 (Behavior Pattern)
```

### 설계 결과

```
Input:  워크숍 산출물
- FormType 목록 15~30개
- Form 구조 (Section/Field)
- Document 흐름도

Output:
- REST API 명세 (OpenAPI 3.0)
- GraphQL 스키마
- 인증/인가 정책
- 실제 구현 예제 (Node.js/TypeScript)

시간: 1~2일
```

---

## REST API 설계

### 1. FormType → REST 리소스 매핑

#### 기본 규칙

**FormType명 → 리소스 경로**:
```
FormType: "작업지시서" → /api/v1/work-orders
FormType: "품질검사보고서" → /api/v1/quality-reports
FormType: "설비점검일지" → /api/v1/equipment-inspections
```

**FormType 접미사별 엔드포인트 패턴 (Document CRUD)**:

```yaml
-서 (書, 요청/의뢰 FormType):
  POST   /resource          # Document 신규 작성
  PUT    /resource/:id      # Document 수정
  POST   /resource/:id/submit   # Document 제출
  POST   /resource/:id/approve  # Document 승인
  POST   /resource/:id/reject   # Document 반려
  GET    /resource/:id      # Document 조회
  GET    /resource          # Document 목록

-지 (紙, 기록 FormType):
  POST   /resource          # Document 신규 기록
  PUT    /resource/:id      # Document 수정
  GET    /resource/:id/history  # Document 이력 조회
  GET    /resource/:id      # Document 조회
  GET    /resource          # Document 목록

-표 (表, 집계 FormType):
  GET    /resource          # Document 집계 결과 조회
  GET    /resource/export   # Excel/PDF 다운로드

-록 (錄, 목록 FormType):
  POST   /resource          # Document 신규 등록
  PUT    /resource/:id      # Document 수정
  DELETE /resource/:id      # Document 삭제
  GET    /resource/:id      # Document 조회
  GET    /resource          # Document 목록 (필터링, 페이징)
```

---

### 2. 제조 MES 실전 예제

#### 2.1 FormType: "작업지시서" API

**리소스**: `/api/v1/work-orders`
**개념**: FormType → API 리소스, Document → 리소스 인스턴스

**Form 구조 (Section/Field) → API 스키마**:
```
[Main Section] 지시정보
- 제품명 [Reference Section Field] → productId (FK)
- 수량 [Numeric Field] → targetQuantity
- 납기일 [Date Field] → dueDate
```

**엔드포인트 설계**:

```typescript
// ===== 1. Document 신규 작성 =====
POST /api/v1/work-orders
Content-Type: application/json
Authorization: Bearer {token}

Request: (Form Field → Request Body)
{
  "productId": "uuid",              // Reference Section Field
  "machineId": "uuid",              // Reference Section Field
  "targetQuantity": 1000,           // Main Section Field [Numeric]
  "dueDate": "2024-11-20",          // Main Section Field [Date]
  "priority": "HIGH",               // Main Section Field [Selection]
  "notes": "긴급 생산"               // Main Section Field [Text]
}

Response: 201 Created (Document 생성)
{
  "id": "uuid",                     // Document ID
  "orderNumber": "WO20241105001",   // Document 번호
  "productId": "uuid",
  "machineId": "uuid",
  "targetQuantity": 1000,
  "dueDate": "2024-11-20",
  "priority": "HIGH",
  "status": "DRAFT",                // Document 상태
  "createdAt": "2024-11-05T10:00:00Z",
  "createdBy": "user-uuid"
}

// ===== 2. 제출 (승인 요청) =====
POST /api/v1/work-orders/{id}/submit
Authorization: Bearer {token}

Response: 200 OK
{
  "id": "uuid",
  "status": "SUBMITTED",
  "submittedAt": "2024-11-05T10:30:00Z",
  "submittedBy": "user-uuid"
}

// ===== 3. 승인 =====
POST /api/v1/work-orders/{id}/approve
Authorization: Bearer {token}

Request:
{
  "comment": "승인합니다"
}

Response: 200 OK
{
  "id": "uuid",
  "status": "APPROVED",
  "approvedAt": "2024-11-05T11:00:00Z",
  "approvedBy": "manager-uuid",
  "approvalComment": "승인합니다"
}

// ===== 4. 목록 조회 (필터링) =====
GET /api/v1/work-orders?status=APPROVED&dueDate=2024-11-20&page=1&limit=20
Authorization: Bearer {token}

Response: 200 OK
{
  "data": [
    {
      "id": "uuid",
      "orderNumber": "WO20241105001",
      "product": {
        "id": "uuid",
        "name": "제품A",
        "code": "P001"
      },
      "machine": {
        "id": "uuid",
        "name": "설비1호기"
      },
      "targetQuantity": 1000,
      "status": "APPROVED",
      "dueDate": "2024-11-20"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 45,
    "totalPages": 3
  }
}
```

---

#### 2.2 품질검사보고서 API

**리소스**: `/api/v1/quality-reports`

**검사 항목 = 1:N 관계 = 중첩 리소스**:

```typescript
// ===== 1. 검사 보고서 작성 (검사 항목 포함) =====
POST /api/v1/quality-reports
Authorization: Bearer {token}

Request:
{
  "workOrderId": "uuid",
  "inspector": "검사자A",
  "inspectionDate": "2024-11-05",
  "items": [
    {
      "name": "치수 검사",
      "specification": "100±0.5mm",
      "measured": "100.2mm",
      "result": "OK"
    },
    {
      "name": "표면 검사",
      "specification": "긁힘 없음",
      "measured": "양호",
      "result": "OK"
    },
    {
      "name": "중량 검사",
      "specification": "500±5g",
      "measured": "506g",
      "result": "NG",
      "ngReason": "중량 초과"
    }
  ],
  "overallResult": "NG",
  "ngAction": "재작업 지시"
}

Response: 201 Created
{
  "id": "uuid",
  "reportNumber": "QR20241105001",
  "workOrderId": "uuid",
  "inspector": "검사자A",
  "inspectionDate": "2024-11-05",
  "items": [
    {
      "id": "item-uuid-1",
      "name": "치수 검사",
      "result": "OK"
    },
    {
      "id": "item-uuid-2",
      "name": "표면 검사",
      "result": "OK"
    },
    {
      "id": "item-uuid-3",
      "name": "중량 검사",
      "result": "NG"
    }
  ],
  "overallResult": "NG",
  "createdAt": "2024-11-05T14:00:00Z"
}

// ===== 2. 이력 조회 (-지 패턴) =====
GET /api/v1/quality-reports/{id}/history
Authorization: Bearer {token}

Response: 200 OK
{
  "reportId": "uuid",
  "history": [
    {
      "version": 1,
      "timestamp": "2024-11-05T14:00:00Z",
      "userId": "user-uuid",
      "changes": {
        "overallResult": "NG",
        "items": [...]
      }
    },
    {
      "version": 2,
      "timestamp": "2024-11-05T15:30:00Z",
      "userId": "user-uuid",
      "changes": {
        "ngAction": "재작업 지시 → 재검사 완료"
      }
    }
  ]
}
```

---

#### 2.3 생산현황표 API (-표 패턴)

**리소스**: `/api/v1/production-status`

**집계 = Read-Only = GET만 제공**:

```typescript
// ===== 1. 일일 생산 현황 =====
GET /api/v1/production-status/daily?date=2024-11-05
Authorization: Bearer {token}

Response: 200 OK
{
  "date": "2024-11-05",
  "summary": {
    "totalOrders": 12,
    "completedOrders": 8,
    "inProgressOrders": 4,
    "totalProduced": 8500,
    "totalDefects": 120,
    "defectRate": 1.41
  },
  "byMachine": [
    {
      "machineId": "uuid",
      "machineName": "설비1호기",
      "orders": 3,
      "produced": 2500,
      "defects": 30,
      "utilizationRate": 85.5
    },
    {
      "machineId": "uuid",
      "machineName": "설비2호기",
      "orders": 5,
      "produced": 4000,
      "defects": 60,
      "utilizationRate": 92.3
    }
  ],
  "byProduct": [
    {
      "productId": "uuid",
      "productName": "제품A",
      "produced": 3000,
      "defects": 40
    }
  ]
}

// ===== 2. Excel 다운로드 =====
GET /api/v1/production-status/daily/export?date=2024-11-05&format=xlsx
Authorization: Bearer {token}

Response: 200 OK
Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
Content-Disposition: attachment; filename="production-status-20241105.xlsx"

[Binary Excel Data]
```

---

### 3. 인증 및 인가

#### 3.1 JWT 기반 인증

```typescript
// ===== 로그인 =====
POST /api/v1/auth/login
Content-Type: application/json

Request:
{
  "username": "user@example.com",
  "password": "password123"
}

Response: 200 OK
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiresIn": 3600,
  "user": {
    "id": "uuid",
    "username": "user@example.com",
    "name": "홍길동",
    "role": "OPERATOR"
  }
}

// ===== 토큰 갱신 =====
POST /api/v1/auth/refresh
Content-Type: application/json

Request:
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}

Response: 200 OK
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiresIn": 3600
}
```

---

#### 3.2 역할 기반 접근 제어 (RBAC)

**문서 접미사별 권한**:

```yaml
작업지시서 (-서):
  DRAFT:
    - 작성자: 수정, 삭제
  SUBMITTED:
    - 승인자: 승인, 반려
  APPROVED:
    - 모두: 읽기 전용

품질검사보고서 (-지):
  - 검사자: 작성, 수정
  - 품질관리자: 모든 작업
  - 일반 사용자: 읽기 전용

생산현황표 (-표):
  - 모두: 읽기 전용
  - 관리자: Excel 다운로드
```

**구현 예제**:

```typescript
// middleware/auth.ts
import { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';

interface JWTPayload {
  userId: string;
  role: string;
}

export const authenticate = (req: Request, res: Response, next: NextFunction) => {
  const token = req.headers.authorization?.split(' ')[1];

  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }

  try {
    const payload = jwt.verify(token, process.env.JWT_SECRET!) as JWTPayload;
    req.user = payload;
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
};

export const authorize = (...roles: string[]) => {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    next();
  };
};

// 사용 예제
router.post('/work-orders/:id/approve',
  authenticate,
  authorize('MANAGER', 'ADMIN'),
  approveWorkOrder
);
```

---

### 4. 파일 첨부 처리

**"붙임:" = 파일 업로드 API**:

```typescript
// ===== 1. 파일 업로드 =====
POST /api/v1/work-orders/{id}/attachments
Content-Type: multipart/form-data
Authorization: Bearer {token}

Request (Form Data):
file: [binary]
description: "설계 도면"

Response: 201 Created
{
  "id": "attachment-uuid",
  "workOrderId": "uuid",
  "filename": "design-drawing.pdf",
  "originalName": "설계도면_v2.pdf",
  "mimeType": "application/pdf",
  "size": 2048576,
  "description": "설계 도면",
  "uploadedAt": "2024-11-05T10:00:00Z",
  "uploadedBy": "user-uuid",
  "url": "/api/v1/attachments/attachment-uuid/download"
}

// ===== 2. 파일 다운로드 =====
GET /api/v1/attachments/{id}/download
Authorization: Bearer {token}

Response: 200 OK
Content-Type: application/pdf
Content-Disposition: attachment; filename="설계도면_v2.pdf"

[Binary PDF Data]

// ===== 3. 첨부 파일 목록 =====
GET /api/v1/work-orders/{id}/attachments
Authorization: Bearer {token}

Response: 200 OK
{
  "workOrderId": "uuid",
  "attachments": [
    {
      "id": "attachment-uuid-1",
      "filename": "design-drawing.pdf",
      "description": "설계 도면",
      "size": 2048576,
      "uploadedAt": "2024-11-05T10:00:00Z"
    },
    {
      "id": "attachment-uuid-2",
      "filename": "inspection-photo.jpg",
      "description": "검사 사진",
      "size": 512000,
      "uploadedAt": "2024-11-05T11:30:00Z"
    }
  ]
}
```

---

### 5. Node.js 구현 예제

#### 5.1 Express + TypeScript 기본 구조

```typescript
// server.ts
import express from 'express';
import cors from 'cors';
import helmet from 'helmet';
import { workOrderRouter } from './routes/work-orders';
import { qualityReportRouter } from './routes/quality-reports';
import { errorHandler } from './middleware/error-handler';

const app = express();

// 미들웨어
app.use(helmet());
app.use(cors());
app.use(express.json());

// 라우트
app.use('/api/v1/work-orders', workOrderRouter);
app.use('/api/v1/quality-reports', qualityReportRouter);

// 에러 핸들러
app.use(errorHandler);

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

---

#### 5.2 작업지시서 라우터

```typescript
// routes/work-orders.ts
import { Router } from 'express';
import { authenticate, authorize } from '../middleware/auth';
import * as controller from '../controllers/work-orders';
import { validate } from '../middleware/validation';
import { createWorkOrderSchema } from '../schemas/work-order';

const router = Router();

// 목록 조회
router.get('/',
  authenticate,
  controller.list
);

// 상세 조회
router.get('/:id',
  authenticate,
  controller.get
);

// 신규 작성
router.post('/',
  authenticate,
  validate(createWorkOrderSchema),
  controller.create
);

// 수정
router.put('/:id',
  authenticate,
  controller.update
);

// 제출
router.post('/:id/submit',
  authenticate,
  controller.submit
);

// 승인
router.post('/:id/approve',
  authenticate,
  authorize('MANAGER', 'ADMIN'),
  controller.approve
);

// 반려
router.post('/:id/reject',
  authenticate,
  authorize('MANAGER', 'ADMIN'),
  controller.reject
);

export { router as workOrderRouter };
```

---

#### 5.3 컨트롤러 구현

```typescript
// controllers/work-orders.ts
import { Request, Response, NextFunction } from 'express';
import { WorkOrderService } from '../services/work-order-service';

const service = new WorkOrderService();

export const create = async (req: Request, res: Response, next: NextFunction) => {
  try {
    const workOrder = await service.create({
      ...req.body,
      createdBy: req.user.userId
    });

    res.status(201).json(workOrder);
  } catch (error) {
    next(error);
  }
};

export const submit = async (req: Request, res: Response, next: NextFunction) => {
  try {
    const { id } = req.params;
    const workOrder = await service.submit(id, req.user.userId);

    res.json(workOrder);
  } catch (error) {
    next(error);
  }
};

export const approve = async (req: Request, res: Response, next: NextFunction) => {
  try {
    const { id } = req.params;
    const { comment } = req.body;

    const workOrder = await service.approve(id, req.user.userId, comment);

    res.json(workOrder);
  } catch (error) {
    next(error);
  }
};

export const list = async (req: Request, res: Response, next: NextFunction) => {
  try {
    const { status, dueDate, page = 1, limit = 20 } = req.query;

    const result = await service.list({
      status: status as string,
      dueDate: dueDate as string,
      page: Number(page),
      limit: Number(limit)
    });

    res.json(result);
  } catch (error) {
    next(error);
  }
};
```

---

#### 5.4 서비스 레이어

```typescript
// services/work-order-service.ts
import { WorkOrder, WorkOrderStatus } from '../models/work-order';
import { db } from '../database';

export class WorkOrderService {
  async create(data: CreateWorkOrderDTO): Promise<WorkOrder> {
    const orderNumber = await this.generateOrderNumber();

    const workOrder = await db.workOrders.create({
      ...data,
      orderNumber,
      status: WorkOrderStatus.DRAFT,
      createdAt: new Date()
    });

    return workOrder;
  }

  async submit(id: string, userId: string): Promise<WorkOrder> {
    const workOrder = await db.workOrders.findById(id);

    if (!workOrder) {
      throw new NotFoundError('Work order not found');
    }

    if (workOrder.status !== WorkOrderStatus.DRAFT) {
      throw new BadRequestError('Only draft orders can be submitted');
    }

    workOrder.status = WorkOrderStatus.SUBMITTED;
    workOrder.submittedAt = new Date();
    workOrder.submittedBy = userId;

    await workOrder.save();

    // 승인자에게 알림
    await this.notifyApprover(workOrder);

    return workOrder;
  }

  async approve(id: string, userId: string, comment?: string): Promise<WorkOrder> {
    const workOrder = await db.workOrders.findById(id);

    if (!workOrder) {
      throw new NotFoundError('Work order not found');
    }

    if (workOrder.status !== WorkOrderStatus.SUBMITTED) {
      throw new BadRequestError('Only submitted orders can be approved');
    }

    workOrder.status = WorkOrderStatus.APPROVED;
    workOrder.approvedAt = new Date();
    workOrder.approvedBy = userId;
    workOrder.approvalComment = comment;

    await workOrder.save();

    // 작성자에게 알림
    await this.notifyCreator(workOrder);

    return workOrder;
  }

  async list(filters: ListFilters): Promise<ListResult<WorkOrder>> {
    const { status, dueDate, page, limit } = filters;

    const query = db.workOrders.query();

    if (status) {
      query.where('status', status);
    }

    if (dueDate) {
      query.where('dueDate', dueDate);
    }

    const total = await query.count();
    const data = await query
      .skip((page - 1) * limit)
      .limit(limit)
      .populate(['product', 'machine'])
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

  private async generateOrderNumber(): Promise<string> {
    const today = new Date();
    const dateStr = today.toISOString().slice(0, 10).replace(/-/g, '');
    const sequence = await db.sequences.getNext('work_order');

    return `WO${dateStr}${String(sequence).padStart(4, '0')}`;
  }

  private async notifyApprover(workOrder: WorkOrder): Promise<void> {
    // 알림 로직 (이메일, Slack 등)
  }

  private async notifyCreator(workOrder: WorkOrder): Promise<void> {
    // 알림 로직
  }
}
```

---

### 6. OpenAPI 명세

#### 6.1 OpenAPI 3.0 정의

```yaml
openapi: 3.0.0
info:
  title: 제조 MES API
  version: 1.0.0
  description: Formology 방법론 기반 제조 실행 시스템 API

servers:
  - url: https://api.example.com/v1
    description: Production server
  - url: http://localhost:3000/v1
    description: Development server

components:
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

  schemas:
    WorkOrder:
      type: object
      properties:
        id:
          type: string
          format: uuid
        orderNumber:
          type: string
          example: "WO20241105001"
        productId:
          type: string
          format: uuid
        machineId:
          type: string
          format: uuid
        targetQuantity:
          type: integer
          minimum: 1
        dueDate:
          type: string
          format: date
        status:
          type: string
          enum: [DRAFT, SUBMITTED, APPROVED, REJECTED, IN_PROGRESS, COMPLETED]
        createdAt:
          type: string
          format: date-time
        createdBy:
          type: string
          format: uuid

    CreateWorkOrderRequest:
      type: object
      required:
        - productId
        - machineId
        - targetQuantity
        - dueDate
      properties:
        productId:
          type: string
          format: uuid
        machineId:
          type: string
          format: uuid
        targetQuantity:
          type: integer
          minimum: 1
        dueDate:
          type: string
          format: date
        priority:
          type: string
          enum: [LOW, NORMAL, HIGH, URGENT]
        notes:
          type: string

    Error:
      type: object
      properties:
        error:
          type: string
        message:
          type: string
        details:
          type: object

paths:
  /work-orders:
    get:
      summary: 작업지시서 목록 조회
      security:
        - bearerAuth: []
      parameters:
        - name: status
          in: query
          schema:
            type: string
            enum: [DRAFT, SUBMITTED, APPROVED, REJECTED, IN_PROGRESS, COMPLETED]
        - name: dueDate
          in: query
          schema:
            type: string
            format: date
        - name: page
          in: query
          schema:
            type: integer
            minimum: 1
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            minimum: 1
            maximum: 100
            default: 20
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/WorkOrder'
                  pagination:
                    type: object
                    properties:
                      page:
                        type: integer
                      limit:
                        type: integer
                      total:
                        type: integer
                      totalPages:
                        type: integer

    post:
      summary: 작업지시서 신규 작성
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateWorkOrderRequest'
      responses:
        '201':
          description: Created
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/WorkOrder'
        '400':
          description: Bad Request
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'

  /work-orders/{id}/submit:
    post:
      summary: 작업지시서 제출 (승인 요청)
      security:
        - bearerAuth: []
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
            format: uuid
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/WorkOrder'

  /work-orders/{id}/approve:
    post:
      summary: 작업지시서 승인
      security:
        - bearerAuth: []
      parameters:
        - name: id
          in: path
          required: true
          schema:
            type: string
            format: uuid
      requestBody:
        content:
          application/json:
            schema:
              type: object
              properties:
                comment:
                  type: string
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/WorkOrder'
```

---

## GraphQL API 설계

### 1. 문서 → GraphQL 타입 매핑

#### 1.1 기본 타입 정의

```graphql
# ===== 작업지시서 =====
type WorkOrder {
  id: ID!
  orderNumber: String!
  product: Product!
  machine: Machine!
  targetQuantity: Int!
  dueDate: Date!
  priority: Priority!
  status: WorkOrderStatus!

  # 승인 워크플로우 (-서 패턴)
  submittedAt: DateTime
  submittedBy: User
  approvedAt: DateTime
  approvedBy: User
  approvalComment: String

  # 첨부 파일 ("붙임:" 패턴)
  attachments: [Attachment!]!

  # 메타데이터
  createdAt: DateTime!
  createdBy: User!
  updatedAt: DateTime!
}

enum WorkOrderStatus {
  DRAFT
  SUBMITTED
  APPROVED
  REJECTED
  IN_PROGRESS
  COMPLETED
}

enum Priority {
  LOW
  NORMAL
  HIGH
  URGENT
}

# ===== 품질검사보고서 =====
type QualityReport {
  id: ID!
  reportNumber: String!
  workOrder: WorkOrder!
  inspector: String!
  inspectionDate: Date!

  # 검사 항목 (1:N 관계)
  items: [InspectionItem!]!

  overallResult: InspectionResult!
  ngAction: String

  # 이력 추적 (-지 패턴)
  history: [AuditLog!]!

  createdAt: DateTime!
  createdBy: User!
}

type InspectionItem {
  id: ID!
  name: String!
  specification: String!
  measured: String!
  result: InspectionResult!
  ngReason: String
}

enum InspectionResult {
  OK
  NG
}

# ===== 생산현황표 =====
type ProductionStatus {
  date: Date!
  summary: ProductionSummary!
  byMachine: [MachineProduction!]!
  byProduct: [ProductProduction!]!
}

type ProductionSummary {
  totalOrders: Int!
  completedOrders: Int!
  inProgressOrders: Int!
  totalProduced: Int!
  totalDefects: Int!
  defectRate: Float!
}
```

---

#### 1.2 Query 정의

```graphql
type Query {
  # ===== 작업지시서 =====
  workOrder(id: ID!): WorkOrder
  workOrders(
    status: WorkOrderStatus
    dueDate: Date
    page: Int = 1
    limit: Int = 20
  ): WorkOrderConnection!

  # ===== 품질검사보고서 =====
  qualityReport(id: ID!): QualityReport
  qualityReports(
    workOrderId: ID
    inspectionDate: Date
    result: InspectionResult
    page: Int = 1
    limit: Int = 20
  ): QualityReportConnection!

  # ===== 생산현황표 (-표 패턴 = Read-Only) =====
  productionStatus(date: Date!): ProductionStatus!
  productionStatusRange(
    startDate: Date!
    endDate: Date!
  ): [ProductionStatus!]!
}

# 페이지네이션
type WorkOrderConnection {
  edges: [WorkOrderEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type WorkOrderEdge {
  node: WorkOrder!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}
```

---

#### 1.3 Mutation 정의

```graphql
type Mutation {
  # ===== 작업지시서 =====
  createWorkOrder(input: CreateWorkOrderInput!): WorkOrder!
  updateWorkOrder(id: ID!, input: UpdateWorkOrderInput!): WorkOrder!
  submitWorkOrder(id: ID!): WorkOrder!
  approveWorkOrder(id: ID!, comment: String): WorkOrder!
  rejectWorkOrder(id: ID!, reason: String!): WorkOrder!

  # ===== 품질검사보고서 =====
  createQualityReport(input: CreateQualityReportInput!): QualityReport!
  updateQualityReport(id: ID!, input: UpdateQualityReportInput!): QualityReport!

  # ===== 파일 첨부 =====
  uploadAttachment(
    entityType: String!
    entityId: ID!
    file: Upload!
    description: String
  ): Attachment!
}

# Input 타입
input CreateWorkOrderInput {
  productId: ID!
  machineId: ID!
  targetQuantity: Int!
  dueDate: Date!
  priority: Priority = NORMAL
  notes: String
}

input CreateQualityReportInput {
  workOrderId: ID!
  inspector: String!
  inspectionDate: Date!
  items: [InspectionItemInput!]!
  overallResult: InspectionResult!
  ngAction: String
}

input InspectionItemInput {
  name: String!
  specification: String!
  measured: String!
  result: InspectionResult!
  ngReason: String
}
```

---

### 2. GraphQL Resolver 구현

#### 2.1 Query Resolver

```typescript
// resolvers/query.ts
import { WorkOrderService } from '../services/work-order-service';
import { QualityReportService } from '../services/quality-report-service';

const workOrderService = new WorkOrderService();
const qualityReportService = new QualityReportService();

export const Query = {
  workOrder: async (_parent: any, { id }: { id: string }, context: Context) => {
    if (!context.user) {
      throw new AuthenticationError('Not authenticated');
    }

    return await workOrderService.findById(id);
  },

  workOrders: async (
    _parent: any,
    args: { status?: string; dueDate?: string; page: number; limit: number },
    context: Context
  ) => {
    if (!context.user) {
      throw new AuthenticationError('Not authenticated');
    }

    const result = await workOrderService.list(args);

    return {
      edges: result.data.map((node, index) => ({
        node,
        cursor: Buffer.from(`${args.page}:${index}`).toString('base64')
      })),
      pageInfo: {
        hasNextPage: result.pagination.page < result.pagination.totalPages,
        hasPreviousPage: result.pagination.page > 1,
        startCursor: result.data.length > 0
          ? Buffer.from(`${args.page}:0`).toString('base64')
          : null,
        endCursor: result.data.length > 0
          ? Buffer.from(`${args.page}:${result.data.length - 1}`).toString('base64')
          : null
      },
      totalCount: result.pagination.total
    };
  },

  productionStatus: async (
    _parent: any,
    { date }: { date: string },
    context: Context
  ) => {
    if (!context.user) {
      throw new AuthenticationError('Not authenticated');
    }

    return await productionStatusService.getDaily(date);
  }
};
```

---

#### 2.2 Mutation Resolver

```typescript
// resolvers/mutation.ts
export const Mutation = {
  createWorkOrder: async (
    _parent: any,
    { input }: { input: CreateWorkOrderInput },
    context: Context
  ) => {
    if (!context.user) {
      throw new AuthenticationError('Not authenticated');
    }

    return await workOrderService.create({
      ...input,
      createdBy: context.user.userId
    });
  },

  submitWorkOrder: async (
    _parent: any,
    { id }: { id: string },
    context: Context
  ) => {
    if (!context.user) {
      throw new AuthenticationError('Not authenticated');
    }

    return await workOrderService.submit(id, context.user.userId);
  },

  approveWorkOrder: async (
    _parent: any,
    { id, comment }: { id: string; comment?: string },
    context: Context
  ) => {
    if (!context.user) {
      throw new AuthenticationError('Not authenticated');
    }

    // 권한 체크
    if (!['MANAGER', 'ADMIN'].includes(context.user.role)) {
      throw new ForbiddenError('Insufficient permissions');
    }

    return await workOrderService.approve(id, context.user.userId, comment);
  },

  uploadAttachment: async (
    _parent: any,
    { entityType, entityId, file, description }: UploadAttachmentArgs,
    context: Context
  ) => {
    if (!context.user) {
      throw new AuthenticationError('Not authenticated');
    }

    const { createReadStream, filename, mimetype } = await file;

    return await attachmentService.upload({
      entityType,
      entityId,
      stream: createReadStream(),
      filename,
      mimetype,
      description,
      uploadedBy: context.user.userId
    });
  }
};
```

---

#### 2.3 Field Resolver (N+1 문제 해결)

```typescript
// resolvers/work-order.ts
import DataLoader from 'dataloader';

export const WorkOrder = {
  product: async (parent: WorkOrder, _args: any, context: Context) => {
    // DataLoader로 N+1 문제 해결
    return context.loaders.product.load(parent.productId);
  },

  machine: async (parent: WorkOrder, _args: any, context: Context) => {
    return context.loaders.machine.load(parent.machineId);
  },

  attachments: async (parent: WorkOrder, _args: any, context: Context) => {
    return await attachmentService.findByEntity('work_order', parent.id);
  },

  createdBy: async (parent: WorkOrder, _args: any, context: Context) => {
    return context.loaders.user.load(parent.createdBy);
  }
};

// DataLoader 설정
export const createLoaders = () => ({
  product: new DataLoader(async (ids: string[]) => {
    const products = await db.products.findByIds(ids);
    return ids.map(id => products.find(p => p.id === id));
  }),

  machine: new DataLoader(async (ids: string[]) => {
    const machines = await db.machines.findByIds(ids);
    return ids.map(id => machines.find(m => m.id === id));
  }),

  user: new DataLoader(async (ids: string[]) => {
    const users = await db.users.findByIds(ids);
    return ids.map(id => users.find(u => u.id === id));
  })
});
```

---

### 3. Apollo Server 설정

```typescript
// server.ts
import { ApolloServer } from '@apollo/server';
import { expressMiddleware } from '@apollo/server/express4';
import { typeDefs } from './schema';
import { resolvers } from './resolvers';
import { createLoaders } from './resolvers/loaders';
import { getUserFromToken } from './auth';

const server = new ApolloServer({
  typeDefs,
  resolvers,
  formatError: (error) => {
    console.error(error);
    return error;
  }
});

await server.start();

app.use(
  '/graphql',
  cors(),
  express.json(),
  expressMiddleware(server, {
    context: async ({ req }) => {
      const token = req.headers.authorization?.split(' ')[1];
      const user = token ? await getUserFromToken(token) : null;

      return {
        user,
        loaders: createLoaders()
      };
    }
  })
);
```

---

## 고급 패턴

### 1. 버전 관리

**"참조:" = 최신 버전 참조**:

```typescript
// ===== API 엔드포인트 =====
GET /api/v1/quality-reports/{id}/versions
Response:
{
  "currentVersion": 3,
  "versions": [
    {
      "version": 1,
      "createdAt": "2024-11-05T10:00:00Z",
      "data": { ... }
    },
    {
      "version": 2,
      "createdAt": "2024-11-05T11:00:00Z",
      "data": { ... }
    },
    {
      "version": 3,
      "createdAt": "2024-11-05T12:00:00Z",
      "data": { ... }
    }
  ]
}

GET /api/v1/quality-reports/{id}/versions/{version}
Response:
{
  "version": 2,
  "data": { ... }
}

// ===== GraphQL =====
type QualityReport {
  id: ID!
  currentVersion: Int!
  versions: [VersionedQualityReport!]!
}

type VersionedQualityReport {
  version: Int!
  createdAt: DateTime!
  data: QualityReportData!
}

query {
  qualityReport(id: "uuid") {
    versions {
      version
      createdAt
      data {
        overallResult
        items { ... }
      }
    }
  }
}
```

---

### 2. 실시간 업데이트 (WebSocket)

```typescript
// ===== GraphQL Subscription =====
type Subscription {
  workOrderStatusChanged(id: ID!): WorkOrder!
  productionStatusUpdated(date: Date!): ProductionStatus!
}

// Resolver
const Subscription = {
  workOrderStatusChanged: {
    subscribe: withFilter(
      () => pubsub.asyncIterator(['WORK_ORDER_STATUS_CHANGED']),
      (payload, variables) => {
        return payload.workOrderStatusChanged.id === variables.id;
      }
    )
  }
};

// 클라이언트 사용
subscription {
  workOrderStatusChanged(id: "uuid") {
    id
    status
    approvedAt
    approvedBy {
      name
    }
  }
}
```

---

### 3. Batch Operations

```typescript
// ===== REST API =====
POST /api/v1/work-orders/batch
Request:
{
  "operations": [
    {
      "action": "approve",
      "id": "uuid-1",
      "comment": "승인"
    },
    {
      "action": "reject",
      "id": "uuid-2",
      "reason": "불량"
    }
  ]
}

Response:
{
  "results": [
    {
      "id": "uuid-1",
      "status": "success",
      "data": { ... }
    },
    {
      "id": "uuid-2",
      "status": "success",
      "data": { ... }
    }
  ]
}

// ===== GraphQL =====
mutation {
  batchApproveWorkOrders(ids: ["uuid-1", "uuid-2"], comment: "승인") {
    id
    status
  }
}
```

---

## 테스트

### 1. API 테스트

```typescript
// tests/work-orders.test.ts
import request from 'supertest';
import { app } from '../server';

describe('Work Orders API', () => {
  let token: string;
  let workOrderId: string;

  beforeAll(async () => {
    // 로그인하여 토큰 획득
    const res = await request(app)
      .post('/api/v1/auth/login')
      .send({
        username: 'test@example.com',
        password: 'password123'
      });

    token = res.body.accessToken;
  });

  it('should create a work order', async () => {
    const res = await request(app)
      .post('/api/v1/work-orders')
      .set('Authorization', `Bearer ${token}`)
      .send({
        productId: 'product-uuid',
        machineId: 'machine-uuid',
        targetQuantity: 1000,
        dueDate: '2024-11-20',
        priority: 'HIGH'
      });

    expect(res.status).toBe(201);
    expect(res.body.orderNumber).toMatch(/^WO\d{8}\d{4}$/);
    expect(res.body.status).toBe('DRAFT');

    workOrderId = res.body.id;
  });

  it('should submit a work order', async () => {
    const res = await request(app)
      .post(`/api/v1/work-orders/${workOrderId}/submit`)
      .set('Authorization', `Bearer ${token}`);

    expect(res.status).toBe(200);
    expect(res.body.status).toBe('SUBMITTED');
    expect(res.body.submittedAt).toBeDefined();
  });

  it('should approve a work order', async () => {
    const res = await request(app)
      .post(`/api/v1/work-orders/${workOrderId}/approve`)
      .set('Authorization', `Bearer ${token}`)
      .send({
        comment: '승인합니다'
      });

    expect(res.status).toBe(200);
    expect(res.body.status).toBe('APPROVED');
    expect(res.body.approvedAt).toBeDefined();
    expect(res.body.approvalComment).toBe('승인합니다');
  });

  it('should list work orders with filters', async () => {
    const res = await request(app)
      .get('/api/v1/work-orders')
      .query({ status: 'APPROVED', page: 1, limit: 10 })
      .set('Authorization', `Bearer ${token}`);

    expect(res.status).toBe(200);
    expect(res.body.data).toBeInstanceOf(Array);
    expect(res.body.pagination.page).toBe(1);
    expect(res.body.pagination.limit).toBe(10);
  });
});
```

---

## 배포 고려사항

### 1. API 버전 관리

```
/api/v1/work-orders  ← 현재 버전
/api/v2/work-orders  ← 향후 버전

헤더 기반:
API-Version: 1
API-Version: 2
```

---

### 2. 성능 최적화

```yaml
캐싱:
  - Redis 캐시 (집계 데이터, 생산현황표)
  - HTTP 캐시 헤더 (ETag, Cache-Control)

데이터베이스:
  - 인덱스 최적화
  - Connection pooling
  - Read replica

GraphQL:
  - DataLoader (N+1 문제)
  - Query complexity 제한
  - Persisted queries
```

---

### 3. 보안

```yaml
인증:
  - JWT with refresh token
  - Token rotation
  - Rate limiting

인가:
  - RBAC (Role-Based Access Control)
  - Row-level security
  - Field-level permissions

입력 검증:
  - Schema validation (Joi, Zod)
  - SQL injection prevention
  - XSS protection
```

---

### 4. 모니터링

```yaml
로깅:
  - Winston, Pino
  - Structured logging
  - Log aggregation (ELK, Datadog)

메트릭:
  - Response time
  - Error rate
  - API usage

추적:
  - OpenTelemetry
  - Distributed tracing
  - APM (Application Performance Monitoring)
```

---

## 체크리스트

### API 설계 체크리스트

```
□ 문서 → 리소스 매핑 완료
□ 접미사별 엔드포인트 패턴 적용
□ 인증/인가 정책 정의
□ 파일 첨부 처리 구현
□ 버전 관리 전략 수립
□ 에러 핸들링 표준화
□ OpenAPI 명세 작성
□ API 테스트 작성
□ 성능 최적화 (캐싱, 인덱싱)
□ 보안 검토 (SQL injection, XSS)
□ 모니터링 설정
□ 문서화 완료
```

---

**관련 문서**:
- [데이터베이스 설계](database-design.md)
- [워크숍 산출물](../04-workshop/outputs.md)
- [제조 MES 사례](../06-case-studies/manufacturing-mes.md)
