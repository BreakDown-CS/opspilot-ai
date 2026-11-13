# 00 — Discovery Summary

> Status: Draft for Project Owner review
> Source: `opspilot-ai-project-blueprint.md` v1.0
> Last updated: 2026-11-13

## 1. Product understanding

OpsPilot AI คือ Business Intelligence application สำหรับ SME ที่ให้ผู้ใช้:

1. ดู Dashboard ยอดขายและสินค้าคงคลังที่คำนวณจากฐานข้อมูลด้วย logic ที่ทดสอบแล้ว
2. ถามคำถามธุรกิจด้วยภาษาธรรมชาติ โดย LLM เลือกเรียก **read-only Business Tools** ที่ backend กำหนดไว้
3. ตรวจสอบย้อนหลังได้ว่าคำตอบ AI มาจาก tool ใด ด้วย arguments อะไร และได้ผลลัพธ์อะไร

แก่นของระบบ: **ตัวเลขมาจาก deterministic code เสมอ — LLM เป็นแค่ชั้นแปลคำถามและอธิบายผล**

## 2. MVP in one view

| Module | สิ่งที่ส่งมอบ | ต้องใช้ LLM? |
|---|---|---|
| A Auth & Org | Login/logout, membership, RBAC, tenant isolation | ไม่ |
| B Dashboard | KPI summary, sales trend, top products | ไม่ |
| C Inventory | Stock by product/warehouse, coverage, risk | ไม่ |
| D Sales | Summary, top products, period comparison, anomaly rules | ไม่ |
| E AI Agent | Tool-calling loop, 6 read-only tools, evidence | ใช่ |
| F AI Audit | Conversation/message/tool execution history | ไม่ (บันทึกผล) |

Module A–D ต้องทำงานได้สมบูรณ์โดยไม่มี LLM (Definition of Done ข้อ "Inventory and Sales analysis work without LLM")

## 3. Facts (ตรวจสอบแล้ว)

| รายการ | ค่า |
|---|---|
| Repository | มีเพียง blueprint และ `docs/`; ยังไม่ได้ `git init` |
| Go | 1.27.0 |
| Node.js | 24.15.0 (LTS) |
| Docker | 29.8.0 |
| OS (dev) | Windows 11 |

## 4. Assumptions (สมมติฐาน — ต้องยืนยัน)

ดูรายละเอียดและสถานะใน [09-decision-log.md](09-decision-log.md)

1. Monorepo เดียว: `backend/`, `frontend/`, `deploy/`, `docs/`
2. Auth แบบ server-side session ใน PostgreSQL + httpOnly cookie
3. ไม่มี self-signup; org และ owner สร้างจาก seed/CLI
4. หนึ่งผู้ใช้อาจเป็นสมาชิกได้หลาย org แต่ทำงานใน "active org" ทีละ org
5. สินค้าอยู่ได้หลายคลัง; เก็บยอดคงเหลือเป็น snapshot (ไม่มี stock ledger ใน MVP)
6. ยอดขายนับเฉพาะ order status `paid`, `fulfilled`
7. หนึ่ง org ใช้สกุลเงินเดียว และ timezone เดียว
8. LLM provider: Anthropic Claude ผ่าน provider interface
9. Demo data สร้างจาก deterministic seed script — ไม่ใช่ข้อมูลจริงของธุรกิจใด

## 5. Unknowns (ยังไม่ทราบ)

- GitHub username (สำหรับ module path) — Owner กำลังสร้าง repo
- งบประมาณ LLM ต่อเดือนที่ยอมรับได้ (เสนอ USD 20)
- ภาษาหลักของ UI (เสนอไทย — D-15)

Resolved 2026-11-13: deployment = local only (D-10), demo = private (D-18), LLM = Claude (D-08) — ดู [09](09-decision-log.md)

## 6. Ambiguities found in blueprint

| # | จุดที่ไม่ชัด | ผลกระทบ | เสนอให้ตัดสินใจใน |
|---|---|---|---|
| 1 | `insufficient_data` นิยามอย่างไร | Risk classification | D-07 |
| 2 | Analyst vs Viewer ต่างกันอย่างไรนอกจาก AI | RBAC matrix | D-13 |
| 3 | "Anomaly detection ด้วย deterministic rules" — rule อะไร | Module D scope | D-14 |
| 4 | `GetBusinessSummary` ซ้ำกับ `GetSalesSummary` หรือไม่ | Tool design | ดู 05 §2 |
| 5 | Dashboard "จำนวน SKU ที่มีสินค้า" นับรวมทุกคลังหรือต่อคลัง | KPI definition | 03 §5 |
| 6 | ช่วงวันที่ inclusive/exclusive และ timezone ของ date filter | API + calculations | D-06 |
| 7 | AI response structure (answer/key_findings/…) ให้ LLM สร้างหรือ backend ประกอบ | Agent design | 05 §6 |

## 7. Document map

| File | Purpose |
|---|---|
| [01-prd.md](01-prd.md) | User stories + acceptance criteria |
| [02-architecture.md](02-architecture.md) | Architecture, ADRs, trust boundaries |
| [03-erd-data-dictionary.md](03-erd-data-dictionary.md) | ERD, tables, business definitions |
| [04-api-contract.md](04-api-contract.md) | REST contract |
| [05-ai-tools-and-authorization.md](05-ai-tools-and-authorization.md) | Tool schemas, agent limits, authz matrix |
| [06-test-plan.md](06-test-plan.md) | Test strategy + golden dataset |
| [07-security-threat-checklist.md](07-security-threat-checklist.md) | Threat model + checklist |
| [08-implementation-phases.md](08-implementation-phases.md) | Phases, dependencies, exit criteria |
| [09-decision-log.md](09-decision-log.md) | Open decisions & status |
