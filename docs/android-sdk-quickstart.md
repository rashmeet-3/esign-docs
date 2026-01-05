# Android SDK Quick Start

Get started with the eSign Android SDK in 5 minutes.

---

## Installation

### Step 1: Add AAR File

Copy `esign-sdk-1.0.0.aar` to your `app/libs/` folder.

### Step 2: Update build.gradle

```groovy
// app/build.gradle
dependencies {
    // eSign SDK
    implementation files('libs/esign-sdk-1.0.0.aar')
    
    // Required dependency
    implementation 'com.squareup.okhttp3:okhttp:4.12.0'
}
```

### Step 3: Add Permissions

```xml
<!-- AndroidManifest.xml -->
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
```

---

## Basic Usage

### 1. Initialize Client

```java
import com.capricorn.esign.*;

ESignClient client = new ESignClient(
    "https://your-api-server.com",  // Your eSign API server URL
    "your-api-token",                // Token from Capricorn
    "your-api-key"                   // Key from Capricorn
);
```

### 2. Create & Sign Request

```java
// Create request
ESignRequest request = new ESignRequest.Builder()
    .pdf64(base64Pdf)               // Base64 encoded PDF
    .title("Contract")               // Document title
    .mode(ESignMode.OTP)            // Authentication mode
    .signerName("John Doe")         // Signer name
    .build();

// Sign document
client.signDocument(request, new ESignClient.ESignCallback() {
    @Override
    public void onSuccess(ESignResponse response) {
        // Open WebView with response.getRedirectUrl()
        openWebView(response.getRedirectUrl());
    }

    @Override
    public void onError(String error) {
        Log.e("ESign", error);
    }
});
```

### 3. Download Signed PDF

```java
client.downloadSignedDocument(reference, new ESignClient.DownloadCallback() {
    @Override
    public void onSuccess(byte[] pdfBytes) {
        // Save PDF bytes to file
        savePdfToFile(pdfBytes, "signed.pdf");
    }

    @Override
    public void onError(String error) {
        Log.e("ESign", error);
    }
});
```

---

## Authentication Modes

| Mode | Code | Description |
|------|------|-------------|
| OTP | `ESignMode.OTP` | OTP to registered mobile |
| Biometric | `ESignMode.BIO` | Fingerprint scan |
| Iris | `ESignMode.IRIS` | Iris scan |
| Face | `ESignMode.FACE` | Face recognition |
| eKYC | `ESignMode.CAPRICORN_EKYC` | Pre-verified KYC (requires ekycId) |

---

## Signing Options

```java
SigningOptions options = new SigningOptions.Builder()
    .coordinates(350, 50, 550, 120)  // Position (x1,y1,x2,y2)
    .allPages()                       // Sign all pages
    .reason("Digital Signature")      // Reason
    .location("India")                // Location
    .showGreenTick()                  // Show ✓ icon
    .dateFormat("dd-MMM-yyyy")        // Date format
    .lockPdf(LockMode.NO_LOCK)        // Lock mode
    .build();
```

---

!!! note "Server Required"
    The Android SDK requires the eSign API server running in the backend. 
    See [eSign API Guide](esign-api-guide.md) for server setup.

---

For complete documentation, see [Android SDK Complete Guide](android-sdk-guide.md).
