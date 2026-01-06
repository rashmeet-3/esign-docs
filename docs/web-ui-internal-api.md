# Web UI Internal API Reference

!!! warning "Internal Use Only"
    These endpoints are for the **tomcat_esign Web UI** (port 8080) internal use only.
    
    **If you're integrating via JavaScript SDK or Android SDK, see the [REST API Reference](rest-api-reference.md) instead.**

---

## Overview

The Web UI provides a step-by-step browser interface for testing eSign functionality. These endpoints are called internally by the Web UI and are **not intended for direct SDK integration**.

**Base URL:** `http://localhost:8080/api/esign`

**Port:** 8080 (tomcat_esign)

---

## When to Use This

| Use Case | Use This? | Instead Use |
|----------|-----------|-------------|
| Browser-based testing | ✅ Yes | - |
| JavaScript SDK integration | ❌ No | [REST API Reference](rest-api-reference.md) |
| Android SDK integration | ❌ No | [REST API Reference](rest-api-reference.md) |
| Direct API integration | ❌ No | [REST API Reference](rest-api-reference.md) |

---

## Authentication Modes

| Mode | Description | Method |
|------|-------------|--------|
| `online-aadhaar-otp` | OTP Authentication | OTP sent to mobile |
| `online-aadhaar-bio` | Biometric (Fingerprint) | Fingerprint scan |
| `online-aadhaar-iris` | Iris Authentication | Iris scan |
| `online-aadhaar-face` | Face Authentication | Face scan |
| `capricorn-ekyc-account` | eKYC Authentication | Pre-verified identity |

---

## Configuration Endpoints

### Set eSign Mode

```
POST /set-mode
Body: {"mode": "2.1" | "3.2"}
Response: {"success": true, "mode": "2.1", "modeName": "OTP-based Signing"}
```

### Get Configuration

```
GET /config
Response: {"success": true, "config": {...}}
```

### Reset Session

```
POST /reset
Response: {"success": true, "message": "Session reset successfully"}
```

---

## Signing Workflow Endpoints

The Web UI uses a 9-step process:

### Step 1: Upload PDF

```
POST /step1/upload
Content-Type: multipart/form-data
Body: file=@document.pdf
Response: {"success": true, "step": 1, "fileName": "...", "documentId": "..."}
```

### Step 2: Set Signer Details

```
POST /step2/signer-details
Body: {
  "signerName": "John Doe",
  "docInfo": "Contract",
  "pageSelection": "all",
  "x1": 350, "y1": 50, "x2": 550, "y2": 120,
  "lockType": "n"
}
Response: {"success": true, "step": 2, "lockMode": "🔓 No Lock"}
```

### Steps 3-8: Signing Process

```
POST /step3/generate-hash
POST /step4/convert-hex
POST /step5/generate-txn
POST /step6/construct-xml
POST /step7/sign-xml
POST /step8/generate-esp-form
```

### Step 9: Embed Signature

```
POST /step9/embed-signature
Body: {"outputFileName": "signed.pdf"}
Response: {"success": true, "step": 9, "signedPdfPath": "..."}
```

---

## Signing Options Reference

| Option | Values | Default | Description |
|--------|--------|---------|-------------|
| `cood` | x1,y1,x2,y2 | 350,50,550,120 | Signature position |
| `pagenum` | 1, all, 1-3, first, last | 1 | Pages to sign |
| `reason` | String | Digital Signature | Signing reason |
| `location` | String | India | Signing location |
| `customtext` | String | - | Custom text in signature |
| `greenticked` | y / n | y | Show green checkmark |
| `dateformat` | Date pattern | dd-MMM-yyyy hh:mm a | Date format |
| `lockpdf` | n / y / cf / cfa / ym | n | PDF lock mode |

---

## PDF Lock Modes

| Value | Mode | Description |
|-------|------|-------------|
| `n` | No Lock | Allow additional signatures (default) |
| `y` | Certified (No Changes) | Fully locked, no modifications |
| `cf` | Certified (Form Filling) | Only form fields editable |
| `cfa` | Certified (Form + Annotations) | Forms and comments allowed |
| `ym` | Signature Lock Dictionary | Lock via PdfSigLockDictionary |

---

## Date Format Patterns

### Pattern Characters

| Symbol | Meaning | Example |
|--------|---------|---------|
| `yyyy` | 4-digit year | 2025 |
| `yy` | 2-digit year | 25 |
| `MMMM` | Full month | January |
| `MMM` | Short month | Jan |
| `MM` | 2-digit month | 01 |
| `dd` | 2-digit day | 05 |
| `HH` | 24-hour (00-23) | 14 |
| `hh` | 12-hour (01-12) | 02 |
| `mm` | Minutes | 30 |
| `ss` | Seconds | 45 |
| `a` | AM/PM | PM |

### Common Formats

| Format | Output |
|--------|--------|
| `dd-MMM-yyyy hh:mm a` | 05-Jan-2025 02:30 PM |
| `dd/MM/yyyy HH:mm:ss` | 05/01/2025 14:30:45 |
| `yyyy-MM-dd HH:mm:ss` | 2025-01-05 14:30:45 |
| `MMMM dd, yyyy` | January 05, 2025 |

---

## Complete Workflow Example

```bash
# Set mode
curl -X POST http://localhost:8080/api/esign/set-mode \
  -H "Content-Type: application/json" \
  -d '{"mode":"2.1"}'

# Upload PDF
curl -X POST http://localhost:8080/api/esign/step1/upload \
  -F "file=@document.pdf"

# Set signer details
curl -X POST http://localhost:8080/api/esign/step2/signer-details \
  -H "Content-Type: application/json" \
  -d '{
    "signerName":"John Doe",
    "docInfo":"Contract",
    "pageSelection":"all",
    "lockType":"n"
  }'

# Execute steps 3-8
curl -X POST http://localhost:8080/api/esign/step3/generate-hash
curl -X POST http://localhost:8080/api/esign/step4/convert-hex
curl -X POST http://localhost:8080/api/esign/step5/generate-txn
curl -X POST http://localhost:8080/api/esign/step6/construct-xml
curl -X POST http://localhost:8080/api/esign/step7/sign-xml
curl -X POST http://localhost:8080/api/esign/step8/generate-esp-form

# After ESP signing, embed signature
curl -X POST http://localhost:8080/api/esign/step9/embed-signature \
  -H "Content-Type: application/json" \
  -d '{"outputFileName":"signed.pdf"}'
```

---

## eSign Mode Differences

| Feature | eSign 2.1 | eSign 3.2 |
|---------|-----------|-----------|
| Auth Modes | OTP, Bio, Iris, Face | eKYC |
| Callback | /esp-response | /3.2/getdocs/ + /3.2/callback/ |
| Document URL | Not required | Required (public URL) |
| Response | Synchronous | Asynchronous |

---

## Quick Tips

1. Set mode first (`/set-mode`)
2. Follow steps in order (1 → 2 → 3 → ... → 9)
3. Use same session for all steps
4. Use `lockType: "y"` for final documents
5. Use `lockType: "n"` for multi-signer workflows

---

**Version:** 1.0 | **For:** Web UI Internal Use Only
