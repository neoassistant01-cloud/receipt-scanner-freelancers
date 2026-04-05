# DOMAIN_MODEL.md — Entities + Invariants

---

## Entities

### Entity: User
```
- id: UUID (primary key)
- email: String (unique, indexed)
- passwordHash: String (nullable, for email auth)
- createdAt: DateTime
- updatedAt: DateTime
- lastLoginAt: DateTime
- preferences: JSON (theme, defaultCategory, etc.)
```
**Owner:** User Service

---

### Entity: Receipt
```
- id: UUID (primary key)
- userId: UUID (foreign key → User.id)
- imagePath: String (local file path)
- imageThumbnail: String (compressed thumbnail path)
- vendorName: String (nullable)
- receiptDate: Date
- totalAmount: Decimal(10,2)
- currency: String (default: "USD")
- lineItems: JSON (array of {description, quantity, unitPrice, amount})
- category: String (IRS category code)
- notes: String (nullable)
- createdAt: DateTime
- updatedAt: DateTime
- extractedAt: DateTime (when AI extracted data)
- extractionConfidence: Float (0.0-1.0)
```
**Owner:** Receipt Service

**Invariant:** INV-001 — totalAmount MUST equal sum of lineItem amounts (if lineItems present)

---

### Entity: ExportRecord
```
- id: UUID (primary key)
- receiptId: UUID (foreign key → Receipt.id)
- provider: Enum (QUICKBOOKS, XERO)
- externalId: String (provider's transaction ID)
- exportedAt: DateTime
- status: Enum (PENDING, SUCCESS, FAILED)
- errorMessage: String (nullable)
- retryCount: Integer (default: 0)
```
**Owner:** Export Service

---

### Entity: Category
```
- id: String (IRS category code, primary key)
- name: String
- icon: String (SF Symbol name)
- parentCategory: String (nullable, for subcategories)
```
**System-owned, read-only**

---

### Entity: IntegrationConnection
```
- id: UUID (primary key)
- userId: UUID (foreign key → User.id)
- provider: Enum (QUICKBOOKS, XERO)
- accessToken: String (encrypted at rest)
- refreshToken: String (encrypted at rest)
- tokenExpiresAt: DateTime
- tenantId: String (provider-specific, e.g., QuickBooks company ID)
- connectedAt: DateTime
- lastSyncAt: DateTime
- status: Enum (ACTIVE, EXPIRED, ERROR)
```
**Owner:** Integration Service

**Invariant:** INV-002 — Each user can have at most ONE connection per provider

---

### Entity: Backup
```
- id: UUID (primary key)
- userId: UUID (foreign key → User.id)
- storageProvider: Enum (ICLOUD, GOOGLE_DRIVE)
- backupPath: String
- createdAt: DateTime
- sizeBytes: Integer
- receiptCount: Integer
- checksum: String (SHA-256)
```
**Owner:** Backup Service

---

## Relationships

```
User (1) ──┬── (N) Receipt
           │
           ├── (N) IntegrationConnection
           │
           └── (N) Backup

Receipt (1) ──┬── (N) ExportRecord
```

---

## Derived Fields

- **Receipt.displayDate:** Format receiptDate as "MMM dd, yyyy"
- **Receipt.monthYear:** Derived from receiptDate for grouping (e.g., "2024-01")
- **Receipt.categoryName:** Lookup from Category entity by category code
- **User.receiptCount:** Count of Receipts where userId = User.id
- **User.totalSpendingSum:** Sum of Receipt.totalAmount for user in current year

---

## Invariants (Rules that MUST always be true)

| ID | Rule | Enforcement |
|----|------|-------------|
| INV-001 | totalAmount = SUM(lineItems.amount) OR lineItems is empty | Business logic in Receipt.save() |
| INV-002 | User has max 1 IntegrationConnection per provider | Database unique constraint on (userId, provider) |
| INV-003 | Receipt.receiptDate cannot be in the future | Validation in Receipt.update() |
| INV-004 | Receipt.imagePath points to existing file | File system check before display |
| INV-005 | ExportRecord.externalId is unique per provider | Database unique constraint on (provider, externalId) |
| INV-006 | Backup.checksum matches actual file content | Verification on restore |
