# API_SPEC.md — Endpoints/Events + Schemas

---

## Overview

This app is mobile-first with local-first data. Most operations happen on-device. Cloud APIs are used for:
1. User authentication (account creation/login)
2. Receipt extraction AI service
3. Accounting software integration (QuickBooks/Xero)
4. Optional cloud backup

---

## API-001: User Registration

**Method:** POST  
**Path:** `/api/v1/auth/register`  
**Auth:** None (public)

**Request:**
```json
{
  "email": "string (valid email)",
  "password": "string (min 8 chars)",
  "source": "string (ios/android)"
}
```

**Response (201):**
```json
{
  "userId": "uuid",
  "email": "string",
  "accessToken": "string (JWT)",
  "refreshToken": "string"
}
```

**Error Codes:**
- 400: Invalid email format
- 409: Email already registered
- 429: Rate limited

---

## API-002: User Login

**Method:** POST  
**Path:** `/api/v1/auth/login`  
**Auth:** None (public)

**Request:**
```json
{
  "email": "string",
  "password": "string"
}
```

**Response (200):** Same as API-001

**Error Codes:**
- 401: Invalid credentials
- 429: Rate limited

---

## API-003: AI Receipt Extraction

**Method:** POST  
**Path:** `/api/v1/extract/receipt`  
**Auth:** Bearer JWT

**Request:**
```json
{
  "imageData": "base64 string",
  "imageFormat": "jpeg/png/webp"
}
```

**Response (200):**
```json
{
  "vendorName": "string",
  "receiptDate": "ISO8601 date",
  "totalAmount": "decimal string",
  "currency": "string",
  "lineItems": [
    {
      "description": "string",
      "quantity": "number",
      "unitPrice": "string",
      "amount": "string"
    }
  ],
  "confidence": {
    "vendorName": 0.95,
    "receiptDate": 0.99,
    "totalAmount": 0.97,
    "lineItems": 0.85
  }
}
```

**Error Codes:**
- 400: Invalid image data
- 413: Image too large (max 10MB)
- 422: Could not extract receipt data
- 429: Rate limited

---

## API-004: OAuth2 - QuickBooks Initiate

**Method:** GET  
**Path:** `/api/v1/integrations/quickbooks/auth`  
**Auth:** Bearer JWT

**Response (302):** Redirect to QuickBooks OAuth URL

**Query Params:** `redirect_uri`, `state` (CSRF token)

---

## API-005: QuickBooks OAuth Callback

**Method:** POST  
**Path:** `/api/v1/integrations/quickbooks/callback`  
**Auth:** None (callback from QuickBooks)

**Request:**
```json
{
  "code": "string (auth code)",
  "realmId": "string (company ID)",
  "state": "string (CSRF token)"
}
```

**Response (200):**
```json
{
  "connectionId": "uuid",
  "provider": "QUICKBOOKS",
  "companyName": "string"
}
```

**Error Codes:**
- 400: Invalid callback params
- 401: CSRF token mismatch

---

## API-006: QuickBooks Export Expense

**Method:** POST  
**Path:** `/api/v1/integrations/quickbooks/expense`  
**Auth:** Bearer JWT

**Request:**
```json
{
  "receiptId": "uuid",
  "vendorName": "string",
  "date": "ISO8601 date",
  "amount": "decimal string",
  "category": "string (IRS code)",
  "notes": "string (optional)",
  "imageBase64": "string (optional, receipt attachment)"
}
```

**Response (201):**
```json
{
  "exportId": "uuid",
  "externalId": "string (QuickBooks transaction ID)",
  "status": "SUCCESS"
}
```

**Error Codes:**
- 400: Invalid request
- 401: Token expired (trigger refresh)
- 403: No permission for this company
- 404: Vendor not found (create or fail)
- 422: Validation error
- 429: Rate limited (QuickBooks: 100 req/min)

---

## API-007: OAuth2 - Xero Initiate

**Method:** GET  
**Path:** `/api/v1/integrations/xero/auth`  
**Auth:** Bearer JWT

**Response (302):** Redirect to Xero OAuth URL

---

## API-008: Xero OAuth Callback

**Method:** POST  
**Path:** `/api/v1/integrations/xero/callback`  
**Auth:** None (callback from Xero)

**Request:**
```json
{
  "code": "string",
  "sessionState": "string"
}
```

**Response (200):** Same as API-005

---

## API-009: Xero Export Expense

**Method:** POST  
**Path:** `/api/v1/integrations/xero/expense`  
**Auth:** Bearer JWT

**Request:** Same as API-006

**Response (201):** Same as API-006

**Error Codes:** Same as API-006 (Xero rate limit: 60 req/min)

---

## API-010: Cloud Backup - Create

**Method:** POST  
**Path:** `/api/v1/backup`  
**Auth:** Bearer JWT

**Request:**
```json
{
  "data": "encrypted base64",
  "checksum": "SHA-256 string"
}
```

**Response (201):**
```json
{
  "backupId": "uuid",
  "createdAt": "ISO8601 datetime",
  "sizeBytes": "number"
}
```

**Error Codes:**
- 400: Invalid data
- 413: Backup too large (max 500MB)
- 507: Insufficient storage

---

## API-011: Cloud Backup - Restore

**Method:** GET  
**Path:** `/api/v1/backup/latest`  
**Auth:** Bearer JWT

**Response (200):**
```json
{
  "backupId": "uuid",
  "data": "encrypted base64",
  "checksum": "string",
  "createdAt": "ISO8601 datetime",
  "receiptCount": "number"
}
```

**Error Codes:**
- 404: No backup found

---

## API-012: Token Refresh

**Method:** POST  
**Path:** `/api/v1/auth/refresh`  
**Auth:** None (public with refresh token)

**Request:**
```json
{
  "refreshToken": "string"
}
```

**Response (200):**
```json
{
  "accessToken": "string (new JWT)",
  "refreshToken": "string (new)"
}
```

**Error Codes:**
- 401: Invalid/expired refresh token

---

## Rate Limits

| Endpoint | Limit |
|-----------|-------|
| Auth (register/login) | 10 req/min per IP |
| Receipt extraction | 30 req/min per user |
| QuickBooks API | 100 req/min (per QuickBooks) |
| Xero API | 60 req/min (per Xero) |
| Backup | 5 req/day per user |

---

## Idempotency

- Receipt extraction: Not idempotent (different images = different results)
- Export to QuickBooks/Xero: Idempotent via exportIdempotencyKey = `${receiptId}-${provider}`
- Token refresh: Not applicable (new tokens generated)
