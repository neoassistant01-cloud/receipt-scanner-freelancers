# PRD.md — Product Requirements Document

## Product: Receipt Scanner for Freelancers

**Version:** 1.0  
**Target:** US freelancers and independent contractors  
**Problem:** Losing 3-5 hours monthly at tax time manually sorting and categorizing receipts

---

## REQ-001: Receipt Image Capture
- Statement: "System must allow users to capture receipt images using the device camera"
- Rationale: Mobile-first experience requires effortless receipt capture
- Priority: P0
- Acceptance: Camera opens within 500ms, captures at minimum 1080p resolution, auto-detects receipt edges
- Notes: Must handle low-light conditions, auto-focus, flash control

## REQ-002: Receipt Image Import from Gallery
- Statement: "System must allow users to import existing receipt images from device photo gallery"
- Rationale: Users may receive receipts via email/messaging and save to gallery first
- Priority: P1
- Acceptance: Photo picker opens, supports JPEG/PNG/WEBP, imports within 2 seconds

## REQ-003: AI Receipt Data Extraction
- Statement: "System must extract vendor name, date, total amount, and line items from receipt images using AI"
- Rationale: Manual data entry is time-consuming and error-prone; AI automation solves the core pain point
- Priority: P0
- Acceptance: Extract vendor name (85%+ accuracy), date (95%+), total amount (95%+), line items (80%+) within 3 seconds of image capture
- Notes: Must handle crumpled, faded, or partially visible receipts

## REQ-004: One-Tap Auto-Categorization
- Statement: "System must automatically suggest a tax category with one tap to confirm"
- Rationale: Users want to categorize in under 3 seconds per receipt
- Priority: P0
- Acceptance: Display single suggested category based on vendor/line items, user confirms with single tap
- Notes: Categories follow IRS Schedule C categories (Advertising, Car/Truck, Office Supplies, Meals, Travel, etc.)

## REQ-005: Manual Category Override
- Statement: "System must allow users to change the suggested category to any available IRS category"
- Rationale: AI suggestions are not always correct; user must have final say
- Priority: P0
- Acceptance: Category picker shows all IRS Schedule C categories, selection persists immediately

## REQ-006: QuickBooks Export
- Statement: "System must export receipt data to QuickBooks Online as expense transactions"
- Rationale: QuickBooks is the dominant accounting software for US freelancers
- Priority: P0
- Acceptance: OAuth2 authentication, creates expense with vendor, date, amount, category, and receipt attachment
- Notes: Must handle QuickBooks API rate limits, sync status feedback

## REQ-007: Xero Export
- Statement: "System must export receipt data to Xero as expense transactions"
- Rationale: Xero is a popular alternative for freelancers
- Priority: P1
- Acceptance: OAuth2 authentication, creates expense with vendor, date, amount, category, and receipt attachment

## REQ-008: Receipt Storage
- Statement: "System must store all receipt images and extracted data locally on device"
- Rationale: Offline access, privacy, no recurring server costs
- Priority: P0
- Acceptance: Images stored as compressed JPEG (quality 80%), data stored in SQLite, retrieval under 200ms

## REQ-009: Receipt Search
- Statement: "System must allow users to search receipts by vendor name, date range, amount range, and category"
- Rationale: Quick retrieval during tax preparation
- Priority: P1
- Acceptance: Search returns results within 500ms, supports partial matching

## REQ-010: Monthly Summary Report
- Statement: "System must generate a monthly summary showing total spending by category"
- Rationale: Tax preparation insight, spending awareness
- Priority: P2
- Acceptance: Report displays bar chart of categories with totals, exportable as PDF

## REQ-011: Data Backup to iCloud/Google Drive
- Statement: "System must allow users to backup receipt data to cloud storage"
- Rationale: Device loss protection, cross-device access
- Priority: P2
- Acceptance: Manual backup trigger, restore from backup, encrypted transfer

## REQ-012: Multi-User Support (Account)
- Statement: "System must support user accounts for data sync across devices"
- Rationale: Freelancers use multiple devices (phone + tablet)
- Priority: P2
- Acceptance: Email/password auth, cloud sync of receipts, conflict resolution on duplicate edits
