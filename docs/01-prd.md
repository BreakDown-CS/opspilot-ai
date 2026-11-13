# 01 — Product Requirements Document (MVP)

> Status: Draft — depends on decisions in [09-decision-log.md](09-decision-log.md)

## 1. Goal

ให้ผู้ใช้ SME ตอบคำถามเรื่องยอดขายและสินค้าคงคลังได้ในหน้าจอเดียว ทั้งผ่าน Dashboard และผ่าน AI Copilot โดยทุกตัวเลขตรวจสอบย้อนกลับได้

### Success metrics (MVP / portfolio)

| Metric | Target |
|---|---|
| Golden dataset — tool selection correctness | ≥ 90% |
| Golden dataset — numerical fidelity (ตัวเลขในคำตอบตรงกับ tool result) | 100% |
| Cross-tenant leakage ใน tests | 0 |
| Dashboard API p95 latency (seed data) | < 300 ms |
| AI answer p95 latency | < 15 s |

## 2. Personas

| Persona | Role ในระบบ | ต้องการ |
|---|---|---|
| Business Owner | Owner | ภาพรวม, จัดการสมาชิก |
| Sales Manager | Manager | ยอดขาย, สินค้าขายดี, เปรียบเทียบช่วงเวลา |
| Inventory Manager | Manager | Stock risk, coverage |
| Analyst | Analyst | วิเคราะห์ข้อมูล, ใช้ AI |
| Viewer | Viewer | อ่าน Dashboard |

## 3. User Stories & Acceptance Criteria

รูปแบบ: **US-xx** — As a … I want … so that …

### Module A — Authentication & Organization

**US-01 Login**
As a user, I want to log in with email and password so that I can access my organization's data.
- AC1: Given ข้อมูลถูกต้อง, When login, Then ได้ session cookie (httpOnly, Secure, SameSite=Lax) และ redirect ไป Dashboard
- AC2: Given รหัสผ่านผิดหรือ email ไม่มี, Then ได้ error `AUTH_INVALID_CREDENTIALS` ข้อความเดียวกัน (ไม่บอกว่าผิดที่ email หรือ password)
- AC3: Login ล้มเหลวเกิน 5 ครั้ง/15 นาที/IP+email → `RATE_LIMITED`
- AC4: Password เก็บเป็น bcrypt hash (cost ≥ 12) เท่านั้น

**US-02 Logout**
- AC1: Logout แล้ว session ถูก revoke ที่ server; ใช้ cookie เดิมเรียก API ได้ `401`

**US-03 Current user & organization**
- AC1: `GET /me` คืน user, memberships, active organization และ role
- AC2: ผู้ใช้หลาย org สลับ active org ได้เฉพาะ org ที่ตนเป็นสมาชิก

**US-04 Tenant isolation**
As an owner, I want my data isolated from other organizations.
- AC1: ทุก endpoint ที่อ่าน tenant data ใช้ organization จาก session เท่านั้น
- AC2: ส่ง `organization_id` ใน query/body → ถูก ignore หรือ `400` (ไม่มีผลต่อ scope)
- AC3: อ้าง resource ID ของ org อื่น → `404` (ไม่ใช่ `403` เพื่อไม่เปิดเผยการมีอยู่)

**US-05 Member management (Owner)**
- AC1: Owner ดูรายชื่อสมาชิกและ role ได้
- AC2: Owner เปลี่ยน role / ลบสมาชิกได้ แต่ลบหรือลด role ของ Owner คนสุดท้ายไม่ได้
- AC3: Owner เพิ่มสมาชิกด้วย email ของ user ที่มีอยู่แล้ว (ไม่มี invitation email ใน MVP — D-03)

### Module B — Dashboard

**US-10 KPI summary**
As a manager, I want to see total sales, orders, in-stock SKUs and at-risk SKUs for a date range.
- AC1: Default range = 30 วันล่าสุดตาม timezone ของ org
- AC2: ทุก KPI แสดงช่วงวันที่และ timezone ที่ใช้
- AC3: ตัวเลขตรงกับ SQL fixture ใน integration test
- AC4: ไม่มีข้อมูล → แสดง empty state ไม่ใช่ 0 ที่ทำให้เข้าใจผิด

**US-11 Sales trend**
- AC1: Granularity `day` หรือ `month`; วันที่ไม่มียอดขายแสดงเป็น 0 (gap-filled)
- AC2: Range สูงสุด 366 วัน

**US-12 Top products widget**
- AC1: แสดง top N (default 5, max 50) เรียงตาม net sales หรือ units

### Module C — Inventory Intelligence

**US-20 Inventory listing**
- AC1: แสดง on_hand, reserved, available ต่อ product × warehouse
- AC2: Filter ตาม warehouse, SKU/name search; pagination

**US-21 Inventory risk**
As an inventory manager, I want to see products likely to run out within N days.
- AC1: Input threshold_days (1–90, default 7), lookback_days (7–180, default 30), warehouse optional
- AC2: Classification ตาม policy ใน 03 §6; ไม่มีการหารด้วยศูนย์
- AC3: แต่ละรายการแสดง available_stock, units_sold, average_daily_sales, coverage_days, risk_level
- AC4: Response มี `calculation_version` และ filters ที่ใช้
- AC5: Suggested reorder quantity (ถ้ามี) ติด label "informational recommendation"

### Module D — Sales Intelligence

**US-30 Sales summary** — gross, discount, net, order count, units, AOV สำหรับช่วงวันที่
**US-31 Top products** — ranking by net sales / units, limit 1–50
**US-32 Compare periods**
- AC1: Default previous period = ช่วงยาวเท่ากันก่อนหน้าติดกัน
- AC2: previous = 0 → `change_status = "no_baseline"` และ `percentage_change = null`
**US-33 Anomaly flags** — แสดงวันที่ยอดขายผิดปกติตาม rule ใน D-14 พร้อมเหตุผลของ rule

### Module E — AI Copilot

**US-40 Ask a question**
As a manager, I want to ask "สินค้าตัวไหนมีโอกาสหมดภายใน 7 วัน?" and get a grounded answer.
- AC1: Agent เรียก `GetInventoryRisk` ด้วย `threshold_days=7`
- AC2: คำตอบแสดง answer, key findings, evidence (tool + filters + ตัวเลข), limitations, recommendations แยกส่วนกัน
- AC3: ตัวเลขในคำตอบต้องมีอยู่ใน tool result (ตรวจโดย numeric fidelity check — 05 §7)
- AC4: มี link ไปยัง execution detail

**US-41 Unanswerable question**
- AC1: คำถามนอก scope (เช่น "กำไรสุทธิปีนี้") → แจ้งว่าไม่มีข้อมูลนี้ในระบบ ไม่สร้างตัวเลข
- AC2: คำถามกำกวม (เช่น "ยอดขายเป็นยังไง") → ใช้ default range และระบุ assumption ในคำตอบ หรือถามกลับ

**US-42 Limits**
- AC1: ข้อความเกิน 2,000 ตัวอักษร → `400`
- AC2: เกิน tool iteration / timeout → controlled failure พร้อมข้อความที่เข้าใจได้ และบันทึก audit status `failed`
- AC3: Rate limit ต่อ user (D-09)

**US-43 Permission-aware tools**
- AC1: Viewer ไม่สามารถใช้ AI (D-13) → endpoint คืน `403`
- AC2: Tool ที่ role ไม่มีสิทธิ์จะไม่ถูกส่งให้ LLM เลย และถ้า LLM ขอเรียก → reject + audit

### Module F — AI Audit & History

**US-50 Conversation history** — list conversations ของตนเอง (Owner เห็นทั้ง org — D-13), เปิดดู messages
**US-51 Execution detail**
- AC1: แสดง tool name, sanitized arguments, status, duration, result summary, model, token usage, เวลาเริ่ม/จบ
- AC2: ไม่มี secrets, cookie, system prompt ใน response

## 4. Non-functional requirements

| Area | Requirement |
|---|---|
| Security | ดู [07](07-security-threat-checklist.md) |
| Performance | Dashboard p95 < 300 ms บน seed data (~5k orders) |
| Reliability | LLM provider ล่ม → Module A–D ยังใช้งานได้ |
| Observability | Structured JSON logs + request ID; metrics ตาม blueprint §15 |
| i18n | UI ภาษาไทยเป็นหลัก (D-15); AI ตอบตามภาษาคำถาม |
| Accessibility | ใช้ Ant Design defaults; contrast และ keyboard nav พื้นฐาน |

## 5. Out of scope (MVP)

ตาม blueprint §4.2 — RAG, ML forecasting, ERP integration, PO, write-capable tools, billing, mobile app, microservices, streaming, self-signup, invitation emails, partial returns
