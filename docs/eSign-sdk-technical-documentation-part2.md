# eSign SDK v1.0.0 - Technical Documentation (Part 2)

## 7.3 Detailed API Reference

### 7.3.1 Set eSign Mode

**Endpoint:** `POST /api/esign/set-mode`

**Description:** Set the eSign mode to either 2.1 (OTP-based) or 3.2 (eKYC-based)

**Request:**
```json
{
  "mode": "2.1"
}
```

**Request Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `mode` | string | Yes | eSign mode: "2.1" or "3.2" |

**Response (Success):**
```json
{
  "success": true,
  "mode": "2.1",
  "modeName": "OTP-based Signing",
  "message": "eSign mode set to 2.1"
}
```

**Response (Error):**
```json
{
  "success": false,
  "error": "Invalid mode. Use '2.1' or '3.2'"
}
```

**Example (cURL):**
```bash
curl -X POST http://localhost:8080/api/esign/set-mode \
  -H "Content-Type: application/json" \
  -d '{"mode":"2.1"}'
```

---

### 7.3.2 Serve PDF Document

**Endpoint:** `GET /api/esign/documents/{documentId}.pdf`

**Description:** Serve a PDF document publicly (required for eSign 3.2)

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `documentId` | string | Document identifier (UUID) |

**Response:** PDF file (application/pdf)

**Example:**
```bash
curl http://localhost:8080/api/esign/documents/abc123.pdf -o document.pdf
```

---

### 7.3.3 Step 1: Upload PDF

**Endpoint:** `POST /api/esign/step1/upload`

**Description:** Upload a PDF file for signing

**Request:** Multipart form data
- `file`: PDF file (max 50MB)

**Response (Success):**
```json
{
  "success": true,
  "step": 1,
  "message": "PDF uploaded successfully",
  "fileName": "contract.pdf",
  "fileSize": 245678,
  "savedPath": "./temp/abc123.pdf",
  "documentId": "abc123",
  "documentPublicUrl": "http://localhost:8080/api/esign/documents/abc123.pdf",
  "esignMode": "2.1"
}
```

**Example (cURL):**
```bash
curl -X POST http://localhost:8080/api/esign/step1/upload \
  -F "file=@/path/to/document.pdf"
```

**Example (JavaScript):**
```javascript
const formData = new FormData();
formData.append('file', pdfFile);

fetch('http://localhost:8080/api/esign/step1/upload', {
  method: 'POST',
  body: formData
})
.then(response => response.json())
.then(data => console.log(data));
```

---

### 7.3.4 Step 2: Set Signer Details

**Endpoint:** `POST /api/esign/step2/signer-details`

**Description:** Set signer information and signature position

**Request:**
```json
{
  "signerName": "John Doe",
  "docInfo": "Employment Contract",
  "pageNo": 1,
  "x1": 350,
  "y1": 50,
  "x2": 550,
  "y2": 120
}
```

**Request Parameters:**

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `signerName` | string | Yes | - | Name of the signer |
| `docInfo` | string | Yes | - | Document description |
| `pageNo` | integer | No | 1 | Page number for signature |
| `x1` | integer | No | 350 | Signature box X1 coordinate |
| `y1` | integer | No | 50 | Signature box Y1 coordinate |
| `x2` | integer | No | 550 | Signature box X2 coordinate |
| `y2` | integer | No | 120 | Signature box Y2 coordinate |

**Response:**
```json
{
  "success": true,
  "step": 2,
  "message": "Signer details set successfully",
  "signerName": "John Doe",
  "docInfo": "Employment Contract",
  "signaturePosition": {
    "page": 1,
    "x1": 350,
    "y1": 50,
    "x2": 550,
    "y2": 120
  }
}
```

**Example (cURL):**
```bash
curl -X POST http://localhost:8080/api/esign/step2/signer-details \
  -H "Content-Type: application/json" \
  -d '{
    "signerName": "John Doe",
    "docInfo": "Employment Contract",
    "pageNo": 1,
    "x1": 350,
    "y1": 50,
    "x2": 550,
    "y2": 120
  }'
```

---

### 7.3.5 Step 3: Generate Document Hash

**Endpoint:** `POST /api/esign/step3/generate-hash`

**Description:** Generate SHA-256 hash of the PDF document

**Request:** No body required (uses session data)

**Response:**
```json
{
  "success": true,
  "step": 3,
  "message": "Document hash generated successfully",
  "hashAlgorithm": "SHA-256",
  "hashBytes": [45, 78, 123, ...],
  "hashLength": 32
}
```

**Example:**
```bash
curl -X POST http://localhost:8080/api/esign/step3/generate-hash \
  -H "Content-Type: application/json"
```

---

### 7.3.6 Step 4: Convert Hash to Hex

**Endpoint:** `POST /api/esign/step4/convert-hex`

**Description:** Convert hash bytes to hexadecimal string

**Response:**
```json
{
  "success": true,
  "step": 4,
  "message": "Hash converted to hex successfully",
  "hashHex": "2d4e7b9a...",
  "hashLength": 64
}
```

---

### 7.3.7 Step 5: Generate Transaction ID

**Endpoint:** `POST /api/esign/step5/generate-txn`

**Description:** Generate unique transaction ID for this signing session

**Response:**
```json
{
  "success": true,
  "step": 5,
  "message": "Transaction ID generated successfully",
  "transactionId": "UKC:public:abc12345",
  "timestamp": "2025-12-15T15:30:00"
}
```

---

### 7.3.8 Step 6: Construct XML Request

**Endpoint:** `POST /api/esign/step6/construct-xml`

**Description:** Construct eSign XML request according to protocol

**Response:**
```json
{
  "success": true,
  "step": 6,
  "message": "XML request constructed successfully",
  "xmlPreview": "<?xml version=\"1.0\"?><Esign ver=\"2.1\" ...>",
  "xmlLength": 2456
}
```

---

### 7.3.9 Step 7: Sign XML with Certificate

**Endpoint:** `POST /api/esign/step7/sign-xml`

**Description:** Sign the XML request with ASP certificate

**Response:**
```json
{
  "success": true,
  "step": 7,
  "message": "XML signed successfully",
  "signedXmlPreview": "<?xml version=\"1.0\"?><Esign ...><Signature ...>",
  "signatureAlgorithm": "RSA-SHA256",
  "certificateSubject": "CN=YourASP, O=YourOrg, C=IN"
}
```

---

### 7.3.10 Step 8: Generate ESP Redirect Form

**Endpoint:** `POST /api/esign/step8/generate-esp-form`

**Description:** Generate HTML form to redirect user to ESP

**Response:**
```json
{
  "success": true,
  "step": 8,
  "message": "ESP redirect form generated",
  "espUrl": "https://demo.esign.digital/esign/2.1/signdoc/",
  "htmlForm": "<html><body><form method='POST' ...>",
  "autoSubmit": true
}
```

**Usage:** Load the `htmlForm` in browser to redirect user to ESP

---

### 7.3.11 ESP Response Callback (eSign 2.1)

**Endpoint:** `POST /api/esign/esp-response`

**Description:** Callback endpoint where ESP posts the signed response (eSign 2.1)

**Request Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `esignresp` | string | Signed XML response from ESP |
| `msg` | string | Alternative parameter name |
| `eSignResponse` | string | Alternative parameter name |

**Response:** HTML page displaying the response

**Note:** This endpoint is called by ESP, not by your application

---

### 7.3.12 ESP GetDocs Callback (eSign 3.2)

**Endpoint:** `POST /api/esign/3.2/getdocs/`

**Description:** Callback endpoint where ESP posts signed documents (eSign 3.2)

**Response:** `OK` (acknowledgment)

---

### 7.3.13 ESP Redirect Callback (eSign 3.2)

**Endpoint:** `GET/POST /api/esign/3.2/callback/`

**Description:** Redirect URL where user is sent after signing (eSign 3.2)

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `txnRef` | string | Base64-encoded transaction reference |
| `status` | string | Signing status |
| `txn` | string | Transaction ID |

**Response:** HTML page with signing status

---

### 7.3.14 Get ESP Response Data

**Endpoint:** `GET /api/esign/esp-response-data`

**Description:** Retrieve ESP response data from session

**Response:**
```json
{
  "success": true,
  "espResponseReceived": true,
  "responseXml": "<?xml version=\"1.0\"?>...",
  "pkcs7Signature": "MIIGxwYJKoZIhvcNAQcCoIIGuDCC...",
  "transactionId": "UKC:public:abc12345"
}
```

---

### 7.3.15 Embed Signature (Manual)

**Endpoint:** `POST /api/esign/embed-signature`

**Description:** Manually embed PKCS#7 signature into PDF

**Request:**
```json
{
  "transactionId": "UKC:public:abc12345"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Signature embedded successfully",
  "signedPdfPath": "./signed/contract_signed_abc123.pdf",
  "downloadUrl": "/download/contract_signed_abc123.pdf"
}
```

---

### 7.3.16 Step 9: Embed Signature (Workflow)

**Endpoint:** `POST /api/esign/step9/embed-signature`

**Description:** Step 9 in the workflow - embed signature into PDF

**Request:**
```json
{
  "outputFileName": "contract_signed.pdf"
}
```

**Response:**
```json
{
  "success": true,
  "step": 9,
  "message": "Signature embedded successfully",
  "signedPdfPath": "./signed/contract_signed.pdf",
  "fileSize": 256789,
  "signerName": "John Doe",
  "signedAt": "2025-12-15T15:35:00"
}
```

---

### 7.3.17 Get Configuration

**Endpoint:** `GET /api/esign/config`

**Description:** Get current configuration (sanitized)

**Response:**
```json
{
  "success": true,
  "config": {
    "esp21Url": "https://demo.esign.digital/esign/2.1/signdoc/",
    "esp32Url": "https://demo.esign.digital/esign/3.2/signdoc/",
    "aspId": "capricornaspid",
    "baseUrl": "http://localhost:8080",
    "tempPath": "./temp/",
    "outputPath": "./signed/"
  }
}
```

**Note:** Sensitive data (passwords, certificates) are not included

---

### 7.3.18 Reset Session

**Endpoint:** `POST /api/esign/reset`

**Description:** Reset the current signing session

**Response:**
```json
{
  "success": true,
  "message": "Session reset successfully"
}
```

---

## 8. Usage Guide

### 8.1 Complete Workflow - eSign 2.1 (OTP-based)

#### Step-by-Step Process

**Step 1: Set Mode**
```bash
curl -X POST http://localhost:8080/api/esign/set-mode \
  -H "Content-Type: application/json" \
  -d '{"mode":"2.1"}'
```

**Step 2: Upload PDF**
```bash
curl -X POST http://localhost:8080/api/esign/step1/upload \
  -F "file=@contract.pdf"
```

**Step 3: Set Signer Details**
```bash
curl -X POST http://localhost:8080/api/esign/step2/signer-details \
  -H "Content-Type: application/json" \
  -d '{
    "signerName": "John Doe",
    "docInfo": "Employment Contract"
  }'
```

**Step 4: Generate Hash**
```bash
curl -X POST http://localhost:8080/api/esign/step3/generate-hash
```

**Step 5: Convert to Hex**
```bash
curl -X POST http://localhost:8080/api/esign/step4/convert-hex
```

**Step 6: Generate Transaction ID**
```bash
curl -X POST http://localhost:8080/api/esign/step5/generate-txn
```

**Step 7: Construct XML**
```bash
curl -X POST http://localhost:8080/api/esign/step6/construct-xml
```

**Step 8: Sign XML**
```bash
curl -X POST http://localhost:8080/api/esign/step7/sign-xml
```

**Step 9: Generate ESP Form**
```bash
curl -X POST http://localhost:8080/api/esign/step8/generate-esp-form
```

**Step 10: User Signs at ESP**
- Load the HTML form in browser
- User enters Aadhaar number and OTP
- ESP redirects back to your callback URL

**Step 11: Embed Signature**
```bash
curl -X POST http://localhost:8080/api/esign/step9/embed-signature \
  -H "Content-Type: application/json" \
  -d '{"outputFileName": "contract_signed.pdf"}'
```

---

### 8.2 Complete Workflow - eSign 3.2 (eKYC-based)

The workflow is similar to 2.1, with these differences:

1. **Set mode to 3.2:**
```bash
curl -X POST http://localhost:8080/api/esign/set-mode \
  -H "Content-Type: application/json" \
  -d '{"mode":"3.2"}'
```

2. **Different ESP URLs** are used automatically
3. **Two callbacks:**
   - `/api/esign/3.2/getdocs/` - ESP posts signature
   - `/api/esign/3.2/callback/` - User redirect

---

### 8.3 JavaScript Integration Example

```javascript
class ESignClient {
  constructor(baseUrl = 'http://localhost:8080') {
    this.baseUrl = baseUrl;
    this.apiBase = `${baseUrl}/api/esign`;
  }

  async setMode(mode) {
    const response = await fetch(`${this.apiBase}/set-mode`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ mode })
    });
    return response.json();
  }

  async uploadPdf(file) {
    const formData = new FormData();
    formData.append('file', file);
    
    const response = await fetch(`${this.apiBase}/step1/upload`, {
      method: 'POST',
      body: formData
    });
    return response.json();
  }

  async setSignerDetails(signerName, docInfo, position = {}) {
    const response = await fetch(`${this.apiBase}/step2/signer-details`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        signerName,
        docInfo,
        pageNo: position.pageNo || 1,
        x1: position.x1 || 350,
        y1: position.y1 || 50,
        x2: position.x2 || 550,
        y2: position.y2 || 120
      })
    });
    return response.json();
  }

  async executeStep(stepNumber) {
    const endpoints = {
      3: 'step3/generate-hash',
      4: 'step4/convert-hex',
      5: 'step5/generate-txn',
      6: 'step6/construct-xml',
      7: 'step7/sign-xml',
      8: 'step8/generate-esp-form'
    };

    const response = await fetch(`${this.apiBase}/${endpoints[stepNumber]}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' }
    });
    return response.json();
  }

  async embedSignature(outputFileName) {
    const response = await fetch(`${this.apiBase}/step9/embed-signature`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ outputFileName })
    });
    return response.json();
  }

  async completeSigningWorkflow(pdfFile, signerName, docInfo) {
    try {
      // Step 1: Upload PDF
      console.log('Uploading PDF...');
      await this.uploadPdf(pdfFile);

      // Step 2: Set signer details
      console.log('Setting signer details...');
      await this.setSignerDetails(signerName, docInfo);

      // Steps 3-8: Execute workflow
      for (let step = 3; step <= 8; step++) {
        console.log(`Executing step ${step}...`);
        const result = await this.executeStep(step);
        
        if (step === 8) {
          // Step 8 returns HTML form - display it
          document.body.innerHTML = result.htmlForm;
          return; // User will be redirected to ESP
        }
      }
    } catch (error) {
      console.error('Error in signing workflow:', error);
      throw error;
    }
  }
}

// Usage
const esign = new ESignClient();
const fileInput = document.getElementById('pdfFile');

esign.completeSigningWorkflow(
  fileInput.files[0],
  'John Doe',
  'Employment Contract'
);
```

---

### 8.4 Python Integration Example

```python
import requests

class ESignClient:
    def __init__(self, base_url='http://localhost:8080'):
        self.base_url = base_url
        self.api_base = f'{base_url}/api/esign'
        self.session = requests.Session()
    
    def set_mode(self, mode):
        response = self.session.post(
            f'{self.api_base}/set-mode',
            json={'mode': mode}
        )
        return response.json()
    
    def upload_pdf(self, pdf_path):
        with open(pdf_path, 'rb') as f:
            files = {'file': f}
            response = self.session.post(
                f'{self.api_base}/step1/upload',
                files=files
            )
        return response.json()
    
    def set_signer_details(self, signer_name, doc_info, **position):
        data = {
            'signerName': signer_name,
            'docInfo': doc_info,
            'pageNo': position.get('pageNo', 1),
            'x1': position.get('x1', 350),
            'y1': position.get('y1', 50),
            'x2': position.get('x2', 550),
            'y2': position.get('y2', 120)
        }
        response = self.session.post(
            f'{self.api_base}/step2/signer-details',
            json=data
        )
        return response.json()
    
    def execute_step(self, step_number):
        endpoints = {
            3: 'step3/generate-hash',
            4: 'step4/convert-hex',
            5: 'step5/generate-txn',
            6: 'step6/construct-xml',
            7: 'step7/sign-xml',
            8: 'step8/generate-esp-form'
        }
        response = self.session.post(
            f'{self.api_base}/{endpoints[step_number]}'
        )
        return response.json()
    
    def embed_signature(self, output_filename):
        response = self.session.post(
            f'{self.api_base}/step9/embed-signature',
            json={'outputFileName': output_filename}
        )
        return response.json()
    
    def complete_signing_workflow(self, pdf_path, signer_name, doc_info):
        # Step 1: Upload PDF
        print('Uploading PDF...')
        self.upload_pdf(pdf_path)
        
        # Step 2: Set signer details
        print('Setting signer details...')
        self.set_signer_details(signer_name, doc_info)
        
        # Steps 3-8
        for step in range(3, 9):
            print(f'Executing step {step}...')
            result = self.execute_step(step)
            
            if step == 8:
                # Save ESP form to file
                with open('esp_redirect.html', 'w') as f:
                    f.write(result['htmlForm'])
                print('ESP redirect form saved to esp_redirect.html')
                print('Open this file in browser to complete signing')
                return result

# Usage
client = ESignClient()
client.set_mode('2.1')
client.complete_signing_workflow(
    'contract.pdf',
    'John Doe',
    'Employment Contract'
)
```

---

### 8.5 Java Integration Example

```java
import java.io.*;
import java.net.http.*;
import java.nio.file.*;
import com.google.gson.*;

public class ESignClient {
    private final String baseUrl;
    private final String apiBase;
    private final HttpClient httpClient;
    private final Gson gson;
    private final CookieManager cookieManager;
    
    public ESignClient(String baseUrl) {
        this.baseUrl = baseUrl;
        this.apiBase = baseUrl + "/api/esign";
        this.cookieManager = new CookieManager();
        this.httpClient = HttpClient.newBuilder()
            .cookieHandler(cookieManager)
            .build();
        this.gson = new Gson();
    }
    
    public JsonObject setMode(String mode) throws Exception {
        JsonObject request = new JsonObject();
        request.addProperty("mode", mode);
        
        HttpRequest httpRequest = HttpRequest.newBuilder()
            .uri(URI.create(apiBase + "/set-mode"))
            .header("Content-Type", "application/json")
            .POST(HttpRequest.BodyPublishers.ofString(gson.toJson(request)))
            .build();
        
        HttpResponse<String> response = httpClient.send(
            httpRequest, 
            HttpResponse.BodyHandlers.ofString()
        );
        
        return gson.fromJson(response.body(), JsonObject.class);
    }
    
    public JsonObject uploadPdf(Path pdfPath) throws Exception {
        byte[] pdfBytes = Files.readAllBytes(pdfPath);
        String boundary = "----Boundary" + System.currentTimeMillis();
        
        ByteArrayOutputStream outputStream = new ByteArrayOutputStream();
        PrintWriter writer = new PrintWriter(new OutputStreamWriter(outputStream));
        
        writer.append("--").append(boundary).append("\r\n");
        writer.append("Content-Disposition: form-data; name=\"file\"; filename=\"")
              .append(pdfPath.getFileName().toString()).append("\"\r\n");
        writer.append("Content-Type: application/pdf\r\n\r\n");
        writer.flush();
        
        outputStream.write(pdfBytes);
        
        writer.append("\r\n--").append(boundary).append("--\r\n");
        writer.flush();
        
        HttpRequest request = HttpRequest.newBuilder()
            .uri(URI.create(apiBase + "/step1/upload"))
            .header("Content-Type", "multipart/form-data; boundary=" + boundary)
            .POST(HttpRequest.BodyPublishers.ofByteArray(outputStream.toByteArray()))
            .build();
        
        HttpResponse<String> response = httpClient.send(
            request,
            HttpResponse.BodyHandlers.ofString()
        );
        
        return gson.fromJson(response.body(), JsonObject.class);
    }
    
    // Additional methods similar to Python example...
}
```

---

## 9. Security & Compliance

### 9.1 Security Features

#### 9.1.1 Cryptographic Security
- **Hash Algorithm**: SHA-256 (FIPS 140-2 compliant)
- **Signature Algorithm**: RSA with SHA-256
- **Signature Format**: PKCS#7 (CMS)
- **Certificate Standard**: X.509 v3
- **Key Length**: Minimum 2048-bit RSA

#### 9.1.2 XML Security
- **XML Signature**: W3C XML-Signature standard
- **Canonicalization**: Exclusive XML Canonicalization
- **Transforms**: Enveloped signature transform
- **Certificate Embedding**: X.509 certificate included in signature

#### 9.1.3 Transport Security
- **Protocol**: HTTPS/TLS 1.2+ for ESP communication
- **Certificate Validation**: Full chain validation
- **Session Security**: HTTP-only, secure cookies (in production)

### 9.2 Compliance

#### 9.2.1 Indian IT Act Compliance
- ✅ Compliant with IT Act 2000 (amended 2008)
- ✅ Follows CCA guidelines for digital signatures
- ✅ Aadhaar-based authentication (eSign 2.1 & 3.2)

#### 9.2.2 Data Protection
- **Personal Data**: Minimal collection (only signer name)
- **Data Retention**: Session-based (cleared after signing)
- **File Storage**: Temporary files auto-deleted (configurable)
- **Logging**: No sensitive data in logs

### 9.3 Security Best Practices

#### 9.3.1 Production Deployment
1. **Use HTTPS**: Deploy behind reverse proxy with SSL/TLS
2. **Secure Configuration**: Store credentials in environment variables
3. **File Permissions**: Restrict access to config files
4. **Regular Updates**: Keep Java and dependencies updated
5. **Firewall**: Limit access to necessary ports only

#### 9.3.2 Certificate Management
- Store `.pfx` files securely (encrypted filesystem)
- Use strong passwords (minimum 12 characters)
- Rotate certificates before expiry
- Backup certificates securely

#### 9.3.3 Session Management
- Configure session timeout (default: 30 minutes)
- Use secure session cookies in production
- Clear sensitive session data after signing

---

## 10. Troubleshooting

### 10.1 Common Issues

#### 10.1.1 Application Won't Start

**Issue:** Port 8080 already in use
```
Error: Port 8080 is already in use
```

**Solution:**
```properties
# Change port in application.properties
server.port=8081
```

---

**Issue:** Java version error
```
Error: Unsupported class file major version 61
```

**Solution:** Install Java 17 or higher
```bash
java -version  # Check current version
# Install Java 17
```

---

**Issue:** License file not found
```
Error: License file not found at ./config/eSignLicense
```

**Solution:** Check file path and permissions
```bash
ls -la ./config/eSignLicense
# Ensure file exists and is readable
```

---

#### 10.1.2 PDF Upload Issues

**Issue:** File too large
```json
{
  "error": "Maximum upload size exceeded"
}
```

**Solution:** Increase upload limit
```properties
spring.servlet.multipart.max-file-size=100MB
spring.servlet.multipart.max-request-size=100MB
```

---

**Issue:** Invalid PDF format
```json
{
  "error": "Invalid PDF file"
}
```

**Solution:** Ensure file is valid PDF (not corrupted)

---

#### 10.1.3 ESP Communication Issues

**Issue:** ESP callback not received
```
ESP response timeout
```

**Solutions:**
1. **Check callback URL is publicly accessible**
   ```bash
   # Test with ngrok for local development
   ngrok http 8080
   # Use ngrok URL in application.properties
   ```

2. **Verify firewall allows inbound connections**

3. **Check ESP logs** for errors

---

**Issue:** XML signature validation failed
```
Error: Invalid XML signature
```

**Solutions:**
1. Verify certificate is valid and not expired
2. Check certificate password is correct
3. Ensure certificate is in PKCS#12 (.pfx) format

---

#### 10.1.4 Signature Embedding Issues

**Issue:** Signature field not found
```
Error: Signature field 'Signature1' not found in PDF
```

**Solution:** Ensure PDF was prepared with placeholder in Step 3

---

**Issue:** Invalid PKCS#7 signature
```
Error: Invalid PKCS#7 signature data
```

**Solutions:**
1. Check ESP response was received correctly
2. Verify signature data is not corrupted
3. Check session data is intact

---

### 10.2 Debugging

#### 10.2.1 Enable Debug Logging

Add to `application.properties`:
```properties
logging.level.com.capricorn.esign=DEBUG
logging.level.org.springframework=INFO
```

#### 10.2.2 Check Application Logs

**Linux:**
```bash
journalctl -u esign-sdk -f
```

**Docker:**
```bash
docker logs -f esign-sdk
```

**Windows:**
Check console output or redirect to file:
```powershell
java -jar esign-sdk-1.0.jar > esign.log 2>&1
```

#### 10.2.3 Test API Endpoints

```bash
# Test health
curl http://localhost:8080/api/esign/config

# Test mode setting
curl -X POST http://localhost:8080/api/esign/set-mode \
  -H "Content-Type: application/json" \
  -d '{"mode":"2.1"}'
```

---

### 10.3 Performance Tuning

#### 10.3.1 JVM Options

For production, use these JVM options:
```bash
java -Xms2G -Xmx4G \
  -XX:+UseG1GC \
  -XX:MaxGCPauseMillis=200 \
  -jar esign-sdk-1.0.jar
```

#### 10.3.2 File Cleanup

Configure automatic cleanup of old files:
```bash
# Add cron job (Linux)
0 2 * * * find /opt/esign-sdk/temp -type f -mtime +7 -delete
0 2 * * * find /opt/esign-sdk/signed -type f -mtime +30 -delete
```

---

## 11. Appendices

### 11.1 Glossary

| Term | Definition |
|------|------------|
| **ASP** | Application Service Provider - Entity authorized to use eSign |
| **ESP** | eSign Service Provider - Government-authorized signing service |
| **CCA** | Controller of Certifying Authorities - Regulatory body |
| **PKCS#7** | Public Key Cryptography Standard #7 - Signature format |
| **X.509** | Standard for public key certificates |
| **SHA-256** | Secure Hash Algorithm 256-bit |
| **OTP** | One-Time Password |
| **eKYC** | Electronic Know Your Customer |
| **Aadhaar** | India's biometric identification system |

### 11.2 File Structure Reference

```
esign-sdk-1.0/
├── esign-sdk-1.0.jar          # Main application JAR
├── application.properties        # Configuration file
├── config/                       # Configuration directory
│   ├── eSignLicense             # eSign license file
│   ├── privatekey.pfx           # ASP certificate
│   └── sample.pdf               # Sample PDF for testing
├── temp/                         # Temporary files (auto-created)
│   ├── {uuid}.pdf               # Uploaded PDFs
│   └── {uuid}_prepared.pdf      # Prepared PDFs with placeholders
├── signed/                       # Signed PDFs (auto-created)
│   └── {filename}_signed.pdf    # Final signed documents
└── logs/                         # Application logs (optional)
    └── esign-sdk.log
```

### 11.3 HTTP Status Codes

| Code | Meaning | When Used |
|------|---------|-----------|
| 200 | OK | Successful request |
| 400 | Bad Request | Invalid parameters |
| 404 | Not Found | Resource not found |
| 500 | Internal Server Error | Server-side error |

### 11.4 Error Codes

| Error Code | Description | Solution |
|------------|-------------|----------|
| `INVALID_MODE` | Invalid eSign mode | Use "2.1" or "3.2" |
| `FILE_NOT_FOUND` | PDF file not found | Check file path |
| `INVALID_PDF` | Invalid PDF format | Use valid PDF file |
| `LICENSE_ERROR` | License file error | Check license file |
| `CERT_ERROR` | Certificate error | Verify certificate and password |
| `ESP_ERROR` | ESP communication error | Check network and ESP URL |
| `SIGNATURE_ERROR` | Signature embedding error | Check ESP response |

### 11.5 Support & Resources

**Documentation:**
- eSign API Specification: https://esign.digital/docs
- CCA Guidelines: https://cca.gov.in

**Support:**
- Email: support@www.esign.network
- Phone: +91-XXX-XXX-XXXX
- Website: https://www.esign.network/esign-sdk

**Updates:**
- Check for updates: https://www.esign.network/esign-sdk/updates
- Release notes: https://www.esign.network/esign-sdk/releases

---

## Document Revision History

| Version | Date | Changes | Author |
|---------|------|---------|--------|
| 1.0 | 2025-12-15 | Initial release | Capricorn Technologies |

---

**End of Technical Documentation**

© 2025 Capricorn Technologies. All rights reserved.
