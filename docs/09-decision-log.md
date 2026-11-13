# 09 — Decision Log

> สถานะ: **Proposed** = ข้อเสนอของ AI team รอ Project Owner ยืนยัน · **Approved** · **Rejected** · **Open** = ยังไม่มีข้อเสนอ / ต้องการข้อมูลจาก Owner
> ห้าม implement ส่วนที่ขึ้นกับ decision ที่ยังไม่ Approved
> Last updated: 2026-11-13

| ID | Decision | Resolution | Trade-offs | Status |
|---|---|---|---|---|
| D-01 | Repo & module path | Repo `https://github.com/BreakDown-CS/opspilot-ai`; monorepo `backend/ frontend/ deploy/ docs/`; Go module `github.com/BreakDown-CS/opspilot-ai/backend` | | **Approved** 2026-11-13 |
| D-02 | Authentication | Server-side session (Postgres) + httpOnly cookie + CSRF origin check | Revoke ได้จริง; ทุก request แตะ DB (เล็กน้อย) | **Approved** 2026-11-13 |
| D-03 | Org onboarding | Seed/CLI สร้าง org+owner; Owner เพิ่มสมาชิกจาก user ที่มีอยู่; ไม่มี signup/invite email | ลด scope; demo ต้องใช้ seed accounts | **Approved** 2026-11-13 |
| D-04 | Inventory model | Multi-warehouse; `inventory_balances` snapshot; available = on_hand − reserved; no ledger | ไม่มีประวัติ stock; ledger เป็น V1.1+ | **Approved** 2026-11-13 |
| D-05 | Sales definition | Count `paid`,`fulfilled`; exclude canceled/refunded whole order; no partial returns; net = Σ(qty×price − discount), excl. VAT/shipping | ง่าย ทดสอบได้; ไม่สะท้อน partial refund | **Approved** 2026-11-13 |
| D-06 | Money, time, ranges | 1 currency/org (THB), NUMERIC(14,2), money as string in API; timestamptz UTC; org timezone Asia/Bangkok; date range inclusive, max 366 days | | **Approved** 2026-11-13 |
| D-07 | Risk policy | lookback 30d; critical ≤ 2d; high ≤ threshold (7d); insufficient_data = tracking < lookback; reorder target 14d (informational) | heuristic, ไม่ใช่ forecast | **Approved** 2026-11-13 |
| D-08 | LLM provider/model | **Zero-cost**: provider interface เดียว + adapters `mock` (tests/CI/offline demo), `openai_compat` (Groq free tier เป็นหลัก — OpenAI-compatible API + tool calling, ไม่ต้องใช้บัตร), `ollama` (local, ออฟไลน์ 100%) ; Anthropic adapter เลื่อนเป็น optional หลัง MVP | Free tier มี rate limit ต่ำและเปลี่ยนได้ตลอด; คุณภาพ tool selection ต่ำกว่า Claude → ชดเชยด้วย strict validation + golden dataset; ใช้ได้กับ **ข้อมูล demo สมมติเท่านั้น** | **Approved** 2026-11-13 (แทน Claude เพราะงบ = 0) |
| D-09 | AI limits & budget | **งบ = USD 0**. 5 iterations, 8 tool calls, 60s (Ollama: 180s), max_tokens 1,024 ต่อ call, 2,000 chars, history 6 msgs, 20 req/min/user; system prompt + tool schemas ให้สั้น (< 2k tokens) เพื่อไม่ชน TPM ของ free tier; รับ 429 → แจ้งผู้ใช้ "AI quota เต็ม ลองใหม่ใน N วินาที" | | **Approved** 2026-11-13 |
| D-10 | Deployment target | **Local only**: Docker Compose บนเครื่องตัวเอง (dev และ demo); ไม่มี VPS/domain/NPM ใน MVP; Portainer (local) optional | เปลี่ยนจาก blueprint §16 / DoD ข้อ Portainer deployment → post-MVP | **Approved** 2026-11-13 |
| D-11 | Soft delete | ไม่ใช้; products ใช้ `status=archived` | | **Approved** 2026-11-13 |
| D-12 | Seed strategy | 2 orgs, 50 SKU, 3 warehouses, ~5k orders/90d per org, deterministic; blocked in production | | **Approved** 2026-11-13 |
| D-13 | Role differences | Viewer: Dashboard/Inventory/Sales summary อ่านอย่างเดียว ไม่มี AI/compare; Analyst = Manager สำหรับการอ่าน+AI; Owner เห็น conversation ทั้ง org + จัดการสมาชิก | Manager vs Analyst ยังเหมือนกันใน MVP | **Approved** 2026-11-13 |
| D-14 | Anomaly rule | median ± 3×MAD of previous 28 days, ≥ 14 days data | อาจตัดออกถ้าเวลาไม่พอ | **Approved** 2026-11-13 |
| D-15 | UI language | UI ไทย; AI ตอบตามภาษาคำถาม; docs/code comments อังกฤษ+ไทยตาม blueprint | | **Approved** 2026-11-13 |
| D-16 | Chart library | `@ant-design/charts` (เข้ากับ antd) | bundle ใหญ่กว่า recharts | **Approved** 2026-11-13 |
| D-17 | Conversation retention | 90 days, purge job (V1.1 ถ้าไม่ทัน) | | **Approved** 2026-11-13 |
| D-18 | Demo format | **Live demo บนเครื่องตัวเอง** — เปิดโปรแกรมแล้วสาธิตให้ดู (+ อัดวิดีโอสำหรับ portfolio); ไม่มี public URL | ไม่ต้องทำ demo guard/hosting | **Approved** 2026-11-13 (ชี้แจงจาก Owner) |
| D-19 | Migration tool | goose (SQL files, Go-embeddable) | vs golang-migrate: ใกล้เคียงกัน | **Approved** 2026-11-13 |
| D-20 | Fiber version | v3 | v3 API ต่างจาก v2; ตัวอย่างออนไลน์ส่วนใหญ่เป็น v2 | **Approved** 2026-11-13 |
| D-21 | Git | `git init -b main` + remote `origin` แล้ว; ห้าม commit/push โดยไม่ได้รับอนุมัติแต่ละครั้ง | | **Done** 2026-11-13 |
| D-23 | Public demo hosting | ไม่จำเป็น — demo รันบน local (D-18) | | **Closed** 2026-11-13 |
| D-22 | Node.js | Node 24 LTS (ติดตั้งแล้ว v24.15.0) | | **Approved** 2026-11-13 |

## LLM notes (D-08, D-09) — zero-cost options

ข้อมูลจากการค้นเว็บ 2026-11-13 (ส่วนใหญ่เป็นแหล่ง third-party ตัวเลขไม่ตรงกัน) — **ต้องเช็ก limits จริงใน console ของ provider ก่อน P4**

| Option | ค่าใช้จ่าย | Tool calling | ข้อดี | ข้อจำกัด |
|---|---|---|---|---|
| **Groq free tier** (แนะนำเป็นหลัก) | ฟรี ไม่ต้องใช้บัตร | ✅ (Llama / Qwen / GPT-OSS ฯลฯ) | เร็วมาก; OpenAI-compatible | ~30 RPM และ TPM ต่ำ (รายงาน 6k–12k) → ราว 1 คำถาม/นาที; model list เปลี่ยนบ่อย |
| Google Gemini API free tier (สำรอง) | ฟรีพร้อม rate limit | ✅ | คุณภาพดี | Google ไม่ประกาศ limit ต่อ model แล้ว (ดูใน AI Studio); มีรายงานการลด quota; free tier อาจนำข้อมูลไปปรับปรุงโมเดล |
| **Ollama (local)** (สำรองออฟไลน์) | ฟรี 100% | ✅ (เช่น Qwen 3 ขนาดเล็ก) | ไม่ต้องใช้เน็ต/ไม่ส่งข้อมูลออก | เครื่องนี้: i7-8565U, RAM 48 GB, GPU Quadro P520 2 GB → รันบน CPU เป็นหลัก **ช้า** (คาด 30 วินาที–2 นาที/คำถาม สำหรับ model 3–4B); ต้องดาวน์โหลด model หลาย GB |
| `mock` provider | ฟรี | scripted | deterministic; ใช้ใน tests/CI/สาธิตตอนไม่มีเน็ต | ไม่ใช่ AI จริง |

**กลยุทธ์**
1. Phase 0–3 ไม่ใช้ LLM เลย
2. Phase 4–5 พัฒนา agent loop กับ `mock` ก่อน แล้วต่อ Groq ผ่าน `openai_compat` adapter
3. ตอน demo: ใช้ Groq; ถ้า quota เต็ม/เน็ตมีปัญหา → สลับ `LLM_PROVIDER=ollama` หรือ `mock` ได้ด้วย env เดียว
4. ห้ามใช้ข้อมูลธุรกิจจริงกับ free tier — seed data สมมติเท่านั้น
5. Claude adapter (`anthropic`) เพิ่มได้ภายหลังโดยไม่แก้ business logic — เป็นจุดขายเรื่อง provider abstraction ใน portfolio

Sources: [Groq free tier — benchlm](https://benchlm.ai/free-tier/groq), [Groq community: free tier limits](https://community.groq.com/t/is-there-a-free-tier-and-what-are-its-limits/790), [Gemini free tier — benchlm](https://benchlm.ai/free-tier/google-ai-studio), [Gemini API pricing](https://ai.google.dev/gemini-api/docs/pricing)

## Questions for Project Owner

(ไม่มีคำถามค้าง — D-08, D-09 อนุมัติแล้ว 2026-11-13)
