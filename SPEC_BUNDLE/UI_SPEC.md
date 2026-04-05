# UI_SPEC.md — UI Component Inventory

---

## Screen Inventory

### SCREEN-001: Splash/Welcome
- Purpose: First launch, value prop
- Components: Logo, tagline, CTA button ("Get Started"), login link
- Linked to: REQ-012, FLOW-001

### SCREEN-002: Login/Register
- Purpose: Authentication
- Components: Email input, password input, "Sign In" button, "Create Account" link, Apple/Google SSO buttons
- Linked to: REQ-012, FLOW-001

### SCREEN-003: Onboarding (3 slides)
- Purpose: Tutorial
- Components: Illustration, title, description, pagination dots, skip button, next/finish buttons
- Linked to: REQ-012, FLOW-001

### SCREEN-004: Home (Receipt List)
- Purpose: Main dashboard
- Components: Header (title, search icon, profile icon), receipt cards list, FAB "+" button, bottom nav (Home, Reports, Settings)
- Linked to: REQ-008, REQ-009, FLOW-002, FLOW-003

### SCREEN-005: Camera Capture
- Purpose: Receipt capture
- Components: Camera preview, receipt edge detection overlay, capture button, flash toggle, gallery import button
- Linked to: REQ-001, FLOW-002

### SCREEN-006: Receipt Preview/Edit
- Purpose: Review extracted data
- Components: Receipt image thumbnail, vendor field (editable), date picker, amount field, category chip (tappable), line items list, save button
- Linked to: REQ-003, REQ-004, REQ-005, FLOW-002

### SCREEN-007: Receipt Detail
- Purpose: Full receipt view/edit
- Components: Zoomable receipt image, all fields as editable inputs, category picker, export button, delete button
- Linked to: REQ-005, REQ-006, REQ-007, FLOW-004

### SCREEN-008: Search
- Purpose: Find receipts
- Components: Search input, filter chips (date, category, amount), results list
- Linked to: REQ-009, FLOW-008

### SCREEN-009: Reports/Monthly Summary
- Purpose: Tax preparation insight
- Components: Month selector, bar chart (category vs amount), category breakdown list, export PDF button
- Linked to: REQ-010, FLOW-009

### SCREEN-010: Settings
- Purpose: App configuration
- Components: Profile section, backup options, integrations (QuickBooks, Xero), about, logout
- Linked to: REQ-006, REQ-007, REQ-011, FLOW-007

### SCREEN-011: Integration Setup (OAuth)
- Purpose: Connect accounting software
- Components: OAuth web view (system), loading state, success/failure callback handling
- Linked to: REQ-006, REQ-007, FLOW-007

---

## Component Inventory

### COMP-001: ReceiptCard
- States: Default, pressed, selected
- Displays: Vendor name, date, amount, category chip, export status icon
- Interaction: Tap to open detail

### COMP-002: CategoryChip
- States: Default, selected, editable
- Displays: Category name with icon
- Interaction: Tap to change

### COMP-003: AmountInput
- States: Default, focused, error
- Validation: Numeric only, 2 decimal places, max $999,999.99

### COMP-004: DatePicker
- States: Default, open
- Constraint: Cannot select future dates

### COMP-005: FAB (Floating Action Button)
- States: Default, pressed
- Action: Opens capture flow

### COMP-006: SearchBar
- States: Collapsed, expanded, focused
- Behavior: Debounced input, clear button

### COMP-007: CategoryPicker
- States: Default, open, selected
- Display: Modal bottom sheet with category list and search

### COMP-008: ExportButton
- States: Default, loading, success, error
- Actions: Tap triggers export flow

### COMP-009: ChartBar
- Purpose: Monthly summary visualization
- Interaction: Tap shows drill-down

### COMP-010: EmptyState
- Display: Icon, message, optional CTA

---

## IRS Schedule C Categories (Full List)

1. Advertising
2. Car and Truck Expenses
3. Commissions and Fees
4. Contract Labor
5. Depletion
6. Employee Benefits
7. Insurance
8. Legal and Professional Services
9. Office Expenses
10. Rent or Lease (Vehicle, Machinery, Equipment)
11. Repairs and Maintenance
12. Supplies
13. Taxes and Licenses
14. Travel
15. Meals (50%)
16. Utilities
17. Wages
18. Other Expenses

---

## Accessibility Requirements

- All interactive elements have accessibility labels
- Minimum touch target: 44x44pt
- Color contrast ratio: 4.5:1 minimum
- VoiceOver support for all screens
- Dynamic Type support (text scaling)
- Reduce Motion respect for animations

---

## Copy Strings (Key Placeholders)

```
app.name = "ReceiptSnap"
welcome.tagline = "Scan. Categorize. Done."
btn.get_started = "Get Started"
btn.save = "Save"
btn.cancel = "Cancel"
btn.delete = "Delete"
btn.export = "Export"
label.vendor = "Vendor"
label.date = "Date"
label.amount = "Amount"
label.category = "Category"
label.search = "Search receipts..."
empty.receipts = "No receipts yet. Tap + to add your first receipt."
success.saved = "Receipt saved"
success.exported = "Exported to [QuickBooks/Xero]"
error.capture = "Could not read receipt clearly"
error.network = "No internet connection"
