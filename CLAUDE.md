# CLAUDE.md - AI Assistant Guide for VSA App

## Project Overview

**Amazon VSA Audit System** - A single-page web application for Vehicle Safety Audits. The app allows users to upload vehicle data, perform audits, generate QR codes for passing vehicles, and export audit reports.

**Repository:** `/home/user/vsa-app`
**Type:** Standalone HTML SPA (no build system)
**Active Version:** `vsa_app_firebase.html` (Firebase-backed)

## Quick Reference

| Aspect | Details |
|--------|---------|
| Language | Vanilla JavaScript, HTML5, CSS3 |
| Backend | Firebase Realtime Database + Auth |
| Dependencies | CDN-loaded (XLSX, QRCode.js, Firebase SDK) |
| Build System | None (direct HTML editing) |
| Testing | Manual browser testing |
| Deployment | Static file hosting |

## File Structure

```
vsa-app/
├── vsa_app_firebase.html   # PRIMARY - Firebase version (~3,100 lines)
├── vsa_app.html            # Legacy Google Drive version (~2,065 lines)
├── CLAUDE.md               # This file
└── .git/                   # Version control
```

**Important:** The Firebase version (`vsa_app_firebase.html`) is the actively maintained file. The Google Drive version is legacy.

## Architecture

### Single-File Application

All code (HTML, CSS, JavaScript) is contained in monolithic HTML files. No separate source directories, modules, or build artifacts.

### Page Structure

The app uses CSS class toggling for navigation between pages:
- `login-page` - Email/password authentication
- `upload-page` - Vehicle data file uploads
- `audit-page` - License plate search and audit workflow
- `download-page` - Reports and data export

### Global State

```javascript
let vehicleDatabase = [];    // Array of vehicle objects
let currentVehicle = null;   // Selected vehicle for audit
let auditResult = null;      // Current audit result
let currentUser = null;      // Authenticated Firebase user
let app, database, auth;     // Firebase SDK instances
```

## Key Code Patterns

### Page Navigation

```javascript
const pages = ['login-page', 'upload-page', 'audit-page', 'download-page'];
function showPage(pageId) {
    pages.forEach(p => document.getElementById(p).classList.remove('active'));
    document.getElementById(pageId).classList.add('active');
}
```

### Async Firebase Operations

```javascript
async function saveVehicleDataToFirebase() {
    const dbRef = database.ref('vehicles');
    await dbRef.set(vehicleDatabase);
}

async function loadVehicleDataFromFirebase() {
    const snapshot = await database.ref('vehicles').once('value');
    vehicleDatabase = snapshot.val() || [];
}
```

### File Processing

```javascript
function readCSV(file) {
    return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = (e) => { /* parse CSV */ };
        reader.readAsText(file);
    });
}

function readExcel(file, skipRows) {
    return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.onload = (e) => {
            const workbook = XLSX.read(data, { type: 'array' });
            // Extract sheet data
        };
        reader.readAsArrayBuffer(file);
    });
}
```

### Status Messages

```javascript
function showStatus(elementId, message, type) {
    // type: 'success' | 'error' | 'info'
    const el = document.getElementById(elementId);
    el.textContent = message;
    el.className = `status-message ${type}`;
}
```

## Data Structures

### Vehicle Object

```javascript
{
    licensePlate: "ABC123",
    vehicleName: "Vehicle Name",
    diveDeepStatus: "Pass/Fail",      // From dive-deep CSV
    vinAuditStatus: "Pass/Fail",      // From VIN audit Excel
    groundedStatus: "Yes/No",         // From grounded list Excel
    info: { /* additional fields */ }
}
```

### Audit Object

```javascript
{
    licensePlate: "ABC123",
    vehicle: "Vehicle Name",
    timestamp: "2026-02-03T12:00:00.000Z",
    result: "VAN_OK" | "VSA_BLOCK",
    problem: "Description if blocked",
    auditor: "user@example.com",
    qrCode: { /* QR metadata */ }
}
```

## Core Functions Reference

### Authentication
- `handleLogin(e)` - Firebase email/password auth
- `checkLoginStatus()` - Verify session state
- `logout()` - Sign out and clear state

### Data Operations
- `loadVehicleDataFromFirebase()` - Fetch vehicle database
- `saveVehicleDataToFirebase()` - Persist vehicle database
- `saveAuditToFirebase(audit)` - Store audit record
- `buildVehicleDatabase(diveDeep, vinAudit, grounded)` - Merge uploaded data

### Audit Workflow
- `searchVehicle()` - License plate lookup with autocomplete
- `displayVehicleInfo(vehicle)` - Show vehicle details
- `vanOK()` - Record passing audit
- `vsaBlock()` - Record failing audit
- `submitProblem()` - Add problem description
- `showQROverlay()` - Display generated QR code

### Reporting
- `downloadAllAudits(type)` - Export to Excel ('all'|'passed'|'failed')
- `viewVehicleData(type)` - Modal with vehicle table
- `viewAuditData(type)` - Modal with audit table
- `exportModalData()` - Export current modal to CSV

### UI Utilities
- `showPage(pageId)` - Navigate between pages
- `showStatus(elementId, message, type)` - Display feedback
- `showLoading(text)` - Show spinner overlay
- `closeModal()` - Close data modal
- `toggleSection(sectionId)` - Collapse/expand sections

## Development Workflow

### Making Changes

1. Edit `vsa_app_firebase.html` directly
2. Open in browser to test
3. Commit with descriptive message
4. Push to feature branch

### No Build Process

There is no npm, webpack, or any build tooling. Changes are made directly to HTML files.

### Local Testing

```bash
# Option 1: Open file directly
open vsa_app_firebase.html

# Option 2: Simple HTTP server
python -m http.server 8000
# Then visit http://localhost:8000/vsa_app_firebase.html
```

## Code Conventions

### Naming
- **Functions:** camelCase (`handleLogin`, `saveVehicleDataToFirebase`)
- **Constants:** UPPER_SNAKE_CASE (`CONFIG`, `firebaseConfig`)
- **DOM IDs:** kebab-case (`login-page`, `audit-status`)
- **CSS Classes:** kebab-case (`status-message`, `modal-content`)

### Async/Await
All Firebase and file operations use async/await pattern with try/catch for error handling.

### Event Handlers
Inline event handlers in HTML (`onclick="functionName()"`), not addEventListener.

### Styling
- All CSS is inline in `<style>` tags
- Amazon brand colors: `#FF9900` (orange), `#131A22` (dark background)
- Font: Amazon Ember (with system font fallback)
- Mobile-first responsive design with `@media (max-width: 768px)`

## Firebase Configuration

Located at approximately line 1627 in `vsa_app_firebase.html`:

```javascript
const firebaseConfig = {
    apiKey: "AIzaSyBpWSHW3fnPUvADjVxKNh5qpJ2lXkJNZRo",
    authDomain: "vsa-app-13a1f.firebaseapp.com",
    databaseURL: "https://vsa-app-13a1f-default-rtdb.europe-west1.firebasedatabase.app",
    projectId: "vsa-app-13a1f",
    // ...
};
```

**Note:** Firebase security rules must be configured in the Firebase Console, not in code.

## External Dependencies (CDN)

```html
<!-- XLSX for Excel/CSV processing -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>

<!-- QRCode.js for QR generation -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>

<!-- Firebase SDK -->
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-database-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-auth-compat.js"></script>
```

## Common Tasks

### Adding a New Page

1. Add HTML section with unique ID in the pages container
2. Add ID to the `pages` array
3. Add navigation trigger calling `showPage('new-page-id')`
4. Add corresponding CSS styles

### Adding a New Firebase Operation

```javascript
async function newOperation() {
    showLoading('Processing...');
    try {
        const ref = database.ref('path/to/data');
        await ref.set(data);
        showStatus('status-id', 'Success!', 'success');
    } catch (error) {
        console.error('Error:', error);
        showStatus('status-id', 'Error: ' + error.message, 'error');
    } finally {
        hideLoading();
    }
}
```

### Adding a New Export Function

1. Filter/prepare data array
2. Create XLSX workbook using `XLSX.utils.json_to_sheet()`
3. Trigger download with `XLSX.writeFile()`

## Important Notes for AI Assistants

1. **Single-file architecture** - All changes go in one HTML file. No imports/exports.

2. **No testing framework** - Test manually in browser after changes.

3. **CDN dependencies** - Don't attempt to npm install; libraries load from CDN.

4. **Firebase credentials** - Already configured; don't modify unless necessary.

5. **Legacy file** - Avoid modifying `vsa_app.html` (Google Drive version) unless specifically requested.

6. **Inline everything** - CSS goes in `<style>`, JS in `<script>`, all within the HTML file.

7. **Mobile responsiveness** - Test changes at both desktop and mobile widths.

8. **Error handling** - Always wrap async operations in try/catch with user feedback via `showStatus()`.

## Git Workflow

```bash
# Feature branches follow this pattern
git checkout -b claude/feature-name-sessionid

# Commit messages are descriptive
git commit -m "Add audit progress section with styling and logic"

# Push to feature branch
git push -u origin claude/feature-name-sessionid
```

## Troubleshooting

### Firebase Connection Issues
- Check `updateFirebaseStatus()` calls
- Verify Firebase configuration matches console settings
- Check browser console for auth/database errors

### File Upload Problems
- Verify file type matches expected format (CSV/Excel)
- Check `readCSV()` and `readExcel()` error handling
- Ensure XLSX library is loaded from CDN

### UI Not Updating
- Check `showPage()` is called correctly
- Verify element IDs match between HTML and JS
- Look for CSS class toggle issues
