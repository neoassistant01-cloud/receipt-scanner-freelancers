# STATE_MACHINE.md — State Transitions + Rules

---

## Receipt Lifecycle State Machine

```
                    ┌─────────────────┐
                    │    CREATED      │
                    │ (image captured)│
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │   EXTRACTING    │
                    │  (AI processing)│
                    └────────┬────────┘
                             │
              ┌──────────────┴──────────────┐
              │                             │
              ▼                             ▼
       ┌──────────────┐            ┌──────────────┐
       │  EXTRACTED   │            │   FAILED     │
       │ (data ready) │            │ (extraction  │
       └──────┬───────┘            │   failed)     │
              │                     └──────┬───────┘
              │                              │
              ▼                              ▼
       ┌──────────────┐            ┌──────────────┐
       │   EDITING    │            │   MANUAL      │
       │ (user review)│            │   ENTRY      │
       └──────┬───────┘            └──────┬───────┘
              │                              │
              ▼                              │
       ┌──────────────┐                      │
       │   SAVED      │◄─────────────────────┘
       │ (confirmed)  │
       └──────┬───────┘
              │
              ▼
       ┌──────────────┐
       │   EXPORTED   │
       │ (to QB/Xero) │
       └──────────────┘
```

---

## State Transition Table

| Current State | Event | Guard | Next State | Side Effects |
|---------------|-------|-------|------------|--------------|
| CREATED | capture_complete | image_valid | EXTRACTING | Call AI extraction API |
| CREATED | capture_complete | image_invalid | FAILED | Show "Image unclear" error |
| EXTRACTING | extraction_success | confidence >= 0.7 | EDITING | Populate extracted fields |
| EXTRACTING | extraction_success | confidence < 0.7 | EDITING | Populate fields, show low confidence warning |
| EXTRACTING | extraction_failed | network_error | FAILED | Queue retry, show offline message |
| EXTRACTING | extraction_failed | api_error | FAILED | Show error, offer manual entry |
| FAILED | user_retry | - | EXTRACTING | Retry extraction |
| FAILED | user_manual | - | MANUAL_ENTRY | Open manual entry form |
| EDITING | user_save | all_required_fields | SAVED | Persist to SQLite, index for search |
| EDITING | user_discard | - | (delete) | Delete temp image |
| SAVED | user_export | provider_connected | EXPORTED | Call provider API |
| SAVED | user_export | !provider_connected | SAVED | Prompt OAuth flow first |
| EXPORTED | export_success | - | EXPORTED | Update export record, show success |
| EXPORTED | export_failed | retry_available | (retry) | Queue retry with backoff |
| EXPORTED | export_failed | !retry_available | SAVED | Show error, allow manual retry |

---

## Integration Connection State Machine

```
       ┌──────────────┐
       │   DISCONNECTED  │
       └──────┬───────┘
              │ (user initiates OAuth)
              ▼
       ┌──────────────┐
       │  AUTH_PENDING │
       │ (OAuth redirect)│
       └──────┬───────┘
              │ (OAuth callback success)
              ▼
       ┌──────────────┐
       │    CONNECTED  │
       └──────┬───────┘
              │
              ▼
       ┌──────────────┐
       │   ACTIVE      │
       │ (token valid) │
       └──────┬───────┘
              │ (token expires)
              ▼
       ┌──────────────┐
       │    EXPIRED    │
       └──────┬───────┘
              │ (auto-refresh success)
              ▼
       ┌──────────────┐
       │    ACTIVE     │
       └──────────────┘

       (user disconnects)
              │
              ▼
       ┌──────────────┐
       │  DISCONNECTED │
       └──────────────┘
```

---

## Invalid Transition Behavior

| Invalid Transition | Behavior |
|--------------------|----------|
| SAVED → EXTRACTING | No-op, already extracted |
| FAILED → EXPORTED | No-op, must extract first |
| DISCONNECTED → EXPORTED | Return error code ERR_PROVIDER_NOT_CONNECTED |
| ACTIVE → CONNECTED | No-op, already connected |

---

## Concurrency Rules

- **Receipt edits:** Last-write-wins with timestamp, no automatic merge
- **Cloud sync:** Optimistic local writes, conflict resolution on sync
- **Export:** Idempotency key prevents duplicate exports
- **Token refresh:** Background refresh with token lease (5 min)

---

## Error Codes

| Code | Meaning |
|------|---------|
| ERR_IMAGE_CAPTURE_FAILED | Camera or file error |
| ERR_IMAGE_UNCLEAR | Low quality, retry needed |
| ERR_EXTRACTION_FAILED | AI service error |
| ERR_EXTRACTION_OFFLINE | No network |
| ERR_PROVIDER_NOT_CONNECTED | No OAuth |
| ERR_PROVIDER_TOKEN_EXPIRED | Refresh needed |
| ERR_PROVIDER_RATE_LIMITED | Backoff required |
| ERR_PROVIDER_API_ERROR | Provider returned error |
| ERR_SAVE_FAILED | Database error |
