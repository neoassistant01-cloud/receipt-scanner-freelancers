# USER_FLOWS.md — User Flows + UX States

---

## FLOW-001: First-Time Setup

**Entry Conditions:**
- App installed for first time
- No existing user account

**Steps:**
1. App displays welcome screen with value proposition
2. User taps "Get Started"
3. User enters email, creates password (or continues with Apple/Google SSO)
4. System creates account, stores credentials securely
5. App shows onboarding tutorial (3 screens)
6. Tutorial completes, lands on Home screen

**Failure Branches:**
- Network failure during signup → Show retry button with offline message
- Email already exists → Show login prompt
- SSO failure → Fallback to email/password

**UI States:**
- Loading: Spinner while creating account
- Empty: Tutorial slides with swipe indicators
- Error: Red toast with retry option
- Success: Transition to Home

---

## FLOW-002: Capture New Receipt

**Entry Conditions:**
- User on Home screen
- Has camera permission

**Steps:**
1. User taps "+" FAB button
2. Camera view opens with receipt detection overlay
3. System detects receipt edges, shows targeting frame
4. User taps capture (or auto-capture when stable)
5. Image captured, shows preview with "Use Photo" / "Retake" options
6. User confirms photo
7. System shows processing indicator while AI extracts data
8. Extraction complete, shows extracted data card with:
   - Vendor name (editable)
   - Date (editable)
   - Total amount (editable)
   - Line items (editable)
   - Suggested category (tappable to change)
9. User taps "Save" to confirm
10. Receipt saved, returns to Home with success toast

**Failure Branches:**
- Camera permission denied → Show permission request screen
- Poor image quality → Show "Could not read receipt clearly, please try again"
- AI extraction fails → Show manual entry fallback with image reference
- No network for AI → Queue for later processing, show "Will process when online"
- Save fails → Show error with retry option

**UI States:**
- Loading: "Processing receipt..." spinner
- Empty: "No receipts yet" with camera icon
- Error: Red toast with specific error message
- Success: Green toast "Receipt saved"

**Instrumentation:**
- Event: receipt_capture_start
- Event: receipt_capture_complete
- Event: receipt_extraction_success / failure
- Event: receipt_save_success

---

## FLOW-003: Import from Gallery

**Entry Conditions:**
- User on Home screen
- Has photo library permission

**Steps:**
1. User taps "+" FAB → selects "Import from Gallery"
2. Photo picker opens
3. User selects image(s)
4. Returns to app with selected images
5. Process continues as FLOW-002 from step 7

**Failure Branches:**
- Permission denied → Show settings redirect
- No images selected → Return to previous state
- Invalid image format → Show error, suggest different format

---

## FLOW-004: Review and Edit Receipt

**Entry Conditions:**
- User on Home screen
- At least one receipt exists

**Steps:**
1. User taps on receipt card in list
2. Receipt detail screen shows:
   - Full receipt image (zoomable)
   - All extracted fields (editable)
   - Category with change option
   - Export status
   - Delete option
3. User edits any field
4. Changes auto-save with "Saved" indicator

**Failure Branches:**
- Save conflict (sync) → Show "Latest version loaded, your changes saved locally"
- Delete without confirmation → Confirmation dialog first

---

## FLOW-005: QuickBooks Export

**Entry Conditions:**
- Receipt selected for export
- User has QuickBooks connected
- No existing export of this receipt

**Steps:**
1. User taps "Export" on receipt detail
2. Export options: QuickBooks / Xero (if connected)
3. User selects QuickBooks
4. System calls QuickBooks API to create expense
5. Loading: "Exporting to QuickBooks..."
6. Success: Receipt marked as "Exported to QuickBooks"
7. Error: Show specific error (auth expired, rate limit, etc.)

**Failure Branches:**
- QuickBooks not connected → Prompt to connect (OAuth flow)
- OAuth token expired → Auto-refresh or prompt re-auth
- Rate limited → Queue retry, show "Will retry shortly"
- API error → Show error message with support contact

---

## FLOW-006: Xero Export

**Entry Conditions:**
- Receipt selected for export
- User has Xero connected
- No existing export of this receipt

**Steps:** Same as FLOW-005 but with Xero API

**Failure Branches:** Same as FLOW-005

---

## FLOW-007: Connect Accounting Software

**Entry Conditions:**
- User on Settings > Integrations screen
- No accounting software connected

**Steps:**
1. User taps "Connect QuickBooks" or "Connect Xero"
2. OAuth2 redirect to QuickBooks/Xero login
3. User authorizes (may need company selection)
4. Callback returns with auth code
5. System exchanges code for tokens
6. Connection saved, returns to settings

**Failure Branches:**
- User cancels OAuth → Return to settings, show "Not connected"
- OAuth fails → Show error with retry option
- Token exchange fails → Show error, suggest retry

---

## FLOW-008: Search Receipts

**Entry Conditions:**
- User on Home screen

**Steps:**
1. User taps search icon in header
2. Search bar expands
3. User types query (vendor name)
4. Results filter in real-time
5. User can add filters: date range, category, amount range
6. User taps result to view detail

**Failure Branches:**
- No results → Show "No receipts match your search"
- Search fails → Show error, allow retry

---

## FLOW-009: View Monthly Summary

**Entry Conditions:**
- User on Home screen

**Steps:**
1. User taps "Reports" in navigation
2. Current month summary displayed
3. Bar chart shows spending by category
4. User can tap category to see list of receipts
5. User can select previous months
6. User can export report as PDF

**Failure Branches:**
- No data for month → Show empty state "No receipts this month"

---

## FLOW-010: Backup and Restore

**Entry Conditions:**
- User on Settings > Backup screen

**Steps:**
1. User taps "Backup Now"
2. System compresses and encrypts data
3. Shows destination options: iCloud / Google Drive
4. User selects, authenticates if needed
5. Upload begins, progress shown
6. Success: "Backup complete"

**Restore:**
1. User taps "Restore"
2. Confirms overwrite warning
3. Downloads and decrypts backup
4. Merges with existing data
5. Success: "Restored X receipts"

**Failure Branches:**
- Backup fails → Show error with retry
- Restore fails → Show error, keep existing data
- Storage full → Show storage management options
