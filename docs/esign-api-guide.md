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
14. [Complete Examples](#14-complete-examples)

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
| `online-aadhaar-ekyc` | eKYC-based authentication | eSign 3.2 | Pre-verified eKYC identity |

### Mode Selection Guide

| Use Case | Recommended Mode |
|----------|------------------|
| Remote signing (user not physically present) | `online-aadhaar-otp` |
| In-person signing with fingerprint device | `online-aadhaar-bio` |
| High-security environments | `online-aadhaar-iris` |
| Modern biometric systems | `online-aadhaar-face` |
| Pre-verified KYC customers | `online-aadhaar-ekyc` |

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

**eKYC Mode (Requires ekycid):**
```xml
<mode>online-aadhaar-ekyc</mode>
<ekycid>EKYC123456789</ekycid>
```

### Mode Requirements

| Mode | Requires ekycid | Requires Device | User Interaction |
|------|-----------------|-----------------|------------------|
| `online-aadhaar-otp` | No | No | Enter OTP from mobile |
| `online-aadhaar-bio` | No | Fingerprint scanner | Place finger on scanner |
| `online-aadhaar-iris` | No | Iris scanner | Look into scanner |
| `online-aadhaar-face` | No | Camera | Face the camera |
| `online-aadhaar-ekyc` | **Yes** | No | Minimal (pre-verified) |

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

| Code | Description |
|------|-------------|
| `AUTH_001` | Invalid authentication token |
| `VAL_001` | Missing required field |
| `VAL_003` | Invalid mode specified |
| `ESP_001` | User cancelled signing |
| `ESP_002` | OTP verification failed |
| `ESP_003` | Aadhaar authentication failed |

---

## 14. Complete Examples

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

### Example 5: eKYC Mode

```json
{
    "parameter": {
        "uploadpdf": {
            "pdf64": "...",
            "title": "KYC Document",
            "mode": "online-aadhaar-ekyc",
            "ekycid": "EKYC123456789",
            "signername": "Amit Kumar"
        }
    }
}
```

---

## Support

**Version:** 1.0  
**Last Updated:** January 2025
