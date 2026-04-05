# DATA_MODEL.md — Persistence + Indexes + Migrations

---

## Overview

The app uses a hybrid storage strategy:
- **Local-first:** SQLite for structured data, file system for images
- **Cloud (optional):** Backend API for auth, extraction, integrations, backup

---

## Local Storage: SQLite

### Table: users

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | TEXT | PRIMARY KEY | UUID |
| email | TEXT | UNIQUE, NOT NULL | User email |
| password_hash | TEXT | NULLABLE | For email auth |
| created_at | TEXT | NOT NULL | ISO8601 timestamp |
| updated_at | TEXT | NOT NULL | ISO8601 timestamp |
| last_login_at | TEXT | NULLABLE | ISO8601 timestamp |
| preferences | TEXT | NULLABLE | JSON blob |

**Indexes:** `idx_users_email` on email

---

### Table: receipts

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | TEXT | PRIMARY KEY | UUID |
| user_id | TEXT | NOT NULL, FK → users.id | Owner |
| image_path | TEXT | NOT NULL | Local file path |
| image_thumbnail | TEXT | NULLABLE | Compressed thumbnail |
| vendor_name | TEXT | NULLABLE | Extracted vendor |
| receipt_date | TEXT | NOT NULL | ISO8601 date |
| total_amount | REAL | NOT NULL | Amount in USD cents |
| currency | TEXT | NOT NULL DEFAULT 'USD' | Currency code |
| line_items | TEXT | NULLABLE | JSON array |
| category | TEXT | NOT NULL | IRS category code |
| notes | TEXT | NULLABLE | User notes |
| created_at | TEXT | NOT NULL | ISO8601 timestamp |
| updated_at | TEXT | NOT NULL | ISO8601 timestamp |
| extracted_at | TEXT | NULLABLE | When AI ran |
| extraction_confidence | REAL | NULLABLE | 0.0-1.0 |
| is_synced | INTEGER | NOT NULL DEFAULT 0 | Cloud sync status |

**Indexes:**
- `idx_receipts_user_id` on user_id
- `idx_receipts_receipt_date` on receipt_date
- `idx_receipts_category` on category
- `idx_receipts_vendor_name` on vendor_name (FTS5)

---

### Table: export_records

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | TEXT | PRIMARY KEY | UUID |
| receipt_id | TEXT | NOT NULL, FK → receipts.id | Source receipt |
| provider | TEXT | NOT NULL | 'QUICKBOOKS' or 'XERO' |
| external_id | TEXT | NULLABLE | Provider's transaction ID |
| exported_at | TEXT | NOT NULL | ISO8601 timestamp |
| status | TEXT | NOT NULL | PENDING/SUCCESS/FAILED |
| error_message | TEXT | NULLABLE | Error details |
| retry_count | INTEGER | NOT NULL DEFAULT 0 | Retry attempts |

**Indexes:**
- `idx_export_receipt_id` on receipt_id
- `idx_export_provider_external` on (provider, external_id) UNIQUE

---

### Table: integration_connections

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | TEXT | PRIMARY KEY | UUID |
| user_id | TEXT | NOT NULL, FK → users.id | Owner |
| provider | TEXT | NOT NULL | 'QUICKBOOKS' or 'XERO' |
| access_token | TEXT | NOT NULL | Encrypted |
| refresh_token | TEXT | NOT NULL | Encrypted |
| token_expires_at | TEXT | NOT NULL | ISO8601 timestamp |
| tenant_id | TEXT | NULLABLE | Company ID |
| connected_at | TEXT | NOT NULL | ISO8601 timestamp |
| last_sync_at | TEXT | NULLABLE | ISO8601 timestamp |
| status | TEXT | NOT NULL DEFAULT 'ACTIVE' | ACTIVE/EXPIRED/ERROR |

**Indexes:**
- `idx_conn_user_provider` on (user_id, provider) UNIQUE

---

### Table: backups

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | TEXT | PRIMARY KEY | UUID |
| user_id | TEXT | NOT NULL, FK → users.id | Owner |
| storage_provider | TEXT | NOT NULL | 'ICLOUD' or 'GOOGLE_DRIVE' |
| backup_path | TEXT | NOT NULL | Cloud path |
| created_at | TEXT | NOT NULL | ISO8601 timestamp |
| size_bytes | INTEGER | NOT NULL | Size |
| receipt_count | INTEGER | NOT NULL | Count |
| checksum | TEXT | NOT NULL | SHA-256 |

**Indexes:** `idx_backups_user_id` on user_id

---

## Local Storage: File System

### Image Storage

**Location:** App documents directory  
**Naming:** `{receipt_id}.jpg` (original), `{receipt_id}_thumb.jpg` (thumbnail)  
**Compression:** Original: JPEG quality 85%, Thumbnail: 200x200, JPEG quality 70%

**Retention:** Keep until receipt deleted by user

---

## Migrations

### Version 1.0 (Initial)

```sql
-- Create tables
CREATE TABLE users (...);
CREATE TABLE receipts (...);
CREATE TABLE export_records (...);
CREATE TABLE integration_connections (...);
CREATE TABLE backups (...);

-- Create indexes
CREATE INDEX idx_receipts_user_id ON receipts(user_id);
CREATE INDEX idx_receipts_receipt_date ON receipts(receipt_date);
CREATE INDEX idx_receipts_category ON receipts(category);
CREATE INDEX idx_export_receipt_id ON export_records(receipt_id);
CREATE INDEX idx_conn_user_provider ON integration_connections(user_id, provider);

-- Enable FTS for vendor search
CREATE VIRTUAL TABLE receipts_fts USING fts5(vendor_name, content=receipts, content_rowid=rowid);
CREATE TRIGGER receipts_ai AFTER INSERT ON receipts BEGIN
  INSERT INTO receipts_fts(rowid, vendor_name) VALUES (new.rowid, new.vendor_name);
END;
CREATE TRIGGER receipts_ad AFTER DELETE ON receipts BEGIN
  INSERT INTO receipts_fts(receipts_fts, rowid, vendor_name) VALUES('delete', old.rowid, old.vendor_name);
END;
CREATE TRIGGER receipts_au AFTER UPDATE ON receipts BEGIN
  INSERT INTO receipts_fts(receipts_fts, rowid, vendor_name) VALUES('delete', old.rowid, old.vendor_name);
  INSERT INTO receipts_fts(rowid, vendor_name) VALUES (new.rowid, new.vendor_name);
END;
```

---

## Data Retention

| Data Type | Retention | Deletion |
|-----------|-----------|----------|
| Receipt images | Until user deletes | User-initiated only |
| Receipt data | Until user deletes | User-initiated only |
| Export records | 2 years | Auto-archive to cold storage |
| Backups | Keep last 5 | Auto-delete oldest |
| Session tokens | 30 days | Auto-expire |

---

## Data Export/Portability

- User can export all data as JSON via Settings
- Export includes: receipts, categories, settings
- Images exported as base64 in separate zip
