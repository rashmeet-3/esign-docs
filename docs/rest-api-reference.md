# REST API Quick Reference

Quick reference for the eSign REST API used by JavaScript SDK and Android SDK.

**Base URL:** `https://your-ngrok-url.ngrok-free.dev/api/v1/esign`

**Port:** 8081 (esign-api)

---

## Authentication

All requests require authentication headers:

```json
{
  "auth": {
    "command": "esign",
    "token": "YOUR_API_TOKEN",
    "key": "YOUR_API_KEY"
  }
}
```

---

## Endpoints Summary

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/v1/esign/upload-pdf` | Upload and sign a PDF |
| `GET` | `/api/v1/esign/status/{reference}` | Check signing status |
| `GET` | `/api/v1/esign/document/{reference}` | Download signed PDF |
| `GET` | `/api/v1/esign/health` | Health check |

---

## 1. Upload and Sign PDF

### `POST /api/v1/esign/upload-pdf`

Initiates the signing process.

=== "JSON Request"

    ```json
    {
      "auth": {
        "command": "esign",
        "token": "YOUR_TOKEN",
        "key": "YOUR_KEY"
      },
      "parameter": {
        "uploadpdf": {
          "pdf64": "BASE64_ENCODED_PDF",
          "title": "Document Title",
          "mode": "online-aadhaar-otp",
          "txn": "YOUR_TRANSACTION_ID",
          "signername": "Signer Name",
          "callbackurl": "https://your-server.com/callback",
          "option": {
            "cood": "350,50,550,120",
            "pagenum": "1",
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

=== "XML Request"

    ```xml
    <?xml version="1.0" encoding="UTF-8"?>
    <request>
      <auth>
        <command>esign</command>
        <token>YOUR_TOKEN</token>
        <key>YOUR_KEY</key>
      </auth>
      <parameter>
        <uploadpdf>
          <pdf64>BASE64_ENCODED_PDF</pdf64>
          <title>Document Title</title>
          <mode>online-aadhaar-otp</mode>
          <txn>YOUR_TRANSACTION_ID</txn>
          <signername>Signer Name</signername>
          <callbackurl>https://your-server.com/callback</callbackurl>
          <option>
            <cood>350,50,550,120</cood>
            <pagenum>1</pagenum>
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

=== "cURL Example"

    ```bash
    curl -X POST "https://your-ngrok-url.ngrok-free.dev/api/v1/esign/upload-pdf" \
      -H "Content-Type: application/json" \
      -d '{
        "auth": {
          "command": "esign",
          "token": "YOUR_TOKEN",
          "key": "YOUR_KEY"
        },
        "parameter": {
          "uploadpdf": {
            "pdf64": "JVBERi0xLjQK...",
            "title": "Contract",
            "mode": "online-aadhaar-otp",
            "txn": "TXN123456",
            "signername": "John Doe"
          }
        }
      }'
    ```

### Success Response

```json
{
  "success": "OK",
  "command": "esign",
  "responsedata": {
    "response": {
      "txn": "TXN123456",
      "reference": "A1B2C3D4E5F6...",
      "redirecturl": "https://your-ngrok-url.ngrok-free.dev/api/v1/esign/redirect/A1B2C3D4E5F6...",
      "getsigneddocurl": "https://your-ngrok-url.ngrok-free.dev/api/v1/esign/document/A1B2C3D4E5F6..."
    }
  }
}
```

### Error Response

```json
{
  "success": "FAIL",
  "errorcode": "AUTH_001",
  "errormessage": "Invalid authentication token"
}
```

---

## 2. Check Status

### `GET /api/v1/esign/status/{reference}`

=== "cURL"

    ```bash
    curl "https://your-ngrok-url.ngrok-free.dev/api/v1/esign/status/A1B2C3D4E5F6"
    ```

### Response

```json
{
  "reference": "A1B2C3D4E5F6...",
  "status": "COMPLETED",
  "clientTxn": "TXN123456",
  "createdAt": "2025-01-06T10:30:00Z",
  "completedAt": "2025-01-06T10:35:00Z"
}
```

### Status Values

| Status | Description |
|--------|-------------|
| `PENDING` | Waiting for user to complete signing |
| `PROCESSING` | ESP response received, processing |
| `COMPLETED` | Signing completed successfully |
| `FAILED` | Signing failed |

---

## 3. Download Signed PDF

### `GET /api/v1/esign/document/{reference}`

=== "cURL"

    ```bash
    curl -o signed.pdf "https://your-ngrok-url.ngrok-free.dev/api/v1/esign/document/A1B2C3D4E5F6"
    ```

=== "Browser"

    Open URL directly in browser to download.

---

## 4. Health Check

### `GET /api/v1/esign/health`

=== "cURL"

    ```bash
    curl "https://your-ngrok-url.ngrok-free.dev/api/v1/esign/health"
    ```

### Response

```json
{
  "status": "UP"
}
```

---

## Authentication Modes

| Mode | Description | Use Case |
|------|-------------|----------|
| `online-aadhaar-otp` | OTP to registered mobile | Remote signing |
| `online-aadhaar-bio` | Fingerprint scan | In-person with device |
| `online-aadhaar-iris` | Iris scan | High security |
| `online-aadhaar-face` | Face scan | Modern biometric |
| `capricorn-ekyc-account` | Pre-verified eKYC | KYC customers |

---

## Signing Options

| Option | Values | Default | Description |
|--------|--------|---------|-------------|
| `cood` | `x1,y1,x2,y2` | `350,50,550,120` | Signature position |
| `pagenum` | `1`, `all`, `first`, `last`, `1-3`, `1,3,5` | `1` | Pages to sign |
| `reason` | String | `Digital Signature` | Signing reason |
| `location` | String | `India` | Signing location |
| `customtext` | String | - | Custom text |
| `greenticked` | `y` / `n` | `y` | Green checkmark |
| `dateformat` | Pattern | `dd-MMM-yyyy hh:mm a` | Date format |
| `lockpdf` | `n` / `y` / `cf` / `cfa` | `n` | Lock mode |

---

## Page Selection

| Value | Description |
|-------|-------------|
| `1` | First page only |
| `first` | First page |
| `last` | Last page |
| `all` | All pages |
| `1-3` | Pages 1 to 3 |
| `1,3,5` | Pages 1, 3, and 5 |

---

## PDF Lock Modes

| Value | Mode | Description |
|-------|------|-------------|
| `n` | No Lock | Allow more signatures (default) |
| `y` | Fully Locked | No changes allowed |
| `cf` | Form Filling Only | Only form fields editable |
| `cfa` | Forms + Annotations | Forms and comments allowed |

---

## Date Formats

| Format | Output |
|--------|--------|
| `dd-MMM-yyyy hh:mm a` | 06-Jan-2025 02:30 PM |
| `dd/MM/yyyy HH:mm` | 06/01/2025 14:30 |
| `yyyy-MM-dd` | 2025-01-06 |
| `MMMM dd, yyyy` | January 06, 2025 |

---

## Error Codes

| Code | Description |
|------|-------------|
| `AUTH_001` | Invalid token |
| `AUTH_002` | Invalid key |
| `VAL_001` | Missing required field |
| `VAL_002` | Invalid PDF data |
| `VAL_003` | Invalid mode |
| `TXN_DUPLICATE` | Transaction ID already used |
| `ESP_001` | User cancelled |
| `ESP_002` | OTP failed |
| `ESP_003` | Aadhaar auth failed |
| `PROC_001` | PDF data required |
| `PROC_002` | Unsupported mode |

---

## Complete Flow

```mermaid
sequenceDiagram
    participant App as Your App
    participant API as eSign API
    participant ESP as ESP Server
    participant User as User

    App->>API: POST /upload-pdf
    API-->>App: redirectUrl, reference
    App->>User: Open redirectUrl
    User->>ESP: Complete OTP/Auth
    ESP->>API: Callback with signature
    API-->>API: Embed signature in PDF
    App->>API: GET /status/{reference}
    API-->>App: status: COMPLETED
    App->>API: GET /document/{reference}
    API-->>App: Signed PDF
```

---

## SDK Integration

### JavaScript SDK

```javascript
const client = new ESignClient({
  baseUrl: 'https://your-ngrok-url.ngrok-free.dev',
  token: 'YOUR_TOKEN',
  key: 'YOUR_KEY'
});

const result = await client.signDocument({
  pdfBase64: base64Pdf,
  signerName: 'John Doe',
  mode: ESignMode.AADHAAR_OTP
});

window.location.href = result.redirectUrl;
```

### Android SDK

```kotlin
val client = ESignClient.Builder()
    .baseUrl("https://your-ngrok-url.ngrok-free.dev")
    .token("YOUR_TOKEN")
    .key("YOUR_KEY")
    .build()

val request = ESignRequest.Builder()
    .pdfBase64(base64Pdf)
    .signerName("John Doe")
    .mode(ESignMode.AADHAAR_OTP)
    .build()

client.signDocument(request) { response ->
    // Open redirectUrl in WebView or browser
}
```

---

## Quick Test

```bash
# 1. Health check
curl https://your-ngrok-url.ngrok-free.dev/api/v1/esign/health

# 2. Upload PDF (replace with your values)
curl -X POST https://your-ngrok-url.ngrok-free.dev/api/v1/esign/upload-pdf \
  -H "Content-Type: application/json" \
  -d @request.json

# 3. Check status
curl https://your-ngrok-url.ngrok-free.dev/api/v1/esign/status/YOUR_REFERENCE

# 4. Download signed PDF
curl -o signed.pdf https://your-ngrok-url.ngrok-free.dev/api/v1/esign/document/YOUR_REFERENCE
```

---

**Version:** 1.0 | **For:** JavaScript SDK, Android SDK, Direct API Integration
