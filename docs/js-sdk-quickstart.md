# JavaScript SDK Quick Start

Get started with the eSign JavaScript SDK in 5 minutes.

---

## Overview

The eSign JavaScript SDK is a client library for integrating digital document signing into your web applications. It works in both **Browser** and **Node.js** environments.

| Feature | Supported |
|---------|-----------|
| Browser | ✅ Yes |
| Node.js | ✅ Yes |
| TypeScript | ✅ Yes |
| Zero Dependencies | ✅ Yes (browser) |

---

## Installation

### Browser (CDN/Local)

```html
<!-- Development -->
<script src="dist/esign-sdk.js"></script>

<!-- Production (minified) -->
<script src="dist/esign-sdk.min.js"></script>
```

### Node.js (NPM)

```bash
npm install @capricorn/esign-sdk
```

---

## Quick Start

### Step 1: Include the SDK

=== "Browser"
    ```html
    <script src="esign-sdk.min.js"></script>
    <script>
        const { ESignClient, ESignRequest, ESignMode } = ESignSDK;
    </script>
    ```

=== "Node.js"
    ```javascript
    const { ESignClient, ESignRequest, ESignMode } = require('@capricorn/esign-sdk');
    ```

### Step 2: Initialize Client

```javascript
const client = new ESignClient({
    apiBaseUrl: 'https://your-api-server.com',  // Your eSign API server
    token: 'your-api-token',                     // Token from Capricorn
    key: 'your-api-key'                          // Key from Capricorn
});
```

### Step 3: Sign Document

```javascript
// Create signing request
const response = await client.signDocument(
    new ESignRequest()
        .pdf64(base64EncodedPdf)      // Base64 PDF
        .title('Contract Agreement')   // Document title
        .mode(ESignMode.OTP)          // Authentication mode
        .signerName('John Doe')        // Signer name
);

// Redirect user to authentication page
window.open(response.redirectUrl, '_blank');

// Store reference for later
const reference = response.reference;
```

### Step 4: Download Signed PDF

```javascript
// After user completes OTP authentication...
const signedPdf = await client.downloadSignedDocumentBlob(reference);

// Save file
ESignUtils.downloadBlob(signedPdf, 'signed_document.pdf');
```

---

## Authentication Modes

| Mode | Code | Description |
|------|------|-------------|
| **OTP** | `ESignMode.OTP` | OTP to registered mobile |
| **Biometric** | `ESignMode.BIO` | Fingerprint scan |
| **Iris** | `ESignMode.IRIS` | Iris scan |
| **Face** | `ESignMode.FACE` | Face recognition |
| **eKYC** | `ESignMode.CAPRICORN_EKYC` | Pre-verified KYC (requires `ekycId`) |

---

## Complete Browser Example

```html
<!DOCTYPE html>
<html>
<head>
    <title>eSign Demo</title>
</head>
<body>
    <input type="file" id="pdfFile" accept=".pdf">
    <input type="text" id="signerName" placeholder="Your Name">
    <button onclick="signDocument()">Sign Document</button>

    <script src="esign-sdk.min.js"></script>
    <script>
    const { ESignClient, ESignRequest, ESignMode, ESignUtils } = ESignSDK;

    const client = new ESignClient({
        apiBaseUrl: 'https://your-api-server.com',
        token: 'your-token',
        key: 'your-key'
    });

    async function signDocument() {
        // Get file and convert to Base64
        const file = document.getElementById('pdfFile').files[0];
        const pdf64 = await ESignUtils.fileToBase64(file);
        const signerName = document.getElementById('signerName').value;

        // Sign document
        const response = await client.signDocument(
            new ESignRequest()
                .pdf64(pdf64)
                .title(file.name)
                .mode(ESignMode.OTP)
                .signerName(signerName)
        );

        // Redirect to OTP page
        window.open(response.redirectUrl, '_blank');
        
        // Store reference
        localStorage.setItem('esign_ref', response.reference);
    }
    </script>
</body>
</html>
```

---

## Common Issues

| Issue | Solution |
|-------|----------|
| CORS error | Ensure your API server has CORS enabled |
| Network error | Check API URL and server availability |
| Invalid PDF | Verify Base64 encoding is correct |
| Missing ekycId | Add `.ekycId('...')` for eKYC mode |

---

## Next Steps

- [Complete JavaScript SDK Guide](js-sdk-guide.md) - Full API reference
- [eSign API Guide](esign-api-guide.md) - Backend API documentation
- [API Reference Card](api-reference-card.md) - Quick reference

---

!!! note "Server Required"
    The JavaScript SDK is a client library. You must have the **eSign API server** running.
    See [Deployment Checklist](deployment-checklist.md) for server setup.

---

**Version:** 1.0.0  
**Last Updated:** December 2025  
**© 2025 Capricorn Technologies**
