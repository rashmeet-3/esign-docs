# eSign SDK - Complete Installation Guide

Complete setup guide for Windows, Linux, and Mac.

---

## Table of Contents

1. [Components Overview](#components-overview)
2. [What You Get from Capricorn (ESP)](#what-you-get-from-capricorn-esp)
3. [What You Configure Yourself](#what-you-configure-yourself)
4. [Prerequisites](#prerequisites)
5. [Installation by Operating System](#installation-by-operating-system)
6. [Configuration](#configuration)
7. [Build & Deploy](#build--deploy)
8. [Starting & Stopping Services](#starting--stopping-services)
9. [Testing with ngrok](#testing-with-ngrok)
10. [When to Rebuild](#when-to-rebuild)
11. [Common Errors & Solutions](#common-errors--solutions)
12. [Quick Reference](#quick-reference)

---

## Components Overview

| Component | Port | Description |
|-----------|------|-------------|
| **tomcat_esign** | 8080 | Core SDK + Web UI (browser testing) |
| **esign-api** | 8081 | REST API for JavaScript/Android SDKs |
| **JavaScript SDK** | - | Browser-based client library |
| **Android SDK (AAR)** | - | Mobile client library |

---

## What You Get from Capricorn (ESP)

When you register with **Capricorn Technologies** (the ESP Provider), they send you:

### Files

| File | Description | Where to Place |
|------|-------------|----------------|
| `eSignLicense` | License file to activate SDK | `config/eSignLicense` (in both projects) |
| `privatekey.pfx` | Digital certificate for signing | `config/privatekey.pfx` (in both projects) |

### Credentials

| Credential | Description | Example |
|------------|-------------|---------|
| **ASP ID** | Your organization's unique ID | `yourcompanyaspid` |
| **Signer ID** | eKYC signer identifier (for 3.2 mode) | `user@yourcompany.capricorn` |
| **Certificate Password** | Password for privatekey.pfx | `your_password` |
| **ESP URL (2.1)** | OTP signing endpoint | `https://demo.esign.digital/esign/2.1/signdoc/` |
| **ESP URL (3.2)** | eKYC signing endpoint | `https://demo.esign.digital/esign/3.2/signdoc/` |

### Demo vs Production

| Environment | Purpose |
|-------------|---------|
| Demo URLs | Testing and development |
| Production URLs | Live document signing |

### How to Get Credentials

1. Contact Capricorn Technologies
2. Complete registration process
3. Sign agreement and pay fees
4. Receive credentials via secure channel

---

## What You Configure Yourself

These are NOT from Capricorn - **you create them**:

| Credential | Description | Purpose |
|------------|-------------|---------|
| **API Token** | Any secure string | Authenticates SDK clients |
| **API Key** | Any secure string | Additional authentication |
| **Callback URL** | Your ngrok/domain URL | Where ESP sends signed documents |

**Example:**
```properties
api.auth.token=MY_SECURE_TOKEN_12345
api.auth.key=my_secure_key_abcdef
```

---

## Prerequisites

### Required Software

| Software | Minimum Version | Purpose |
|----------|-----------------|---------|
| **Java JDK** | 17 | Runtime and compilation |
| **Apache Maven** | 3.6.0 | Build tool |
| **ngrok** | Any | HTTPS tunnel for testing |

### HTTPS for Callbacks

**HTTPS is recommended** for callback URLs.

- **Development:** ngrok provides HTTPS automatically (free)
- **Production:** Use your own domain with SSL certificate

HTTP might work in some cases, but HTTPS is preferred by ESP providers.

---

## Installation by Operating System

### Windows

#### Automatic (Recommended)
```cmd
install-prerequisites.bat    (Run as Administrator)
```

#### Manual
```cmd
choco install temurin17 -y
choco install maven -y
choco install ngrok -y
```

#### Manual Download
- Java 17: https://adoptium.net/
- Maven: https://maven.apache.org/download.cgi
- ngrok: https://ngrok.com/download

### Linux (Ubuntu/Debian)

#### Automatic
```bash
chmod +x install-prerequisites.sh
./install-prerequisites.sh
```

#### Manual
```bash
sudo apt update
sudo apt install -y openjdk-17-jdk maven

# ngrok
curl -s https://ngrok-agent.s3.amazonaws.com/ngrok.asc | sudo tee /etc/apt/trusted.gpg.d/ngrok.asc >/dev/null
echo "deb https://ngrok-agent.s3.amazonaws.com buster main" | sudo tee /etc/apt/sources.list.d/ngrok.list
sudo apt update && sudo apt install -y ngrok
```

### Mac

#### Automatic
```bash
chmod +x install-prerequisites.sh
./install-prerequisites.sh
```

#### Manual (Homebrew)
```bash
brew install openjdk@17 maven ngrok
echo 'export PATH="/opt/homebrew/opt/openjdk@17/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

### Verify Installation

```bash
java -version      # Should show 17+
mvn -version       # Should show 3.6+
ngrok version      # Any version
```

---

## Configuration

**IMPORTANT:** Both `tomcat_esign` and `esign-api` need to be configured!

### Files to Configure

```
esign-sdk-complete/
├── tomcat_esign/
│   ├── config/
│   │   ├── eSignLicense              ← Replace with YOUR license
│   │   └── privatekey.pfx            ← Replace with YOUR certificate
│   └── src/main/resources/
│       └── application.properties    ← Update settings
│
└── esign-api/
    ├── config/
    │   ├── eSignLicense              ← Replace with YOUR license
    │   └── privatekey.pfx            ← Replace with YOUR certificate
    └── src/main/resources/
        └── application.properties    ← Update settings
```

### Step 1: Replace License Files (Both Projects)

```bash
# Windows
copy "C:\path\to\your\eSignLicense" "tomcat_esign\config\"
copy "C:\path\to\your\privatekey.pfx" "tomcat_esign\config\"
copy "C:\path\to\your\eSignLicense" "esign-api\config\"
copy "C:\path\to\your\privatekey.pfx" "esign-api\config\"

# Linux/Mac
cp /path/to/your/eSignLicense tomcat_esign/config/
cp /path/to/your/privatekey.pfx tomcat_esign/config/
cp /path/to/your/eSignLicense esign-api/config/
cp /path/to/your/privatekey.pfx esign-api/config/
```

### Step 2: Configure tomcat_esign

Edit `tomcat_esign/src/main/resources/application.properties`:

```properties
server.port=8080

# ========================================
# ESP URLs (FROM CAPRICORN)
# ========================================
esign.2_1.esp.url=https://demo.esign.digital/esign/2.1/signdoc/
esign.3_2.esp.url=https://demo.esign.digital/esign/3.2/signdoc/

# ========================================
# Callback URLs (YOUR ngrok/domain URL)
# ========================================
esign.2_1.response.url=https://YOUR-NGROK-URL.ngrok-free.dev/api/esign/esp-response
esign.3_2.response.url=https://YOUR-NGROK-URL.ngrok-free.dev/api/esign/3.2/getdocs/
esign.3_2.redirect.url=https://YOUR-NGROK-URL.ngrok-free.dev/api/esign/3.2/callback/
esign.base.url=https://YOUR-NGROK-URL.ngrok-free.dev

# ========================================
# Credentials (FROM CAPRICORN)
# ========================================
esign.asp.id=YOUR_ASP_ID
esign.3_2.signer.id=youruser@yourcompany.capricorn
esign.certificate.password=YOUR_CERTIFICATE_PASSWORD

# ========================================
# File Paths (usually no change needed)
# ========================================
esign.license.path=./config/eSignLicense
esign.certificate.path=./config/privatekey.pfx
esign.temp.path=./temp/
esign.output.path=./signed/
```

### Step 3: Configure esign-api

Edit `esign-api/src/main/resources/application.properties`:

```properties
server.port=8081

# ========================================
# YOUR Public URL (ngrok or domain)
# ========================================
api.base-url=https://YOUR-NGROK-URL.ngrok-free.dev

# ========================================
# API Authentication (YOU CREATE THESE)
# ========================================
api.auth.token=YOUR_TOKEN
api.auth.key=YOUR_KEY

# ========================================
# ESP URLs (FROM CAPRICORN)
# ========================================
esign.2_1.esp.url=https://demo.esign.digital/esign/2.1/signdoc/
esign.3_2.esp.url=https://demo.esign.digital/esign/3.2/signdoc/

# ========================================
# Credentials (FROM CAPRICORN)
# ========================================
esign.asp.id=YOUR_ASP_ID
esign.3_2.signer.id=youruser@yourcompany.capricorn
esign.certificate.password=YOUR_CERTIFICATE_PASSWORD

# ========================================
# File Paths (usually no change needed)
# ========================================
esign.license.path=./config/eSignLicense
esign.certificate.path=./config/privatekey.pfx
```

### Configuration Summary

| Property | Source | Configure In |
|----------|--------|--------------|
| `api.base-url` | Your ngrok URL | esign-api only |
| `api.auth.token` | You create | esign-api only |
| `api.auth.key` | You create | esign-api only |
| `esign.2_1.esp.url` | From Capricorn | Both |
| `esign.3_2.esp.url` | From Capricorn | Both |
| `esign.asp.id` | From Capricorn | Both |
| `esign.3_2.signer.id` | From Capricorn | Both |
| `esign.certificate.password` | From Capricorn | Both |
| `esign.base.url` | Your ngrok URL | tomcat_esign only |
| `esign.2_1.response.url` | Your ngrok URL + path | tomcat_esign only |
| `esign.3_2.response.url` | Your ngrok URL + path | tomcat_esign only |
| `esign.3_2.redirect.url` | Your ngrok URL + path | tomcat_esign only |

---

## Build & Deploy

### Automatic Build (Recommended)

**Windows:**
```cmd
build.bat
```

**Linux/Mac:**
```bash
./build.sh
```

### What Build Does

1. Builds `tomcat_esign` first with `mvn install` (installs SDK to Maven repo)
2. Builds `esign-api` second with `mvn package` (uses SDK from Maven repo)

### Manual Build

```bash
# Step 1: MUST be first!
cd tomcat_esign
mvn clean install -DskipTests

# Step 2: After tomcat_esign succeeds
cd ../esign-api
mvn clean package -DskipTests
```

### Build Output

```
tomcat_esign/target/br-esign-sdk-1.0.0.jar    (Core SDK + Web UI)
esign-api/target/esign-api-1.0.0.jar          (REST API)
```

---

## Starting & Stopping Services

### Start REST API (for SDK usage)

**Windows:**
```cmd
start.bat
```

**Linux/Mac:**
```bash
./start.sh
```

**URL:** http://localhost:8081

### Start Web UI (for browser testing) - Optional

**Windows:**
```cmd
start-ui.bat
```

**Linux/Mac:**
```bash
./start-ui.sh
```

**URL:** http://localhost:8080

### Stop Services

- Press `Ctrl+C` in the terminal
- Windows: `taskkill /F /IM java.exe`
- Linux/Mac: `./stop.sh` or `pkill -f esign`

### Verify Running

```bash
curl http://localhost:8081/api/v1/esign/health
# Response: {"status":"UP"}
```

---

## Testing with ngrok

### Step-by-Step

**Terminal 1: Start API**
```bash
./start.sh   # or start.bat
```

**Terminal 2: Start ngrok**
```bash
ngrok http 8081
```

Copy the URL: `https://abc123.ngrok-free.dev`

**Update configuration in BOTH projects:**

In `esign-api/src/main/resources/application.properties`:
```properties
api.base-url=https://abc123.ngrok-free.dev
```

In `tomcat_esign/src/main/resources/application.properties`:
```properties
esign.2_1.response.url=https://abc123.ngrok-free.dev/api/esign/esp-response
esign.3_2.response.url=https://abc123.ngrok-free.dev/api/esign/3.2/getdocs/
esign.3_2.redirect.url=https://abc123.ngrok-free.dev/api/esign/3.2/callback/
esign.base.url=https://abc123.ngrok-free.dev
```

**Rebuild and restart:**
```bash
./build.sh   # or build.bat
./start.sh   # or start.bat
```

**Test:**
```bash
curl https://abc123.ngrok-free.dev/api/v1/esign/health
```

### ngrok URL Changes

Every time ngrok restarts, you get a new URL. You must:
1. Copy new ngrok URL
2. Update BOTH `application.properties` files
3. Rebuild: `./build.sh`
4. Restart: `./start.sh`

---

## When to Rebuild

### Quick Reference

| What Changed | Rebuild Needed? | Action |
|--------------|-----------------|--------|
| License files only | ❌ No | Just restart |
| `application.properties` | ✅ Yes | Rebuild + Restart |
| ngrok URL changed | ✅ Yes | Rebuild + Restart |
| Java source code | ✅ Yes | Rebuild + Restart |

### Avoid Rebuild with External Config

Create config file next to JAR:

```
esign-api/
├── target/
│   └── esign-api-1.0.0.jar
└── config/
    └── application.properties    ← External config
```

Spring Boot reads external config automatically - no rebuild needed!

---

## Common Errors & Solutions

### Error: Could not find artifact br-esign-sdk

```
Could not find artifact com.capricorn:br-esign-sdk:1.0.0
```

**Cause:** tomcat_esign not built or not installed to Maven repo.

**Solution:**
```bash
cd tomcat_esign
mvn clean install -DskipTests
```

---

### Error: Port Already in Use

```
Port 8081 was already in use
```

**Solution:**

Windows:
```cmd
netstat -ano | findstr :8081
taskkill /PID <PID> /F
```

Linux/Mac:
```bash
lsof -i :8081
kill -9 <PID>
```

---

### Error: License Not Found

```
License file not found: ./config/eSignLicense
```

**Solution:**
1. Check file exists in `config/` folder of BOTH projects
2. Filename must be exactly `eSignLicense` (case-sensitive)

---

### Error: Certificate Password Wrong

```
keystore password was incorrect
```

**Solution:**
Verify `esign.certificate.password` matches what Capricorn provided (in BOTH projects).

---

### Error: CORS Blocked

```
Access blocked by CORS policy
```

**Solution:**
Ensure `CorsConfig.java` exists in esign-api and rebuild.

---

### Error: No Main Manifest

```
no main manifest attribute in jar
```

**Solution:**
Update `pom.xml` with correct spring-boot-maven-plugin configuration:

```xml
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <mainClass>com.capricorn.esign.ESignApplication</mainClass>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>repackage</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

Then rebuild.

---

### Error: ESP Callback Failed

```
ESP callback URL not reachable
```

**Solution:**
1. Verify ngrok is running
2. Check ngrok URL is correct in BOTH `application.properties`
3. Rebuild and restart

---

## Quick Reference

### Commands

| Task | Windows | Linux/Mac |
|------|---------|-----------|
| Install prerequisites | `install-prerequisites.bat` | `./install-prerequisites.sh` |
| Build all | `build.bat` | `./build.sh` |
| Start REST API | `start.bat` | `./start.sh` |
| Start Web UI | `start-ui.bat` | `./start-ui.sh` |
| Start ngrok | `ngrok http 8081` | `ngrok http 8081` |

### URLs

| Service | URL |
|---------|-----|
| REST API | http://localhost:8081 |
| Health Check | http://localhost:8081/api/v1/esign/health |
| Web UI | http://localhost:8080 |

### Important Files

| File | Purpose |
|------|---------|
| `tomcat_esign/src/main/resources/application.properties` | tomcat_esign configuration |
| `esign-api/src/main/resources/application.properties` | esign-api configuration |
| `tomcat_esign/config/eSignLicense` | License (tomcat_esign) |
| `esign-api/config/eSignLicense` | License (esign-api) |
| `tomcat_esign/config/privatekey.pfx` | Certificate (tomcat_esign) |
| `esign-api/config/privatekey.pfx` | Certificate (esign-api) |

---

**Version:** 1.0.0 | **Last Updated:** January 2026
