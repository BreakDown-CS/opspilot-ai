# 04 — API Contract (v1)

> Status: Draft — ต้องอนุมัติก่อนเริ่ม Frontend

## 1. General

- Base path: `/api/v1`
- Content-Type: `application/json; charset=utf-8`
- Auth: cookie `opspilot_session`; non-GET ต้องส่ง header `X-Requested-With: opspilot` และ `Origin` ต้องอยู่ใน allow-list
- IDs ใน API เป็น UUID (`public_id`)
- Dates: `YYYY-MM-DD` ตาม org timezone; timestamps เป็น RFC 3339 UTC
- Money: string decimal (`"12345.50"`) เพื่อกัน float error + `currency` ใน `meta`
- Request body limit: 16 KB
- Header `X-Request-ID` ตอบกลับทุก response

### Envelope

```json
{ "success": true, "message": "OK", "data": {}, "meta": {}, "error": null }
```

```json
{
  "success": false,
  "message": "Validation failed",
  "data": null,
  "meta": { "request_id": "01J..." },
  "error": {
    "code": "VALIDATION_FAILED",
    "details": [{ "field": "from", "issue": "must be on or before 'to'" }]
  }
}
```

### Error codes

| HTTP | code | ใช้เมื่อ |
|---|---|---|
| 400 | `VALIDATION_FAILED` | input ผิด |
| 400 | `MESSAGE_TOO_LONG` | AI message > 2,000 chars |
| 401 | `AUTH_REQUIRED` | ไม่มี/หมดอายุ session |
| 401 | `AUTH_INVALID_CREDENTIALS` | login ผิด |
| 403 | `FORBIDDEN` | role ไม่มีสิทธิ์ |
| 403 | `CSRF_REJECTED` | Origin/header ไม่ผ่าน |
| 404 | `NOT_FOUND` | ไม่พบ หรือเป็นของ org อื่น |
| 409 | `CONFLICT` | เช่น ลบ owner คนสุดท้าย |
| 413 | `PAYLOAD_TOO_LARGE` | |
| 429 | `RATE_LIMITED` | มี `Retry-After` |
| 502 | `AI_PROVIDER_ERROR` | provider ล่ม/ตอบผิดรูปแบบ |
| 504 | `AI_TIMEOUT` | |
| 500 | `INTERNAL_ERROR` | ไม่มีรายละเอียดภายใน |

### Pagination & sorting

`?page=1&page_size=20` (max 100), `?sort=field` / `?sort=-field` (allow-list ต่อ endpoint)

```json
"meta": { "page": 1, "page_size": 20, "total": 132 }
```

### Date range

`?from=2026-10-01&to=2026-10-31` — default 30 วันล่าสุด, max 366 วัน; response `meta` คืน `{from, to, timezone}` ที่ใช้จริง

## 2. Endpoints

| Method | Path | Roles | Notes |
|---|---|---|---|
| POST | `/auth/login` | public | rate limited |
| POST | `/auth/logout` | any | |
| GET | `/me` | any | |
| POST | `/me/active-organization` | any | body `{organization_id}` — ต้องเป็นสมาชิก |
| GET | `/organization/members` | owner | |
| POST | `/organization/members` | owner | `{email, role}` |
| PATCH | `/organization/members/:userId` | owner | `{role}` |
| DELETE | `/organization/members/:userId` | owner | |
| GET | `/dashboard/summary` | all | date range |
| GET | `/dashboard/sales-trend` | all | `granularity=day|month` |
| GET | `/inventory` | owner, manager, analyst, viewer | `warehouse_id, q`, paging, sort |
| GET | `/inventory/risks` | owner, manager, analyst, viewer | `threshold_days, lookback_days, warehouse_id, risk_level[]` |
| GET | `/warehouses` | all | |
| GET | `/sales/summary` | owner, manager, analyst, viewer | date range |
| GET | `/sales/top-products` | owner, manager, analyst, viewer | date range, `limit, rank_by=net_sales|units` |
| GET | `/sales/compare` | owner, manager, analyst | `from,to,prev_from?,prev_to?` |
| GET | `/sales/anomalies` | owner, manager, analyst | date range |
| POST | `/ai/conversations` | owner, manager, analyst | `{title?}` |
| GET | `/ai/conversations` | owner, manager, analyst | own; owner `?scope=org` |
| GET | `/ai/conversations/:id/messages` | owner, manager, analyst | |
| POST | `/ai/conversations/:id/messages` | owner, manager, analyst | ask; rate limited |
| GET | `/ai/executions/:id` | owner, manager, analyst | ต้องเป็นของตนเอง หรือ owner |
| GET | `/healthz` | public | liveness |
| GET | `/readyz` | public | DB ping |

(Role matrix ฉบับเต็มดู [05 §8](05-ai-tools-and-authorization.md))

## 3. Examples

### POST /auth/login

```json
// request
{ "email": "manager@demo-coffee.test", "password": "••••••••" }
// 200
{ "success": true, "message": "Logged in",
  "data": { "user": { "id": "0193...", "display_name": "Demo Manager" },
            "active_organization": { "id": "0193...", "name": "Demo Coffee Supply", "role": "manager" } },
  "meta": {}, "error": null }
```

### GET /dashboard/summary?from=2026-10-01&to=2026-10-31

```json
{ "success": true, "message": "OK",
  "data": { "net_sales": "482310.00", "gross_sales": "495000.00", "orders": 1612,
            "units_sold": 5230, "average_order_value": "299.20",
            "in_stock_skus": 46, "at_risk_skus": 5 },
  "meta": { "from": "2026-10-01", "to": "2026-10-31", "timezone": "Asia/Bangkok",
            "currency": "THB", "definition_version": "sales-v1" },
  "error": null }
```
*(ตัวเลขตัวอย่าง ไม่ใช่ข้อมูลจริง)*

### GET /inventory/risks?threshold_days=7

```json
{ "success": true, "message": "OK",
  "data": {
    "summary": { "critical": 2, "high": 3, "normal": 38, "no_recent_sales": 4, "insufficient_data": 3 },
    "items": [
      { "product_id": "0193...", "sku": "SKU-001", "name": "Arabica Beans 1kg",
        "available_stock": 10, "units_sold": 150, "average_daily_sales": 5.0,
        "coverage_days": 2.0, "risk_level": "critical",
        "suggested_reorder_qty": 60, "suggestion_type": "informational" } ] },
  "meta": { "filters": { "threshold_days": 7, "lookback_days": 30, "warehouse_id": null },
            "calculation_version": "inventory-risk-v1", "generated_at": "2026-11-13T03:00:00Z",
            "page": 1, "page_size": 20, "total": 5 },
  "error": null }
```

### GET /sales/compare

```json
{ "data": { "current":  { "from": "2026-11-03", "to": "2026-11-09", "net_sales": "120000.00" },
            "previous": { "from": "2026-10-27", "to": "2026-11-02", "net_sales": "100000.00" },
            "change": "20000.00", "percentage_change": 20.0, "change_status": "up" } }
```

### POST /ai/conversations/:id/messages

```json
// request
{ "content": "สินค้าตัวไหนมีโอกาสหมดภายใน 7 วัน?" }
// 200
{ "success": true, "message": "OK",
  "data": {
    "message_id": "0193...",
    "answer": "มีสินค้า 2 รายการที่อยู่ในระดับ critical ...",
    "key_findings": ["SKU-001 เหลือ coverage 2.0 วัน", "..."],
    "evidence": [
      { "tool": "GetInventoryRisk", "tool_execution_id": "0193...",
        "filters": { "threshold_days": 7, "lookback_days": 30 },
        "calculation_version": "inventory-risk-v1",
        "data_points": [ { "sku": "SKU-001", "coverage_days": 2.0, "risk_level": "critical" } ] } ],
    "limitations": ["คำนวณจากยอดขายเฉลี่ย 30 วัน ไม่ใช่การพยากรณ์"],
    "recommendations": ["พิจารณาตรวจสอบการเติม SKU-001 (ข้อเสนอแนะ ไม่ใช่การดำเนินการ)"],
    "execution_id": "0193...",
    "fidelity_passed": true },
  "meta": { "model": "…", "latency_ms": 6120 },
  "error": null }
```

### GET /ai/executions/:id

```json
{ "data": { "id": "0193...", "status": "succeeded", "provider": "anthropic", "model": "…",
            "iterations": 2, "input_tokens": 2310, "output_tokens": 412, "latency_ms": 6120,
            "started_at": "...", "finished_at": "...",
            "tool_calls": [ { "sequence": 1, "tool": "GetInventoryRisk",
                              "arguments": { "threshold_days": 7 }, "status": "succeeded",
                              "duration_ms": 38, "result_summary": { "critical": 2, "high": 3 } } ] } }
```
