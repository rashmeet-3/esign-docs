# eSign API Integration Guide

## Overview

The eSign API provides a REST interface for digitally signing PDF documents using Aadhaar-based authentication. The API supports both **eSign 2.1** and **eSign 3.2** signing modes with multiple authentication methods.

**Base URL:** `https://your-domain.com/api/v1/esign`

**Supported Formats:** XML (default) and JSON

**Version:** 1.0

---

## Table of Contents

1. [Authentication](#1-authentication)
2. [Sign Document Endpoint](#2-sign-document-endpoint)
3. [Request Parameters](#3-request-parameters)
4. [Authentication Modes](#4-authentication-modes)
5. [Signing Options](#5-signing-options)
6. [Date Format Reference](#6-date-format-reference)
7. [PDF Lock Modes](#7-pdf-lock-modes)
8. [Multi-Page Signing](#8-multi-page-signing)
9. [Response Format](#9-response-format)
10. [Retrieve Signed Document](#10-retrieve-signed-document)
11. [Check Transaction Status](#11-check-transaction-status)
12. [Webhook Callbacks](#12-webhook-callbacks)
13. [Error Codes](#13-error-codes)
14. [Transaction ID Uniqueness](#14-transaction-id-uniqueness)
15. [Complete Examples](#15-complete-examples)

---

## 1. Authentication

All requests must include authentication credentials in the `auth` block.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `command` | String | Yes | Must be `"esign"` |
| `token` | String | Yes | Your API authentication token |
| `key` | String | Yes | Your API key |

---

## 2. Sign Document Endpoint

### POST `/api/v1/esign`

Initiates a document signing request.

**Headers:**

| Header | Value | Description |
|--------|-------|-------------|
| `Content-Type` | `application/xml` or `application/json` | Request format |
| `Accept` | `application/xml` or `application/json` | Response format |

---

## 3. Request Parameters

### XML Format

```xml
<?xml version="1.0" encoding="UTF-8"?>
<request>
    <auth>
        <command>esign</command>
        <token>your-api-token</token>
        <key>your-api-key</key>
    </auth>
    <parameter>
        <uploadpdf>
            <pdf64>BASE64_ENCODED_PDF_CONTENT</pdf64>
            <title>Document Title</title>
            <mode>online-aadhaar-otp</mode>
            <txn>CLIENT_TRANSACTION_ID</txn>
            <signername>Signer Name</signername>
            <callbackurl>https://your-callback-url.com/webhook</callbackurl>
            <option>
                <cood>350,50,550,120</cood>
                <pagenum>all</pagenum>
                <reason>Digital Signature</reason>
                <location>India</location>
                <greenticked>y</greenticked>
                <dateformat>dd-MMM-yyyy hh:mm a</dateformat>
                <lockpdf>n</lockpdf>
            </option>
        </uploadpdf>
    </parameter>
</request>
```

### JSON Format

```json
{
    "auth": {
        "command": "esign",
        "token": "your-api-token",
        "key": "your-api-key"
    },
    "parameter": {
        "uploadpdf": {
            "pdf64": "BASE64_ENCODED_PDF_CONTENT",
            "title": "Document Title",
            "mode": "online-aadhaar-otp",
            "txn": "CLIENT_TRANSACTION_ID",
            "signername": "Signer Name",
            "callbackurl": "https://your-callback-url.com/webhook",
            "option": {
                "cood": "350,50,550,120",
                "pagenum": "all",
                "reason": "Digital Signature",
                "location": "India",
                "greenticked": "y",
                "dateformat": "dd-MMM-yyyy hh:mm a",
                "lockpdf": "n"
            }
        }
    }
}
```

### Parameter Reference

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `pdf64` | String | Yes | Base64 encoded PDF document |
| `pdfurl` | String | No | Alternative: URL to fetch PDF from (instead of pdf64) |
| `title` | String | Yes | Document title (max 50 characters) |
| `mode` | String | Yes | Authentication mode (see [Modes](#4-authentication-modes)) |
| `txn` | String | No | Your transaction ID for tracking |
| `signername` | String | Yes | Name of the signer |
| `ekycid` | String | Conditional | eKYC ID (required for eSign 3.2 modes) |
| `callbackurl` | String | No | Webhook URL for completion notification |
| `option` | Object | No | Signing options (see [Options](#5-signing-options)) |

---

## 4. Authentication Modes

The `mode` parameter specifies the Aadhaar authentication method.

### Available Modes

| Mode | Description | Version | Authentication Method |
|------|-------------|---------|----------------------|
| `online-aadhaar-otp` | OTP-based authentication | eSign 2.1 | User receives OTP on registered mobile |
| `online-aadhaar-bio` | Biometric (Fingerprint) authentication | eSign 2.1 | User provides fingerprint scan |
| `online-aadhaar-iris` | Iris scan authentication | eSign 2.1 | User provides iris scan |
| `online-aadhaar-face` | Face authentication | eSign 2.1 | User provides face scan |
| `capricorn-ekyc-account` | eKYC-based authentication | eSign 3.2 | Pre-verified Capricorn eKYC account |

### Mode Selection Guide

| Use Case | Recommended Mode |
|----------|------------------|
| Remote signing (user not physically present) | `online-aadhaar-otp` |
| In-person signing with fingerprint device | `online-aadhaar-bio` |
| High-security environments | `online-aadhaar-iris` |
| Modern biometric systems | `online-aadhaar-face` |
| Pre-verified KYC customers | `capricorn-ekyc-account` |

### Examples

**OTP Mode (Most Common):**
```xml
<mode>online-aadhaar-otp</mode>
```

**Biometric/Fingerprint Mode:**
```xml
<mode>online-aadhaar-bio</mode>
```

**Iris Scan Mode:**
```xml
<mode>online-aadhaar-iris</mode>
```

**Face Authentication Mode:**
```xml
<mode>online-aadhaar-face</mode>
```

**Capricorn eKYC Account Mode (Requires ekycid):**
```xml
<mode>capricorn-ekyc-account</mode>
<ekycid>EKYC123456789</ekycid>
```

### Mode Requirements

| Mode | Requires ekycid | Requires Device | User Interaction |
|------|-----------------|-----------------|------------------|
| `online-aadhaar-otp` | No | No | Enter OTP from mobile |
| `online-aadhaar-bio` | No | Fingerprint scanner | Place finger on scanner |
| `online-aadhaar-iris` | No | Iris scanner | Look into scanner |
| `online-aadhaar-face` | No | Camera | Face the camera |
| `capricorn-ekyc-account` | **Yes** | No | Minimal (pre-verified) |

---

## 5. Signing Options

The `option` block controls signature appearance and placement.

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `cood` | String | `350,50,550,120` | Signature box coordinates: `x1,y1,x2,y2` |
| `pagenum` | String | `1` | Page selection for signature |
| `reason` | String | `Digital Signature` | Reason for signing |
| `location` | String | `India` | Signing location |
| `customtext` | String | - | Custom text to display in signature |
| `greenticked` | String | `y` | Show green checkmark icon: `y` or `n` |
| `dateformat` | String | `dd-MMM-yyyy hh:mm a` | Date format in signature |
| `lockpdf` | String | `n` | PDF lock mode (see [Lock Modes](#7-pdf-lock-modes)) |

> **Note:** All signatures use **PKCS7_COMPLETE** format with **Timestamp (TSA)** and **LTV (Long Term Validation)** enabled by default. This ensures signatures include full certificate chain, cryptographic proof of signing time from a trusted Timestamp Authority, and remain valid for long-term verification even after certificate expiration.

### Coordinate System

Coordinates define the signature box position on the PDF page:

```
(x1, y1) ─────────────────┐
    │                     │
    │   Signature Box     │
    │                     │
    └─────────────────────┘ (x2, y2)
```

- **Origin (0,0):** Bottom-left corner of the page
- **Units:** Points (1 point = 1/72 inch)
- **Standard A4 page:** 595 x 842 points

**Common Placements:**

| Position | Coordinates |
|----------|-------------|
| Bottom-right | `400,50,550,120` |
| Bottom-left | `50,50,200,120` |
| Top-right | `400,750,550,820` |
| Center-bottom | `200,50,400,120` |

---

## 6. Date Format Reference

The `dateformat` parameter controls how the signing date appears in the signature box.

### Format Pattern Characters

| Symbol | Meaning | Example |
|--------|---------|---------|
| `yyyy` | 4-digit year | `2025` |
| `yy` | 2-digit year | `25` |
| `MMMM` | Full month name | `January` |
| `MMM` | Abbreviated month | `Jan` |
| `MM` | 2-digit month | `01` |
| `M` | Month without leading zero | `1` |
| `dd` | 2-digit day | `05` |
| `d` | Day without leading zero | `5` |
| `EEEE` | Full day name | `Monday` |
| `EEE` | Abbreviated day | `Mon` |
| `HH` | 24-hour hour (00-23) | `14` |
| `hh` | 12-hour hour (01-12) | `02` |
| `h` | 12-hour without leading zero | `2` |
| `mm` | Minutes | `30` |
| `ss` | Seconds | `45` |
| `a` | AM/PM marker | `PM` |
| `z` | Timezone | `IST` |
| `Z` | Timezone offset | `+0530` |

### Common Date Formats

| Format Pattern | Example Output |
|----------------|----------------|
| `dd-MMM-yyyy hh:mm a` | `05-Jan-2025 02:30 PM` |
| `dd/MM/yyyy HH:mm:ss` | `05/01/2025 14:30:45` |
| `yyyy-MM-dd HH:mm:ss` | `2025-01-05 14:30:45` |
| `dd-MM-yyyy` | `05-01-2025` |
| `MMMM dd, yyyy` | `January 05, 2025` |
| `dd MMMM yyyy` | `05 January 2025` |
| `EEE, dd MMM yyyy` | `Mon, 05 Jan 2025` |
| `EEEE, MMMM dd, yyyy` | `Monday, January 05, 2025` |
| `yyyy/MM/dd` | `2025/01/05` |
| `dd.MM.yyyy` | `05.01.2025` |

### Date + Time Formats

| Format Pattern | Example Output |
|----------------|----------------|
| `dd-MMM-yyyy hh:mm a` | `05-Jan-2025 02:30 PM` |
| `dd-MMM-yyyy HH:mm` | `05-Jan-2025 14:30` |
| `dd/MM/yyyy hh:mm:ss a` | `05/01/2025 02:30:45 PM` |
| `yyyy-MM-dd'T'HH:mm:ss` | `2025-01-05T14:30:45` |
| `dd MMM yyyy, hh:mm a` | `05 Jan 2025, 02:30 PM` |
| `MMMM dd, yyyy 'at' hh:mm a` | `January 05, 2025 at 02:30 PM` |
| `EEE dd-MMM-yyyy hh:mm a` | `Mon 05-Jan-2025 02:30 PM` |

### Regional Formats

| Region | Format Pattern | Example Output |
|--------|----------------|----------------|
| India | `dd-MMM-yyyy hh:mm a` | `05-Jan-2025 02:30 PM` |
| US | `MM/dd/yyyy hh:mm a` | `01/05/2025 02:30 PM` |
| UK | `dd/MM/yyyy HH:mm` | `05/01/2025 14:30` |
| ISO 8601 | `yyyy-MM-dd'T'HH:mm:ssZ` | `2025-01-05T14:30:45+0530` |
| Europe | `dd.MM.yyyy HH:mm` | `05.01.2025 14:30` |
| Japan | `yyyy/MM/dd HH:mm` | `2025/01/05 14:30` |

### Date Only Formats

| Format Pattern | Example Output |
|----------------|----------------|
| `dd-MMM-yyyy` | `05-Jan-2025` |
| `dd/MM/yyyy` | `05/01/2025` |
| `MM/dd/yyyy` | `01/05/2025` |
| `yyyy-MM-dd` | `2025-01-05` |
| `dd MMMM yyyy` | `05 January 2025` |
| `MMMM dd, yyyy` | `January 05, 2025` |
| `d MMM yyyy` | `5 Jan 2025` |

### Time Only Formats

| Format Pattern | Example Output |
|----------------|----------------|
| `hh:mm a` | `02:30 PM` |
| `HH:mm:ss` | `14:30:45` |
| `hh:mm:ss a` | `02:30:45 PM` |
| `HH:mm` | `14:30` |
| `h:mm a` | `2:30 PM` |

### Special Characters in Format

Use single quotes to include literal text:

| Format Pattern | Example Output |
|----------------|----------------|
| `'Signed on' dd-MMM-yyyy` | `Signed on 05-Jan-2025` |
| `dd-MMM-yyyy 'at' hh:mm a` | `05-Jan-2025 at 02:30 PM` |
| `'Date:' dd/MM/yyyy` | `Date: 05/01/2025` |

### Examples in Request

**Default Format:**
```xml
<option>
    <dateformat>dd-MMM-yyyy hh:mm a</dateformat>
</option>
```
Output: `05-Jan-2025 02:30 PM`

**ISO Format:**
```xml
<option>
    <dateformat>yyyy-MM-dd HH:mm:ss</dateformat>
</option>
```
Output: `2025-01-05 14:30:45`

**Formal Format:**
```xml
<option>
    <dateformat>EEEE, MMMM dd, yyyy 'at' hh:mm a</dateformat>
</option>
```
Output: `Monday, January 05, 2025 at 02:30 PM`

**Date Only:**
```xml
<option>
    <dateformat>dd MMMM yyyy</dateformat>
</option>
```
Output: `05 January 2025`

---

## 7. PDF Lock Modes

The `lockpdf` parameter controls document protection after signing.

| Value | Mode | Description |
|-------|------|-------------|
| `n` | **No Lock** | Allow additional signatures and modifications (default) |
| `y` | **Certified (No Changes)** | Most restrictive - no changes allowed after signing |
| `cf` | **Certified (Form Filling)** | Allow only form field filling |
| `cfa` | **Certified (Form Filling & Annotations)** | Allow form filling and adding annotations |
| `ym` | **Signature Lock Dictionary** | Lock document using PdfSigLockDictionary |

### Choosing the Right Lock Mode

| Use Case | Recommended Mode |
|----------|------------------|
| Single signer, final document | `y` (most secure) |
| Multiple signers needed | `n` (no lock) |
| Fillable form that users complete after signing | `cf` |
| Document needing review comments | `cfa` |
| Legacy system compatibility | `ym` |

---

## 8. Multi-Page Signing

The `pagenum` option supports flexible page selection.

| Value | Description | Example Result (5-page doc) |
|-------|-------------|----------------------------|
| `1` | Single page | Page 1 only |
| `first` | First page | Page 1 |
| `last` | Last page | Page 5 |
| `all` | All pages | Pages 1, 2, 3, 4, 5 |
| `1-3` | Page range | Pages 1, 2, 3 |
| `1,3,5` | Specific pages | Pages 1, 3, 5 |

---

## 8.1 Multi-Location Signing

Multi-location signing allows you to place signatures at **different positions on different pages**. This is useful when you need varying signature placements (e.g., bottom-right on page 1, center on page 3, etc.).

### When to Use Multi-Location

| Scenario | Use |
|----------|-----|
| Same position on all pages | Standard `cood` + `pagenum` |
| Different positions on different pages | `signaturePositions` array |
| Signature on specific pages with specific coordinates | `signaturePositions` array |

### Request Format

To use multi-location signing, set `cood` and `pagenum` to `"custom"` and provide a `signaturePositions` array:

#### JSON Format

```json
{
    "auth": {
        "command": "esign",
        "token": "your-api-token",
        "key": "your-api-key"
    },
    "parameter": {
        "uploadpdf": {
            "pdf64": "BASE64_ENCODED_PDF_CONTENT",
            "title": "Multi-Location Contract",
            "mode": "online-aadhaar-otp",
            "txn": "CLIENT_TRANSACTION_ID",
            "signername": "Signer Name",
            "option": {
                "cood": "custom",
                "pagenum": "custom",
                "reason": "Digital Signature",
                "location": "India",
                "greenticked": "y",
                "lockpdf": "n",
                "signaturePositions": [
                    {
                        "pages": "first",
                        "cood": "50,50,200,120"
                    },
                    {
                        "pages": "last",
                        "cood": "400,50,550,120"
                    },
                    {
                        "pages": "2,3",
                        "cood": "200,400,400,470"
                    }
                ]
            }
        }
    }
}
```

#### XML Format

```xml
<?xml version="1.0" encoding="UTF-8"?>
<request>
    <auth>
        <command>esign</command>
        <token>your-api-token</token>
        <key>your-api-key</key>
    </auth>
    <parameter>
        <uploadpdf>
            <pdf64>BASE64_ENCODED_PDF_CONTENT</pdf64>
            <title>Multi-Location Contract</title>
            <mode>online-aadhaar-otp</mode>
            <txn>CLIENT_TRANSACTION_ID</txn>
            <signername>Signer Name</signername>
            <option>
                <cood>custom</cood>
                <pagenum>custom</pagenum>
                <reason>Digital Signature</reason>
                <location>India</location>
                <greenticked>y</greenticked>
                <lockpdf>n</lockpdf>
                <signaturePositions>
                    <position>
                        <pages>first</pages>
                        <cood>50,50,200,120</cood>
                    </position>
                    <position>
                        <pages>last</pages>
                        <cood>400,50,550,120</cood>
                    </position>
                    <position>
                        <pages>2,3</pages>
                        <cood>200,400,400,470</cood>
                    </position>
                </signaturePositions>
            </option>
        </uploadpdf>
    </parameter>
</request>
```

### signaturePositions Array Reference

Each object in the `signaturePositions` array has:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `pages` | String | Yes | Page selection: `"first"`, `"last"`, `"all"`, `"1"`, `"1-3"`, `"1,3,5"` |
| `cood` | String | Yes | Coordinates for this position: `"x1,y1,x2,y2"` |

### Page Selection Values for signaturePositions

| Value | Description |
|-------|-------------|
| `"first"` | First page only |
| `"last"` | Last page only |
| `"all"` | All pages (same coordinates) |
| `"1"` | Specific page number |
| `"1-3"` | Page range (pages 1, 2, 3) |
| `"1,3,5"` | Specific pages (comma-separated) |

### Important Rules

!!! warning "Required Settings for Multi-Location"
    When using `signaturePositions`, you **must** set:
    
    - `cood` = `"custom"`
    - `pagenum` = `"custom"`
    
    If these are not set to `"custom"`, the `signaturePositions` array will be ignored.

### Multi-Location Examples

**Example 1: Different corners on first and last page**
```json
"signaturePositions": [
    { "pages": "first", "cood": "50,50,200,120" },
    { "pages": "last", "cood": "400,700,550,770" }
]
```

**Example 2: Signature on specific pages with different positions**
```json
"signaturePositions": [
    { "pages": "1", "cood": "350,50,500,120" },
    { "pages": "3", "cood": "50,400,200,470" },
    { "pages": "5", "cood": "350,400,500,470" }
]
```

**Example 3: Same position on a range of pages, different on last**
```json
"signaturePositions": [
    { "pages": "1-4", "cood": "350,50,500,120" },
    { "pages": "last", "cood": "200,50,400,120" }
]
```

**Example 4: Multiple positions including all middle pages**
```json
"signaturePositions": [
    { "pages": "first", "cood": "50,50,200,120" },
    { "pages": "2,3,4", "cood": "200,400,400,470" },
    { "pages": "last", "cood": "400,50,550,120" }
]
```

### Validation Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `VAL_008` | `signaturePositions` provided but `cood` ≠ `"custom"` | Set `cood` to `"custom"` |
| `VAL_009` | `signaturePositions` provided but `pagenum` ≠ `"custom"` | Set `pagenum` to `"custom"` |
| `VAL_010` | Empty `signaturePositions` array | Add at least one position |
| `VAL_011` | Invalid page value in position | Use valid page format |
| `VAL_012` | Invalid coordinates in position | Use `"x1,y1,x2,y2"` format |

---

## 9. Response Format

### Success Response

```json
{
    "success": "OK",
    "command": "esign",
    "requestid": "1702819200123456789",
    "date": "2025-01-05T10:30:00+05:30",
    "responsedata": {
        "response": {
            "txn": "CLIENT_TRANSACTION_ID",
            "reference": "a1b2c3d4e5f6",
            "redirecturl": "https://api.example.com/api/v1/esign/redirect/a1b2c3d4e5f6",
            "getsigneddocurl": "https://api.example.com/api/v1/esign/signed/a1b2c3d4e5f6"
        }
    }
}
```

---

## 10. Retrieve Signed Document

### GET `/api/v1/esign/document/{reference}`

Returns the signed PDF file directly.

---

## 11. Check Transaction Status

### GET `/api/v1/esign/status/{reference}`

| Status | Description |
|--------|-------------|
| `PENDING` | Request created, awaiting user action |
| `PROCESSING` | ESP response received, embedding signature |
| `COMPLETED` | Signing completed successfully |
| `FAILED` | Signing failed |

---

## 12. Webhook Callbacks

If `callbackurl` is provided, a POST request is sent when signing completes or fails.

---

## 13. Error Codes

### Authentication Errors

| Code | Description | Solution |
|------|-------------|----------|
| `AUTH_001` | Invalid authentication token | Check `api.auth.token` in your request matches server config |
| `AUTH_002` | Invalid authentication key | Check `api.auth.key` in your request matches server config |
| `AUTH_003` | Missing authentication | Include `auth` block in request |

### Validation Errors

| Code | Description | Solution |
|------|-------------|----------|
| `VAL_001` | Missing required field | Check all required fields are present (pdf64/pdfurl, title, mode, signername) |
| `VAL_002` | Invalid PDF data | Ensure PDF is valid and properly Base64 encoded |
| `VAL_003` | Invalid mode specified | Use valid mode: `online-aadhaar-otp`, `online-aadhaar-bio`, `online-aadhaar-iris`, `online-aadhaar-face`, or `capricorn-ekyc-account` |
| `VAL_004` | Invalid page selection | Use valid pagenum: `1`, `all`, `first`, `last`, `1-3`, or `1,3,5` |
| `VAL_005` | Invalid coordinates | Ensure cood format is `x1,y1,x2,y2` with valid numbers |
| `VAL_006` | Title too long | Title must be 50 characters or less |
| `VAL_007` | Invalid date format | Check dateformat pattern is valid |

### Transaction Errors

| Code | Description | Solution |
|------|-------------|----------|
| `TXN_DUPLICATE` | Transaction ID already used | Use a unique transaction ID (see below) |
| `TXN_NOT_FOUND` | Transaction not found | Check reference ID is correct |
| `TXN_EXPIRED` | Transaction expired | Start a new signing request |

### ESP (eSign Provider) Errors

| Code | Description | Solution |
|------|-------------|----------|
| `ESP_001` | User cancelled signing | User chose to cancel - no action needed |
| `ESP_002` | OTP verification failed | User entered wrong OTP - retry signing |
| `ESP_003` | Aadhaar authentication failed | Aadhaar verification failed - check Aadhaar details |
| `ESP_004` | ESP server unavailable | ESP server down - retry later |
| `ESP_005` | ESP callback failed | Check callback URL is accessible |
| `ESP_006` | Signature generation failed | Contact ESP provider |

### Processing Errors

| Code | Description | Solution |
|------|-------------|----------|
| `PROC_001` | PDF data required | Provide either `pdf64` or `pdfurl` |
| `PROC_002` | Unsupported authentication mode | Use supported mode for your ESP configuration |
| `PROC_003` | PDF processing failed | Check PDF is not corrupted or password protected |
| `PROC_004` | Signature embedding failed | Contact support |

### License Errors

| Code | Description | Solution |
|------|-------------|----------|
| `LIC_001` | License file not found | Place `eSignLicense` in `config/` folder |
| `LIC_002` | License expired | Contact Capricorn for license renewal |
| `LIC_003` | License invalid | Check license file is correct |

### Certificate Errors

| Code | Description | Solution |
|------|-------------|----------|
| `CERT_001` | Certificate file not found | Place `privatekey.pfx` in `config/` folder |
| `CERT_002` | Certificate password incorrect | Check `esign.certificate.password` in config |
| `CERT_003` | Certificate expired | Contact Capricorn for certificate renewal |

### Error Response Format

**JSON Error Response:**
```json
{
    "success": "FAIL",
    "errorcode": "TXN_DUPLICATE",
    "errormessage": "Transaction ID 'ORDER-12345' has already been used for a completed signing. Each transaction ID can only be used once. Please use a unique transaction ID."
}
```

**XML Error Response:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<response>
    <success>FAIL</success>
    <errorcode>TXN_DUPLICATE</errorcode>
    <errormessage>Transaction ID 'ORDER-12345' has already been used for a completed signing. Each transaction ID can only be used once. Please use a unique transaction ID.</errormessage>
</response>
```

---

## 14. Transaction ID Uniqueness

### Overview

Once a PDF is successfully signed with a transaction ID (`txn`), that ID **cannot be used again**. This prevents duplicate signing and ensures each transaction is unique.

### How It Works

1. You send a request with `txn: "ORDER-12345"`
2. User completes signing successfully
3. Transaction `ORDER-12345` is marked as **COMPLETED**
4. Any future request with `txn: "ORDER-12345"` returns `TXN_DUPLICATE` error

### Best Practices for Transaction IDs

| Practice | Example |
|----------|---------|
| Use unique identifiers | `ORDER-12345`, `INV-2025-001`, `CONTRACT-ABC123` |
| Include timestamp | `DOC-20251215-143052-001` |
| Use UUID | `550e8400-e29b-41d4-a716-446655440000` |
| Prefix by document type | `LOAN-001`, `KYC-002`, `AGREEMENT-003` |

### Handling TXN_DUPLICATE Error

```javascript
// JavaScript example
try {
    const result = await client.signDocument(request);
} catch (error) {
    if (error.code === 'TXN_DUPLICATE') {
        // Generate new transaction ID and retry
        request.txn = generateUniqueId();
        const result = await client.signDocument(request);
    }
}
```

---

## 15. Complete Examples

### Example 1: OTP Mode

```json
{
    "parameter": {
        "uploadpdf": {
            "pdf64": "...",
            "title": "Contract",
            "mode": "online-aadhaar-otp",
            "signername": "Rahul Sharma"
        }
    }
}
```

### Example 2: Biometric Mode

```json
{
    "parameter": {
        "uploadpdf": {
            "pdf64": "...",
            "title": "Loan Agreement",
            "mode": "online-aadhaar-bio",
            "signername": "Priya Patel",
            "option": {
                "lockpdf": "y"
            }
        }
    }
}
```

### Example 3: Iris Mode

```json
{
    "parameter": {
        "uploadpdf": {
            "pdf64": "...",
            "title": "Security Document",
            "mode": "online-aadhaar-iris",
            "signername": "Vikram Singh"
        }
    }
}
```

### Example 4: Face Mode

```json
{
    "parameter": {
        "uploadpdf": {
            "pdf64": "...",
            "title": "Verification",
            "mode": "online-aadhaar-face",
            "signername": "Anita Desai"
        }
    }
}
```

### Example 5: Capricorn eKYC Account Mode

```json
{
    "parameter": {
        "uploadpdf": {
            "pdf64": "...",
            "title": "KYC Document",
            "mode": "capricorn-ekyc-account",
            "ekycid": "EKYC123456789",
            "signername": "Amit Kumar"
        }
    }
}
```

### Example 6: Multi-Location Signing

Sign different pages at different positions:

```json
{
    "auth": {
        "command": "esign",
        "token": "your-api-token",
        "key": "your-api-key"
    },
    "parameter": {
        "uploadpdf": {
            "pdf64": "...",
            "title": "Multi-Page Agreement",
            "mode": "online-aadhaar-otp",
            "txn": "MULTI-LOC-001",
            "signername": "Rahul Sharma",
            "option": {
                "cood": "custom",
                "pagenum": "custom",
                "reason": "Digital Signature",
                "location": "Mumbai, India",
                "greenticked": "y",
                "dateformat": "dd-MMM-yyyy hh:mm a",
                "lockpdf": "n",
                "signaturePositions": [
                    {
                        "pages": "first",
                        "cood": "50,50,200,120"
                    },
                    {
                        "pages": "2,3,4",
                        "cood": "350,400,500,470"
                    },
                    {
                        "pages": "last",
                        "cood": "400,50,550,120"
                    }
                ]
            }
        }
    }
}
```

---

## Support

**Version:** 1.0  
**Last Updated:** December 2025