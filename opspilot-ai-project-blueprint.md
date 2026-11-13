# OpsPilot AI --- Project Blueprint

> **เอกสารกำหนดทิศทางโครงการสำหรับ AI Agent และทีมพัฒนา**\
> Version: 1.0\
> Status: Proposed --- ต้องผ่านการทบทวนและอนุมัติก่อนเริ่มพัฒนา\
> Project type: AI-powered Business Intelligence & Decision Support\
> Difficulty: Intermediate\
> Target delivery: 8 weeks (part-time; estimate, not a guarantee)

------------------------------------------------------------------------

## 1. Executive Summary

**OpsPilot AI** คือระบบ AI Operations Copilot สำหรับธุรกิจขนาดเล็กและขนาดกลาง
(SME) ที่ช่วยให้ผู้ใช้งานตรวจสอบ วิเคราะห์
และทำความเข้าใจข้อมูลการขายและสินค้าคงคลังผ่าน Dashboard
และการถามคำถามด้วยภาษาธรรมชาติ

ระบบไม่ได้เป็นเพียง Chatbot ที่สร้างข้อความจาก LLM แต่เป็น Business Intelligence
Application ที่มี Business Logic และข้อมูลจริงเป็นแหล่งความจริง (source of truth)
โดย AI ทำหน้าที่แปลความต้องการของผู้ใช้ เลือกเรียก Business Tools ที่ได้รับอนุญาต
และอธิบายผลลัพธ์พร้อมหลักฐาน

### Product statement

> Ask business questions in natural language. Get answers grounded in
> validated business data, with traceable calculations and tool
> execution history.

### เป้าหมายหลัก

1.  สร้าง Portfolio ที่แสดงทักษะ Backend Engineering, AI Engineering และ
    System Design
2.  แสดงการใช้ AI Tool Calling อย่างมีขอบเขตและตรวจสอบได้
3.  สร้างระบบที่ Deploy และทดลองใช้งานได้จริง
4.  ออกแบบให้ขยายต่อได้ โดยไม่เริ่มด้วย Microservices หรือความซับซ้อนเกินจำเป็น

### หลักการสำคัญ

-   Business data และ deterministic business logic เป็นแหล่งความจริง
-   LLM ห้ามคำนวณหรือแต่งตัวเลขธุรกิจขึ้นเอง
-   LLM ไม่มีสิทธิ์เชื่อมต่อฐานข้อมูลโดยตรง
-   ทุก Tool ต้องผ่าน validation, authorization และ audit
-   ทุกคำตอบเชิงตัวเลขต้องมีข้อมูลอ้างอิงหรือ metadata ที่ตรวจสอบย้อนกลับได้
-   เริ่มจาก MVP ที่เล็กและสมบูรณ์ก่อน ไม่เพิ่ม feature นอก Scope โดยไม่มีการอนุมัติ

------------------------------------------------------------------------

## 2. Problem Statement

ธุรกิจ SME มักมีข้อมูลยอดขายและสินค้าคงคลังอยู่ในระบบ
แต่ผู้ใช้งานต้องเปิดหลายหน้าจอหรือรอให้เจ้าหน้าที่จัดทำรายงานเพื่อหาคำตอบ เช่น

-   สินค้าตัวไหนมีโอกาสหมดภายใน 7 วัน?
-   สินค้าตัวไหนขายดีที่สุดในเดือนนี้?
-   ยอดขายสัปดาห์นี้เปลี่ยนแปลงจากสัปดาห์ก่อนอย่างไร?
-   สินค้าตัวไหนมี Stock Coverage ต่ำ?
-   ควรตรวจสอบสินค้าใดก่อน และเพราะเหตุใด?

OpsPilot AI จะลดขั้นตอนการเข้าถึงข้อมูล โดยให้ผู้ใช้ถามคำถามผ่าน UI เดียว
และระบบเรียกใช้ Business Tools ที่กำหนดไว้เพื่อคืนผลลัพธ์ที่ตรวจสอบได้

### สิ่งที่ระบบไม่พยายามแก้ใน MVP

-   ไม่ใช่ ERP เต็มรูปแบบ
-   ไม่ใช่ระบบบัญชีหรือระบบการเงิน
-   ไม่ใช่ระบบจัดซื้อหรืออนุมัติ PO
-   ไม่ใช่ระบบพยากรณ์ด้วย Machine Learning
-   ไม่ใช่ระบบที่ให้ AI เขียน SQL ได้อย่างอิสระ
-   ไม่ใช่ระบบ Autonomous Agent ที่ตัดสินใจเปลี่ยนแปลงข้อมูลธุรกิจเอง

------------------------------------------------------------------------

## 3. Target Users and Roles

### กลุ่มผู้ใช้เป้าหมาย

1.  **Business Owner** --- ต้องการภาพรวมธุรกิจและคำตอบที่เข้าใจง่าย
2.  **Sales Manager** --- ต้องการวิเคราะห์ยอดขายและสินค้าขายดี
3.  **Inventory/Warehouse Manager** --- ต้องการติดตามความเสี่ยงสินค้าคงคลัง
4.  **Organization Admin** --- จัดการสมาชิกและสิทธิ์ภายในองค์กร

### Roles เบื้องต้น

  Role      Permissions
  --------- ----------------------------------------
  Owner     เข้าถึงข้อมูลและจัดการองค์กร/สมาชิก
  Manager   ดู Dashboard, Sales, Inventory และใช้ AI
  Analyst   ดูข้อมูลและใช้ AI ตามสิทธิ์ที่กำหนด
  Viewer    อ่าน Dashboard และรายงานที่ได้รับอนุญาต

ต้องตรวจสอบสิทธิ์ที่ Backend ทุกครั้ง ห้ามเชื่อ Role หรือ organization ID ที่ส่งมาจาก
Frontend โดยไม่ตรวจสอบกับ authenticated context

------------------------------------------------------------------------

## 4. Product Scope

### 4.1 MVP Modules

#### Module A --- Authentication and Organization

-   Login / Logout
-   User identity
-   Organization membership
-   Role-based access control (RBAC)
-   ทุก request ที่อ่านข้อมูลธุรกิจต้องมี authenticated organization context
-   ป้องกันการเข้าถึงข้อมูลข้ามองค์กร (tenant isolation)

#### Module B --- Business Dashboard

แสดงข้อมูลสรุปที่คำนวณจากฐานข้อมูล เช่น

-   ยอดขายรวมในช่วงเวลาที่เลือก
-   จำนวนคำสั่งซื้อ
-   จำนวน SKU ที่มีสินค้า
-   จำนวน SKU ที่มีความเสี่ยง
-   สินค้าขายดี
-   แนวโน้มยอดขายตามวันหรือเดือน

ตัวเลขบน Dashboard ต้องมาจาก API/SQL ที่ทดสอบแล้ว ไม่ให้ LLM เป็นผู้สร้างตัวเลข

#### Module C --- Inventory Intelligence

-   ดูยอดคงเหลือแยกตามสินค้าและคลัง
-   คำนวณยอดขายเฉลี่ยต่อวันจากช่วงเวลาที่กำหนด
-   คำนวณ Stock Coverage
-   ระบุสินค้าที่มีความเสี่ยงหมดภายใน N วัน
-   แสดงเหตุผลและค่าที่ใช้คำนวณ
-   แนะนำจำนวนเติมสินค้าได้ในรูปแบบ informational recommendation เท่านั้น

สูตรเริ่มต้น:

`average_daily_sales = units_sold_in_period / number_of_days_in_period`

`stock_coverage_days = available_stock / average_daily_sales`

เงื่อนไขที่ต้องกำหนดให้ชัดเจน: - ถ้า average_daily_sales = 0 ให้ coverage เป็น
null หรือสถานะ `no_recent_sales` ไม่ใช่หารด้วยศูนย์ - นิยาม available_stock
ต้องตกลงให้ชัด เช่น on_hand - reserved -
กำหนดช่วงข้อมูลย้อนหลังและการจัดการวันที่ไม่มีการขาย - คืน calculation metadata
เพื่อให้ UI อธิบายผลได้ - สูตรนี้เป็น heuristic ไม่ใช่การพยากรณ์ที่รับประกันอนาคต

#### Module D --- Sales Intelligence

-   ยอดขายตามช่วงวันที่
-   เปรียบเทียบช่วงเวลาปัจจุบันกับช่วงก่อนหน้า
-   จัดอันดับสินค้าขายดี
-   วิเคราะห์แนวโน้มยอดขาย
-   ตรวจจับความผิดปกติด้วย deterministic rules เบื้องต้น

ต้องกำหนดนิยามยอดขายให้ชัด เช่น นับเฉพาะคำสั่งซื้อที่ชำระแล้วหรือจัดส่งแล้ว
และจัดการการยกเลิก/คืนสินค้าอย่างไร ห้ามให้ AI ตีความนิยามเหล่านี้เอง

#### Module E --- AI Agent

-   รับคำถามภาษาธรรมชาติ
-   เลือก Business Tool ที่เหมาะสมผ่าน LLM Tool Calling
-   ตรวจสอบ arguments ก่อนเรียก Tool
-   เรียก Tool ผ่าน backend เท่านั้น
-   ส่งผลลัพธ์ที่มีโครงสร้างกลับไปให้ LLM อธิบาย
-   แสดงคำตอบพร้อมข้อมูลอ้างอิง/หลักฐาน
-   รองรับคำถามที่ไม่มีข้อมูลเพียงพอ โดยตอบอย่างตรงไปตรงมา
-   จำกัดจำนวน Tool Calls, timeout และ token budget
-   บันทึก conversation และ tool execution history

#### Module F --- AI Audit and History

-   Conversation history
-   Message history
-   Tool name และ arguments ที่ผ่านการ sanitize
-   Tool result summary
-   Execution status
-   Error details ที่ไม่เปิดเผย secret
-   Model/provider metadata
-   Latency และ token usage หาก provider ส่งข้อมูลให้
-   เวลาที่เริ่มและสิ้นสุดการทำงาน

ห้ามเก็บ API keys, passwords, access tokens หรือข้อมูลลับไว้ใน logs

### 4.2 Explicitly Out of Scope for MVP

-   Multi-agent collaboration
-   RAG / Vector database
-   Machine Learning forecasting
-   External ERP integration
-   Purchase order creation/approval
-   Automatic inventory adjustment
-   Autonomous write operations
-   Workflow builder
-   Billing/subscription
-   Mobile application
-   Microservices
-   Real-time streaming infrastructure หากยังไม่จำเป็น

การเพิ่มรายการนอก Scope ต้องมีเหตุผลทางผลิตภัณฑ์และได้รับอนุมัติจาก Project Owner ก่อน

------------------------------------------------------------------------

## 5. Key User Journeys

### Journey 1 --- ตรวจสอบภาพรวมธุรกิจ

1.  ผู้ใช้ Login
2.  Backend ตรวจสอบตัวตนและ Organization
3.  Frontend เรียก Dashboard API
4.  Backend คำนวณ KPI จากฐานข้อมูล
5.  Frontend แสดง KPI และกราฟ

### Journey 2 --- ถาม AI เรื่องสินค้าที่ใกล้หมด

ตัวอย่างคำถาม:

> สินค้าตัวไหนมีโอกาสหมดภายใน 7 วัน?

ลำดับการทำงาน:

1.  Frontend ส่งข้อความและ conversation ID (ถ้ามี) ไป Backend
2.  Backend ตรวจสอบ authentication, organization และ permission
3.  Agent Orchestrator ส่งคำถามและ Tool Definitions ให้ LLM
4.  LLM เลือกเรียก `GetInventoryRisk` พร้อม arguments
5.  Backend ตรวจสอบ schema, limits และ permission
6.  Tool เรียก Usecase ที่คำนวณจาก PostgreSQL
7.  Tool คืน structured result พร้อม calculation metadata
8.  Agent ส่งผลลัพธ์ให้ LLM สร้างคำอธิบาย
9.  Backend ตรวจสอบ output และบันทึก audit
10. Frontend แสดงคำตอบและ evidence

### Journey 3 --- ถามคำถามที่ระบบตอบไม่ได้

1.  ผู้ใช้ถามเรื่องที่ไม่มีข้อมูลหรืออยู่นอกความสามารถ
2.  Agent ไม่เลือก Tool ที่ไม่เกี่ยวข้อง หรือ Tool คืนข้อมูลว่าง
3.  ระบบแจ้งข้อจำกัดและขอข้อมูลเพิ่มเติมเมื่อจำเป็น
4.  ห้ามสร้างตัวเลขหรือข้อเท็จจริงขึ้นเอง

------------------------------------------------------------------------

## 6. System Architecture

### Architectural Style

ใช้ **Modular Monolith + Feature-based Organization + Clean Architecture
principles**

เหตุผล: - เหมาะกับทีมขนาดเล็กและการพัฒนาร่วมกับ AI Agent - ลด operational
overhead เมื่อเทียบกับ Microservices - แยก business capabilities เป็น modules
ได้ - ทดสอบ Usecase และ Business Rules ได้โดยไม่ต้องเรียก LLM - แยก AI
orchestration ออกจาก business logic

### High-level Components

``` mermaid
flowchart TB
    U[Business User]
    FE[Next.js Frontend]
    API[Go Fiber API]
    AUTH[Authentication and RBAC]
    APP[Application Usecases]
    AGENT[AI Agent Orchestrator]
    REG[Business Tool Registry]
    SALES[Sales Module]
    INV[Inventory Module]
    AUDIT[AI Audit Module]
    DB[(PostgreSQL)]
    LLM[External LLM Provider]

    U --> FE
    FE --> API
    API --> AUTH
    AUTH --> APP
    APP --> SALES
    APP --> INV
    APP --> AGENT
    AGENT <--> LLM
    AGENT --> REG
    REG --> SALES
    REG --> INV
    SALES --> DB
    INV --> DB
    AGENT --> AUDIT
    AUDIT --> DB
```

### AI Request Sequence

``` mermaid
sequenceDiagram
    participant User
    participant API as Go API
    participant Agent
    participant LLM
    participant Tool
    participant DB as PostgreSQL

    User->>API: Ask business question
    API->>API: Authenticate and authorize
    API->>Agent: Question + trusted context
    Agent->>LLM: Prompt + allowed tool definitions
    LLM-->>Agent: Tool call request
    Agent->>Agent: Validate tool and arguments
    Agent->>Tool: Execute with trusted organization context
    Tool->>DB: Parameterized read query
    DB-->>Tool: Business data
    Tool-->>Agent: Structured result + evidence
    Agent->>LLM: Tool result
    LLM-->>Agent: Natural language answer
    Agent->>Agent: Validate and persist audit
    Agent-->>API: Answer + evidence + execution metadata
    API-->>User: Response
```

### Trust Boundaries

-   Frontend เป็น untrusted client
-   LLM output เป็น untrusted input
-   Tool arguments ต้อง validate ทุกครั้ง
-   Organization ID ต้องมาจาก authenticated server-side context ไม่ใช่ค่าที่
    LLM เลือกเอง
-   Database access ต้องผ่าน Repository/Usecase ที่กำหนด
-   AI Tools ใน MVP เป็น read-only
-   ห้ามนำ raw SQL จาก LLM ไป execute

------------------------------------------------------------------------

## 7. Recommended Technology Stack

  -----------------------------------------------------------------------
  Layer                   Technology              Purpose
  ----------------------- ----------------------- -----------------------
  Frontend                Next.js App Router      Web application

  Language                TypeScript              Type safety

  UI                      Ant Design              UI components

  Data fetching           TanStack Query          Server state

  Forms                   React Hook Form + Zod   Form validation

  Backend                 Go + Fiber              REST API

  Architecture            Clean Architecture      Separation of concerns
                          principles              

  Database                PostgreSQL 18           Persistent data

  Driver                  pgx / pgxpool           PostgreSQL access

  AI                      Provider LLM API        Tool calling and
                                                  response generation

  Backend tests           Go testing + Testify    Unit/integration tests
                          (if useful)             

  Frontend tests          Vitest + Playwright     Component/E2E tests

  Deployment              Docker Compose +        Container deployment
                          Portainer               

  Reverse proxy           Nginx Proxy Manager     HTTPS and routing
  -----------------------------------------------------------------------

### AI Provider Abstraction

สร้าง interface สำหรับเปลี่ยน Provider/Model ได้โดยไม่ผูก business logic
เข้ากับผู้ให้บริการรายเดียว

แนวคิด interface (ปรับตาม SDK จริงที่เลือก):

``` go
type LLMProvider interface {
    Generate(ctx context.Context, req GenerateRequest) (GenerateResponse, error)
}
```

ต้องตรวจสอบ API documentation และความสามารถ Tool Calling ของ Provider
ก่อนเลือกใช้จริง อย่าสมมติว่า Provider ทุกตัวรองรับ request/response format
เหมือนกัน

### Agent Orchestration

เริ่มด้วย Go-based Agent Loop แบบเรียบง่าย ไม่จำเป็นต้องใช้ Agent Framework ใน
MVP

ความสามารถขั้นต่ำ: - ส่ง system/developer instructions และ tool
definitions - รับ text response หรือ tool call - validate tool name และ
arguments - execute tool ผ่าน registry - ส่ง tool result กลับไปยัง model -
จำกัดจำนวน iteration - timeout และ cancellation - จบด้วย final answer หรือ
controlled failure

------------------------------------------------------------------------

## 8. Backend Design

### Suggested Feature Structure

โครงสร้างตัวอย่าง ต้องปรับหลังอนุมัติ Architecture และ conventions ของ
repository:

``` text
backend/
├── cmd/
│   └── api/
├── internal/
│   ├── platform/
│   │   ├── config/
│   │   ├── database/
│   │   ├── http/
│   │   └── logger/
│   ├── features/
│   │   ├── auth/
│   │   │   ├── handler/
│   │   │   ├── usecase/
│   │   │   ├── repository/
│   │   │   └── domain/
│   │   ├── organization/
│   │   ├── dashboard/
│   │   ├── inventory/
│   │   ├── sales/
│   │   ├── ai/
│   │   │   ├── handler/
│   │   │   ├── usecase/
│   │   │   ├── agent/
│   │   │   ├── tools/
│   │   │   ├── provider/
│   │   │   └── domain/
│   │   └── audit/
│   └── shared/
├── migrations/
└── tests/
```

### Backend Conventions

-   Use `usecase`, not `service`, for application business workflows
    unless a clear technical reason requires otherwise.
-   Use `pgxpool`; do not introduce GORM.
-   Keep handlers thin.
-   Business rules belong in domain/usecase, not HTTP handlers.
-   Repository owns persistence queries.
-   Use context cancellation and request timeouts.
-   Use parameterized SQL.
-   Validate input at API boundary and enforce business invariants in
    usecase/domain.
-   Avoid unnecessary abstractions and speculative frameworks.

### API Response Envelope

ใช้ response format ให้สม่ำเสมอ:

``` json
{
  "success": true,
  "message": "Request completed",
  "data": {},
  "meta": {},
  "error": null
}
```

Error response ต้องไม่เปิดเผย stack trace, SQL details, secrets หรือ
internal infrastructure details ต่อผู้ใช้

### API Endpoint Proposal

Endpoint เป็นข้อเสนอเบื้องต้น ต้องตรวจสอบและอนุมัติใน API Contract ก่อน implement

  -----------------------------------------------------------------------------------------
  Method                  Endpoint                                  Purpose
  ----------------------- ----------------------------------------- -----------------------
  POST                    `/api/v1/auth/login`                      Login

  POST                    `/api/v1/auth/logout`                     Logout

  GET                     `/api/v1/me`                              Current user

  GET                     `/api/v1/dashboard/summary`               Dashboard KPIs

  GET                     `/api/v1/dashboard/sales-trend`           Sales trend

  GET                     `/api/v1/inventory`                       Inventory listing

  GET                     `/api/v1/inventory/risks`                 Inventory risk

  GET                     `/api/v1/sales/summary`                   Sales summary

  GET                     `/api/v1/sales/top-products`              Top products

  POST                    `/api/v1/ai/conversations`                Create conversation

  GET                     `/api/v1/ai/conversations`                List conversations

  GET                     `/api/v1/ai/conversations/:id/messages`   Conversation messages

  POST                    `/api/v1/ai/conversations/:id/messages`   Send user message

  GET                     `/api/v1/ai/executions/:id`               Tool execution details
  -----------------------------------------------------------------------------------------

ต้องกำหนด pagination, date filters, sorting, error codes, authorization
และ response schema ก่อนพัฒนา Frontend

------------------------------------------------------------------------

## 9. Database Design

### Initial Tables

  -----------------------------------------------------------------------
  Table                               Important fields / purpose
  ----------------------------------- -----------------------------------
  users                               User identity

  organizations                       Tenant/business

  organization_members                User membership and role

  products                            SKU, name, status

  warehouses                          Warehouse information

  inventory_balances                  On-hand, reserved, available
                                      quantity

  sales_orders                        Order date, status, totals

  sales_order_items                   Product, quantity, unit price

  ai_conversations                    Organization, user, title

  ai_messages                         Role, content, metadata

  ai_tool_executions                  Tool, arguments, status, duration,
                                      result summary
  -----------------------------------------------------------------------

### Database Requirements

-   Internal IDs ใช้ BIGINT ได้
-   Public-facing identifiers ใช้ UUID ได้
-   ใช้ foreign keys และ indexes ตาม query patterns
-   ใช้ timestamps อย่างสม่ำเสมอ และกำหนด timezone policy
-   Tenant-owned records ต้องมี organization ownership ที่ตรวจสอบได้
-   ทุก query ที่อ่าน tenant data ต้อง scope ด้วย organization context
-   ห้ามพึ่งพา Frontend filtering เพื่อแยกข้อมูล tenant
-   ใช้ migrations ที่ตรวจสอบย้อนหลังได้
-   Seed data ต้องแยกจาก production data

### Data Model Decisions Still Required

ก่อนสร้าง migration ต้องตัดสินใจเรื่องต่อไปนี้: 1. สินค้าหนึ่งรายการอยู่ได้หลายคลังหรือไม่
2. นิยาม available stock และ reserved stock 3. นิยามยอดขายสุทธิ 4. สถานะ
order ใดถูกนับเป็นยอดขาย 5. วิธีจัดการ cancellation และ returns 6. สกุลเงินและ
precision ของ monetary values 7. การเก็บ historical inventory snapshot
หรือ transaction ledger 8. Soft delete จำเป็นกับ entity ใดบ้าง

AI Agent ต้องไม่เป็นผู้ตัดสินใจ business definitions เหล่านี้เอง

------------------------------------------------------------------------

## 10. Business Logic Specifications

### Inventory Risk

Inputs: - organization context - optional warehouse filter - threshold
days (เช่น 7) - historical lookback period

Derived values: - available stock - units sold in lookback period -
average daily sales - estimated stock coverage days - risk
classification

ตัวอย่าง risk classification เบื้องต้น: - `critical`: coverage \<= 2 days -
`high`: coverage \> 2 and \<= 7 days - `normal`: coverage \> 7 days -
`no_recent_sales`: average daily sales = 0 - `insufficient_data`:
ข้อมูลย้อนหลังไม่เพียงพอ

Thresholds ต้องเป็น configuration หรือ business policy ที่ชัดเจน ไม่ hardcode
กระจายหลายจุด

### Sales Analysis

ต้องนิยาม: - gross sales - net sales - order count - comparison period -
timezone ที่ใช้ตัดวัน - treatment of canceled/refunded orders

การเปรียบเทียบช่วงเวลา:
`percentage_change = ((current - previous) / previous) * 100`

กรณี previous = 0 ต้องคืนสถานะพิเศษ ไม่หารด้วยศูนย์หรือสร้างเปอร์เซ็นต์ที่ทำให้เข้าใจผิด

### Evidence Contract

Business Tool ควรคืนผลลัพธ์แบบมีโครงสร้าง เช่น:

``` json
{
  "tool": "GetInventoryRisk",
  "generated_at": "2026-01-01T10:00:00Z",
  "filters": {
    "threshold_days": 7,
    "lookback_days": 30
  },
  "summary": {
    "at_risk_count": 2
  },
  "items": [
    {
      "sku": "SKU-001",
      "available_stock": 10,
      "units_sold": 150,
      "average_daily_sales": 5,
      "coverage_days": 2,
      "risk_level": "critical"
    }
  ],
  "calculation_version": "inventory-risk-v1"
}
```

ข้อมูลข้างต้นเป็นตัวอย่าง schema ไม่ใช่ข้อมูลจริง

AI สามารถอธิบายผลลัพธ์ได้ แต่ห้ามแก้ไขค่าตัวเลขจาก Tool result

------------------------------------------------------------------------

## 11. AI Agent Design

### Allowed Tools in MVP

เริ่มจาก Tools ที่อ่านข้อมูลเท่านั้น:

  Tool                      Input                         Output
  ------------------------- ----------------------------- -----------------
  `GetBusinessSummary`      date range                    KPI summary
  `GetInventoryRisk`        threshold days, filters       Risk list
  `GetInventoryByProduct`   product/SKU filter            Stock detail
  `GetSalesSummary`         date range                    Sales metrics
  `GetTopProducts`          date range, limit             Ranked products
  `CompareSalesPeriods`     current and previous ranges   Comparison

ทุก Tool ต้องมี: - Name และ description ที่ชัดเจน - Strict input schema -
Server-side validation - Authorization policy - Organization context จาก
server - Timeout และ resource limits - Structured output - Error
contract - Audit record

### Agent Safety Rules

1.  Treat user input and model output as untrusted.
2.  Never execute arbitrary SQL generated by the model.
3.  Never allow the model to choose or override organization identity.
4.  Never expose tools the current user is not authorized to call.
5.  Validate tool arguments before execution.
6.  Enforce maximum tool-call iterations.
7.  Enforce request timeout and token budget.
8.  Do not execute write operations in MVP.
9.  Do not reveal system prompts, secrets, internal logs, or other
    tenants' data.
10. If evidence is missing, say so explicitly.
11. Separate observed facts from interpretations and recommendations.
12. Log tool execution without storing credentials or unnecessary
    sensitive data.

### Prompt Injection Considerations

ข้อความจากผู้ใช้ เอกสาร และข้อมูลในฐานข้อมูลอาจมีข้อความที่พยายามสั่งให้ Agent ฝ่าฝืนกฎ
ระบบต้องถือว่าข้อความเหล่านั้นเป็น data ไม่ใช่ privileged instructions
และต้องบังคับใช้ permission ที่ backend ไม่ใช่ฝากความปลอดภัยไว้กับ prompt
เพียงอย่างเดียว

### Response Structure

คำตอบ AI ควรแยก: - `answer`: คำอธิบายภาษาธรรมชาติ - `key_findings`:
ข้อค้นพบ - `evidence`: Tool และข้อมูลที่รองรับคำตอบ - `limitations`:
ข้อจำกัดของข้อมูล - `recommendations`: ข้อเสนอแนะที่ระบุชัดว่าเป็น recommendation -
`execution_id`: สำหรับตรวจสอบย้อนหลัง

หาก Provider รองรับ structured output ให้ใช้ schema validation เพิ่มเติม

------------------------------------------------------------------------

## 12. Frontend Scope

### Main Pages

1.  Login
2.  Dashboard
3.  Inventory
4.  Sales Analytics
5.  AI Copilot
6.  Conversation History
7.  AI Execution Detail
8.  Organization/Member Settings (เฉพาะที่จำเป็นสำหรับ MVP)

### UX Principles

-   แสดง loading, empty, error และ success states
-   คำตอบ AI ต้องแยกข้อความสรุปออกจาก evidence
-   ผู้ใช้เปิดดู Tool execution detail ได้
-   แสดงวันที่และช่วงเวลาที่ใช้วิเคราะห์
-   แสดงคำเตือนเมื่อข้อมูลไม่ครบ
-   ห้ามแสดง recommendation เป็นการดำเนินการที่เกิดขึ้นแล้ว
-   ไม่ต้องทำ UI ซับซ้อนเกินความจำเป็น

------------------------------------------------------------------------

## 13. Testing and Quality Strategy

### Backend

-   Unit tests สำหรับ business calculations
-   Repository integration tests กับ PostgreSQL
-   Handler/API tests
-   Authentication/authorization tests
-   Tenant isolation tests
-   Tool schema validation tests
-   Agent loop tests ด้วย mock LLM provider
-   Timeout, cancellation และ provider error tests

### AI Evaluation

สร้าง Golden Dataset อย่างน้อย 20--30 คำถาม ครอบคลุม: - คำถามที่ตอบได้ด้วย Tool
เดียว - คำถามที่ต้องเปรียบเทียบข้อมูล - คำถามที่ต้องขอ clarification -
คำถามที่ไม่มีข้อมูล - คำถามนอกขอบเขต - Prompt injection attempts - Tool
arguments ที่ไม่ถูกต้อง - User ที่ไม่มี permission

เกณฑ์ประเมิน: - Tool selection correctness - Argument correctness -
Numerical fidelity to tool results - Evidence completeness -
Refusal/abstention correctness - No cross-tenant leakage

อย่าใช้ความลื่นไหลของภาษาคำตอบเป็นตัวชี้วัดคุณภาพเพียงอย่างเดียว

### Frontend

-   Unit/component tests สำหรับ critical components
-   API integration tests
-   Playwright E2E สำหรับ main user journeys

### Release Gate

ห้ามประกาศว่า feature ผ่าน หากยังไม่มีผลทดสอบจริงที่ตรวจสอบได้
ห้ามแต่งผลทดสอบหรืออ้างว่า test ผ่านโดยไม่ได้รัน

------------------------------------------------------------------------

## 14. Security Requirements

ขั้นต่ำสำหรับ MVP:

-   Password hashing ด้วย bcrypt หรือแนวทางที่เหมาะสม
-   Secure session/token handling
-   Authentication และ authorization ที่ Backend
-   Tenant isolation
-   Input validation
-   Parameterized SQL
-   Rate limiting สำหรับ AI endpoints
-   Request size limits
-   LLM timeout และ spending/token limits
-   Secrets ผ่าน environment/secrets management
-   CORS configuration แบบจำกัด origin
-   Safe error messages
-   Audit logging
-   Dependency review
-   Prompt injection defense in depth

ห้าม: - Commit `.env` หรือ API keys - Log credentials หรือ access tokens -
ให้ LLM รัน SQL โดยตรง - เชื่อ organization ID จาก request body
โดยไม่ตรวจสอบ - เปิดเผยข้อมูลระหว่างองค์กร - เพิ่ม write-capable AI Tools ใน MVP

------------------------------------------------------------------------

## 15. Observability and Cost Controls

ติดตามข้อมูลอย่างน้อย: - API latency - Error rate - LLM provider latency -
Token usage (เมื่อมีข้อมูล) - Tool call count - Tool execution duration -
Failed tool calls - Agent iteration count - Approximate cost per
conversation (หากมีราคาและ usage ที่เชื่อถือได้)

Cost controls: - จำกัด max output tokens - จำกัด tool iterations - จำกัด
message length - ตั้ง request timeout - กำหนด rate limit - หลีกเลี่ยงส่ง
conversation history ทั้งหมดโดยไม่จำเป็น - ไม่เรียก LLM เมื่อคำถามตอบได้ด้วย
deterministic API โดยตรง

------------------------------------------------------------------------

## 16. Deployment

### Target Deployment

-   Docker Compose
-   Portainer CE เป็น UI หลักในการจัดการ Stack/Container
-   PostgreSQL
-   Go API
-   Next.js frontend
-   Nginx Proxy Manager สำหรับ reverse proxy/HTTPS
-   Persistent volume สำหรับข้อมูล PostgreSQL
-   Environment variables/secrets แยกตาม environment

### Deployment Requirements

-   มี health checks
-   มี restart policy ที่เหมาะสม
-   มี persistent database storage
-   มี backup/restore procedure
-   แยก development และ production configuration
-   ไม่ expose PostgreSQL สู่ public internet โดยไม่จำเป็น
-   ไม่ใส่ secret ใน image หรือ repository
-   มี deployment instructions ที่ทำตามได้จริงผ่าน Portainer

ห้าม AI Agent เปลี่ยนแปลง VPS, Docker stack, network, volume หรือ production
data โดยไม่ได้รับอนุมัติจาก Project Owner

------------------------------------------------------------------------

## 17. Development Roadmap --- 8 Weeks

Timeline เป็นเป้าหมายประมาณการสำหรับการทำงาน part-time
และปรับได้ตามผลการพัฒนาจริง

  -----------------------------------------------------------------------
  Week                    Focus                   Deliverables
  ----------------------- ----------------------- -----------------------
  1                       Product & Architecture  PRD, Architecture, ERD,
                                                  API Contract, repo
                                                  foundation

  2                       Backend Foundation      DB migrations, auth,
                                                  organization, RBAC

  3                       Business Intelligence   Inventory/Sales
                                                  usecases, calculation
                                                  tests

  4                       Frontend Dashboard      Dashboard, inventory,
                                                  sales, API integration

  5                       AI Agent                Provider interface,
                                                  agent loop, tool
                                                  registry, tool calling

  6                       AI Reliability          Evidence,
                                                  audit/history,
                                                  evaluation dataset

  7                       QA & Security           E2E, tenant isolation,
                                                  prompt injection, bug
                                                  fixes

  8                       Deployment & Portfolio  Docker, Portainer
                                                  deployment, README,
                                                  demo
  -----------------------------------------------------------------------

### Milestones

-   M1: Project foundation approved
-   M2: Secure backend foundation
-   M3: Deterministic business engine works
-   M4: Dashboard integrated
-   M5: First end-to-end AI Tool Calling flow
-   M6: Auditable and evaluated AI responses
-   M7: Release candidate passes critical tests
-   M8: Public/private demo deployment and portfolio documentation

### Definition of Done for Version 1.0

-   User can log in
-   Organization boundaries are enforced
-   Dashboard displays validated business metrics
-   Inventory and Sales analysis work without LLM
-   User can ask supported business questions
-   Agent calls only approved tools
-   AI answers are grounded in tool results
-   Tool executions can be audited
-   Critical automated tests pass
-   Application deploys using documented Portainer workflow
-   README and demo instructions are complete

------------------------------------------------------------------------

## 18. Portfolio Deliverables

สิ่งที่ต้องนำเสนอใน GitHub/Portfolio:

1.  Product overview and problem statement
2.  Architecture diagram
3.  ER diagram
4.  API documentation
5.  AI Agent execution flow
6.  Security design and tenant isolation approach
7.  Test strategy and actual test results
8.  Screenshots of key screens (created by the project team)
9.  Demo video
10. Deployment guide
11. Known limitations and future roadmap

### Interview Talking Points

ผู้พัฒนาควรอธิบายได้ว่า: - ทำไมเลือก Modular Monolith แทน Microservices -
ทำไมไม่ให้ LLM เขียน SQL โดยตรง - Tool Calling ทำงานอย่างไร - ป้องกัน
cross-tenant data leakage อย่างไร - ตรวจสอบความถูกต้องของ AI response
อย่างไร - จัดการ hallucination และ insufficient data อย่างไร - ควบคุม LLM
cost และ latency อย่างไร - ทดสอบ Agent ที่มี nondeterministic output อย่างไร

------------------------------------------------------------------------

## 19. Future Roadmap --- After MVP

พิจารณาเฉพาะเมื่อ Version 1.0 เสถียรและมี feedback:

### V1.1

-   Export reports
-   Saved analysis
-   More flexible filters
-   Improved AI evaluation and observability

### V1.2

-   RAG for business documentation
-   Upload business policy documents
-   Answer policy questions with citations

### V2.0

-   Demand forecasting
-   More advanced anomaly detection
-   Optional integrations with external ERP/POS
-   Carefully controlled approval workflows

ทุก feature ใหม่ต้องมี use case, security review, cost estimate และ
acceptance criteria

------------------------------------------------------------------------

## 20. AI Agent Team Operating Instructions

ส่วนนี้เป็นคำสั่งการทำงานสำหรับ AI Coding Agents ที่อ่านเอกสารนี้

### Role

คุณคือ AI Software Engineering Team ที่ช่วยพัฒนา OpsPilot AI โดยมี Project
Owner เป็นผู้ตัดสินใจและอนุมัติทุกการเปลี่ยนแปลงสำคัญ

### Mandatory Workflow

1.  อ่านเอกสารนี้ทั้งหมดก่อนเสนอแผน
2.  ตรวจสอบ repository และสถานะปัจจุบันก่อนทำงาน
3.  ระบุข้อเท็จจริง สมมติฐาน และสิ่งที่ยังไม่ทราบแยกจากกัน
4.  เสนอ implementation plan ที่ระบุ files, scope, dependencies และ tests
5.  **หยุดรอ Project Owner อนุมัติก่อนแก้ไขไฟล์**
6.  ทำเฉพาะงานที่ได้รับอนุมัติ
7.  ทำการเปลี่ยนแปลงให้น้อยที่สุดและสอดคล้องกับ conventions
8.  รัน tests/lint/build ที่เกี่ยวข้องจริง
9.  ตรวจสอบ git diff และ git status
10. รายงานผลลัพธ์อย่างตรงไปตรงมา

### Permission Boundaries

ห้ามทำสิ่งต่อไปนี้หากไม่ได้รับอนุมัติอย่างชัดเจน: - Commit หรือ Push - เปลี่ยน branch หรือ
rewrite Git history - เพิ่ม dependency - เปลี่ยน environment/configuration -
รัน destructive commands - แก้ไขหรือ migrate production database -
ลบไฟล์หรือข้อมูล - เปลี่ยน Docker/VPS infrastructure - เปลี่ยน API contract หรือ
database schema ที่อนุมัติแล้ว - เพิ่ม feature นอก scope

หากต้องใช้คำสั่งที่มีผลกระทบต่อข้อมูล ระบบ หรือ environment ให้หยุดและขออนุมัติก่อน

### Engineering Rules

-   Backend: Go + Fiber + PostgreSQL + pgxpool
-   Architecture: Modular Monolith, feature-based, Clean Architecture
    principles
-   Application workflows ใช้ `usecase` naming
-   Frontend: Next.js App Router + TypeScript + Ant Design
-   API response envelope ต้องสอดคล้องกับเอกสาร
-   LLM ไม่สามารถเข้าถึง database โดยตรง
-   AI tools เป็น read-only ใน MVP
-   ทุก tenant query ต้อง enforce organization scope
-   ห้ามสร้าง fake test results หรือ fake business data ใน production
-   หลีกเลี่ยง overengineering และ unnecessary abstractions
-   อย่าเปลี่ยนเทคโนโลยีโดยไม่มีเหตุผลและการอนุมัติ

### Handling Ambiguity

หาก requirement ไม่ชัด: - อย่าเดา business rules - ระบุคำถามที่จำเป็น -
เสนอทางเลือกพร้อม trade-offs - รอ Project Owner ตัดสินใจในประเด็นที่มีผลต่อ
architecture, schema, security หรือ product behavior

### Final Report Format

ทุก task ต้องรายงาน:

1.  **Summary** --- ทำอะไร
2.  **Files Changed** --- ไฟล์ที่เพิ่ม/แก้/ลบ
3.  **Design Decisions** --- การตัดสินใจและเหตุผล
4.  **Tests Executed** --- คำสั่งที่รันและผลจริง
5.  **Unverified Items** --- สิ่งที่ยังไม่ได้ตรวจสอบ
6.  **Risks / Follow-ups** --- ประเด็นที่ต้องพิจารณา
7.  **Git Status** --- สถานะการเปลี่ยนแปลง โดยห้าม commit/push เอง

------------------------------------------------------------------------

## 21. Initial Task for the AI Team

**ห้ามเริ่มเขียนโค้ดในขั้นตอนแรก**

งานแรกคือทำ Discovery และ Planning เท่านั้น:

1.  สรุปความเข้าใจ Product และ MVP
2.  ตรวจสอบข้อกำหนดที่ยังไม่ชัดเจน
3.  เสนอ PRD ฉบับละเอียดพร้อม User Stories และ Acceptance Criteria
4.  เสนอ ERD และ Data Dictionary
5.  เสนอ API Contract พร้อม request/response/error examples
6.  เสนอ AI Tool schemas และ authorization matrix
7.  เสนอ test plan และ security threat checklist
8.  แบ่งงานเป็น implementation phases และระบุ dependencies
9.  ระบุข้อสมมติฐานและคำถามที่ต้องให้ Project Owner ตัดสินใจ

ส่งผลลัพธ์เป็นเอกสาร Markdown ใน `docs/` โดยเสนอชื่อไฟล์และเนื้อหาก่อนแก้ไข
repository

**หยุดและรอ Project Owner อนุมัติ ก่อนสร้างหรือแก้ไขไฟล์ใด ๆ**

------------------------------------------------------------------------

## 22. Open Decisions --- ต้องตัดสินใจก่อน Implementation

รายการต่อไปนี้ยังไม่ถือว่าเป็นข้อสรุป:

-   [ ] ชื่อ repository และ module path
-   [ ] รูปแบบ authentication/session
-   [ ] Organization onboarding และการสร้าง tenant
-   [ ] Inventory data model และ stock movement strategy
-   [ ] นิยาม net sales และ order statuses
-   [ ] Lookback period และ risk thresholds
-   [ ] LLM provider และ model ที่ใช้จริง
-   [ ] LLM request/response format และ tool calling support
-   [ ] Conversation retention policy
-   [ ] Deployment target และ resource limits
-   [ ] Demo data volume และ seed strategy
-   [ ] ค่าใช้จ่าย LLM ที่ยอมรับได้

อย่าตัดสินใจเรื่องเหล่านี้แทน Project Owner หากมีผลต่อ business behavior,
security, cost หรือ architecture

------------------------------------------------------------------------

## 23. Guiding Principle

> Build a reliable business system first. Add AI as a controlled
> reasoning and interaction layer---not as a replacement for engineering
> discipline.

เป้าหมายของ OpsPilot AI ไม่ใช่การพิสูจน์ว่า AI สามารถสร้างโค้ดได้มากแค่ไหน
แต่เป็นการพิสูจน์ว่าผู้พัฒนาสามารถออกแบบ สร้าง ทดสอบ ป้องกัน และส่งมอบระบบที่ใช้ AI
อย่างรับผิดชอบได้จริง
