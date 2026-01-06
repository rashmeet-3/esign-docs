# Complete Installation & Deployment Guide

This guide provides step-by-step instructions for setting up the eSign SDK on Windows, Linux, and Mac systems.

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
10. [Production Deployment](#production-deployment)
11. [Common Errors & Solutions](#common-errors--solutions)
12. [Rebuilding After Configuration Changes](#rebuilding-after-configuration-changes)
13. [Quick Reference](#quick-reference)

---

## Components Overview

| Component | Port | Description |
|-----------|------|-------------|
| **esign-api** | 8081 | REST API for JavaScript/Android SDKs |
| **tomcat_esign** | 8080 | Core Java SDK with web interface (optional) |
| **JavaScript SDK** | - | Browser-based client library |
| **Android SDK (AAR)** | - | Mobile client library |
| **ngrok** | - | Provides HTTPS URL for local testing |

---

## What You Get from Capricorn (ESP)

When you register with **Capricorn Technologies** (the ESP Provider), they provide the following:

### Files You Receive

| File | Description | Where to Place |
|------|-------------|----------------|
| `eSignLicense` | License file to activate the SDK | `config/eSignLicense` |
| `privatekey.pfx` | Digital certificate for signing documents | `config/privatekey.pfx` |

### Credentials You Receive

| Credential | Description | Example Value |
|------------|-------------|---------------|
| **ASP ID** | Your organization's unique Application Service Provider ID | `yourcompanyaspid` |
| **Signer ID** | Your eKYC signer identifier (for eSign 3.2 mode) | `user@yourcompany.capricorn` |
| **Certificate Password** | Password to access the privatekey.pfx file | `your_password` |
| **ESP URL (2.1)** | OTP-based signing endpoint | `https://demo.esign.digital/esign/2.1/signdoc/` |
| **ESP URL (3.2)** | eKYC-based signing endpoint | `https://demo.esign.digital/esign/3.2/signdoc/` |

### Demo vs Production URLs

| Environment | Purpose | When to Use |
|-------------|---------|-------------|
| **Demo URLs** | Testing and development | During development and testing |
| **Production URLs** | Live document signing | After testing is complete |

!!! note "How to Get Credentials"
    1. Contact Capricorn Technologies sales/support team
    2. Complete the registration process
    3. Sign the agreement and pay applicable fees
    4. Receive credentials package via secure channel

---

## What You Configure Yourself

These credentials are NOT from Capricorn - **you create them yourself**:

| Credential | Description | Purpose |
|------------|-------------|---------|
| **API Token** | Any secure string you create | Authenticates clients calling your esign-api |
| **API Key** | Any secure string you create | Additional authentication for your API |
| **Callback URL** | Your public HTTPS URL | Where ESP sends signed documents back |

### Example - Creating Your Own API Credentials

```properties
# You create these yourself - use any secure random strings
api.auth.token=MY_SECURE_TOKEN_12345
api.auth.key=my_secure_key_abcdef123456
```

---

## Prerequisites

### Required Software

| Software | Minimum Version | Recommended Version | Purpose |
|----------|-----------------|---------------------|---------|
| **Java JDK** | 17 | 17 or 21 | Runtime and compilation |
| **Apache Maven** | 3.6.0 | 3.9.x | Build tool |
| **ngrok** | Any | Latest | HTTPS tunneling (for testing) |

### Optional Software

| Software | Version | Purpose |
|----------|---------|---------|
| **Python** | 3.x | Test page server |
| **Node.js** | 14.x+ | For localtunnel (alternative to ngrok) |

### Version Check Commands

```bash
java -version      # Should show 17 or higher
mvn -version       # Should show 3.6.0 or higher
ngrok version      # Any version
```

### About HTTPS for Callbacks

!!! info "HTTPS Recommendation"
    **HTTPS is recommended** for callback URLs, though HTTP might work in some cases.
    
    - **Development:** Use ngrok (free) or localtunnel (free) - both provide HTTPS automatically
    - **Production:** Use your own domain with SSL certificate
    
    ESP providers typically prefer HTTPS for security reasons.

---

## Installation by Operating System

### Windows Installation

#### Option 1: Automatic Installation (Recommended)

1. Extract the SDK package
2. Right-click `install-prerequisites.bat` → **Run as administrator**
3. Wait for installation to complete (2-5 minutes)
4. **Close terminal and open a new one** (important for PATH updates)
5. Verify installation:

```cmd
java -version
mvn -version
ngrok version
```

#### Option 2: Manual Installation

**Step 1: Install Chocolatey (Package Manager)**

Open PowerShell as Administrator:

```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

**Step 2: Install Java, Maven, ngrok**

```cmd
choco install temurin17 -y
choco install maven -y
choco install ngrok -y
```

**Step 3: Restart terminal and verify**

```cmd
java -version
mvn -version
ngrok version
```

#### Option 3: Manual Download

| Software | Download URL |
|----------|-------------|
| Java 17 | https://adoptium.net/temurin/releases/?version=17 |
| Maven | https://maven.apache.org/download.cgi |
| ngrok | https://ngrok.com/download |

After downloading:

1. Install Java using the `.msi` installer
2. Extract Maven to `C:\Program Files\Maven`
3. Extract ngrok to `C:\Program Files\ngrok`
4. Add to System PATH:
   - `C:\Program Files\Maven\bin`
   - `C:\Program Files\ngrok`

---

### Linux Installation

#### Ubuntu/Debian

**Automatic:**
```bash
chmod +x install-prerequisites.sh
./install-prerequisites.sh
```

**Manual:**
```bash
# Update package list
sudo apt update

# Install Java 17
sudo apt install -y openjdk-17-jdk

# Install Maven
sudo apt install -y maven

# Install ngrok
curl -s https://ngrok-agent.s3.amazonaws.com/ngrok.asc | sudo tee /etc/apt/trusted.gpg.d/ngrok.asc >/dev/null
echo "deb https://ngrok-agent.s3.amazonaws.com buster main" | sudo tee /etc/apt/sources.list.d/ngrok.list
sudo apt update && sudo apt install -y ngrok

# Verify installation
java -version
mvn -version
ngrok version
```

#### CentOS/RHEL/Fedora

```bash
# Install Java 17
sudo yum install -y java-17-openjdk java-17-openjdk-devel

# Install Maven
sudo yum install -y maven

# Install ngrok
curl -s https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-linux-amd64.tgz | sudo tar xzf - -C /usr/local/bin

# Verify
java -version
mvn -version
ngrok version
```

---

### Mac Installation

#### Automatic:
```bash
chmod +x install-prerequisites.sh
./install-prerequisites.sh
```

#### Manual (Using Homebrew):

**Step 1: Install Homebrew (if not installed)**
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

**Step 2: Install Java, Maven, ngrok**
```bash
# Install Java 17
brew install openjdk@17

# Add Java to PATH
echo 'export PATH="/opt/homebrew/opt/openjdk@17/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

# Install Maven
brew install maven

# Install ngrok
brew install ngrok/ngrok/ngrok

# Verify
java -version
mvn -version
ngrok version
```

---

## Configuration

### What Needs to Be Configured

| Configuration | File Location | When to Update |
|---------------|---------------|----------------|
| **License files** | `config/` folder | Once (replace with your files) |
| **Capricorn credentials** | `application.properties` | Once (add your credentials) |
| **Public URL** | `application.properties` | Every time ngrok URL changes |
| **API credentials** | `application.properties` | Once (set your own) |

### File Locations

```
esign-sdk-complete/
├── esign-api/
│   ├── config/
│   │   ├── eSignLicense          ← Replace with YOUR license
│   │   └── privatekey.pfx        ← Replace with YOUR certificate
│   └── src/main/resources/
│       └── application.properties ← Update credentials & URLs
```

### Step 1: Replace License Files

Copy your Capricorn license files:

=== "Windows"

    ```cmd
    copy "C:\path\to\your\eSignLicense" "esign-api\config\"
    copy "C:\path\to\your\privatekey.pfx" "esign-api\config\"
    ```

=== "Linux/Mac"

    ```bash
    cp /path/to/your/eSignLicense esign-api/config/
    cp /path/to/your/privatekey.pfx esign-api/config/
    ```

!!! note "No Rebuild Required"
    Replacing license files does NOT require rebuilding. Just restart the service.

### Step 2: Configure application.properties

Open `esign-api/src/main/resources/application.properties`:

```properties
# ========================================
# Server Configuration (usually no change needed)
# ========================================
server.port=8081

# ========================================
# YOUR PUBLIC URL
# Update this with your ngrok URL or domain
# ========================================
api.base-url=https://YOUR-NGROK-URL.ngrok-free.dev

# ========================================
# API Authentication (YOU CREATE THESE)
# Use any secure random strings
# ========================================
api.auth.token=YOUR_SECURE_TOKEN
api.auth.key=YOUR_SECURE_KEY

# ========================================
# ESP Configuration (FROM CAPRICORN)
# ========================================
esign.2_1.esp.url=https://demo.esign.digital/esign/2.1/signdoc/
esign.3_2.esp.url=https://demo.esign.digital/esign/3.2/signdoc/

# ========================================
# Your Credentials (FROM CAPRICORN)
# ========================================
esign.asp.id=YOUR_ASP_ID_FROM_CAPRICORN
esign.3_2.signer.id=youruser@yourcompany.capricorn

# ========================================
# Certificate Configuration (FROM CAPRICORN)
# ========================================
esign.license.path=./config/eSignLicense
esign.certificate.path=./config/privatekey.pfx
esign.certificate.password=YOUR_CERTIFICATE_PASSWORD
```

### Configuration Summary Table

| Property | Value Source | Example |
|----------|--------------|---------|
| `api.base-url` | Your ngrok/domain URL | `https://abc123.ngrok-free.dev` |
| `api.auth.token` | You create it | `MY_TOKEN_12345` |
| `api.auth.key` | You create it | `my_key_abcdef` |
| `esign.2_1.esp.url` | From Capricorn | `https://demo.esign.digital/...` |
| `esign.3_2.esp.url` | From Capricorn | `https://demo.esign.digital/...` |
| `esign.asp.id` | From Capricorn | `yourcompanyaspid` |
| `esign.3_2.signer.id` | From Capricorn | `user@company.capricorn` |
| `esign.certificate.password` | From Capricorn | `your_password` |

### Which Changes Require Rebuild?

| Change Type | Rebuild Required? | Action |
|-------------|-------------------|--------|
| License files replaced | ❌ No | Restart only |
| `application.properties` changed | ✅ Yes | Rebuild + Restart |
| Java source code changed | ✅ Yes | Rebuild + Restart |
| `pom.xml` changed | ✅ Yes | Rebuild + Restart |

---

## Build & Deploy

### Automatic Build

=== "Windows"

    ```cmd
    cd esign-sdk-complete
    build.bat
    ```

=== "Linux/Mac"

    ```bash
    cd esign-sdk-complete
    chmod +x build.sh
    ./build.sh
    ```

### Manual Build

```bash
# Navigate to esign-api folder
cd esign-api

# Clean and build
mvn clean package -DskipTests

# JAR file created at:
# esign-api/target/esign-api-1.0.0.jar
```

### Build Output

After successful build:
```
esign-api/target/
└── esign-api-1.0.0.jar    ← Executable JAR file
```

---

## Starting & Stopping Services

### Starting Services

=== "Windows - Automatic"

    ```cmd
    cd esign-sdk-complete
    start.bat
    ```

=== "Windows - Manual"

    ```cmd
    cd esign-api
    mvn spring-boot:run
    ```

=== "Linux/Mac - Automatic"

    ```bash
    cd esign-sdk-complete
    ./start.sh
    ```

=== "Linux/Mac - Manual"

    ```bash
    cd esign-api
    mvn spring-boot:run
    ```

### Stopping Services

=== "Windows"

    - Close the server terminal windows, OR
    - Press `Ctrl+C` in the terminal, OR
    - Run: `taskkill /F /IM java.exe`

=== "Linux/Mac"

    ```bash
    ./stop.sh
    ```
    
    Or press `Ctrl+C` in the terminal, or:
    
    ```bash
    pkill -f "esign-api"
    ```

### Verify Service Running

```bash
# Health check
curl http://localhost:8081/api/v1/esign/health

# Expected response:
# {"status":"UP"}
```

---

## Testing with ngrok

### What is ngrok?

ngrok creates a public HTTPS URL that tunnels to your local server. This is needed because ESP callbacks require a publicly accessible URL.

### Step-by-Step ngrok Setup

**Step 1: Start your service first**
```bash
cd esign-api
mvn spring-boot:run
```

**Step 2: Open a NEW terminal and start ngrok**
```bash
ngrok http 8081
```

**Step 3: Copy the HTTPS URL**

You'll see output like:
```
Forwarding    https://abc123xyz.ngrok-free.dev -> http://localhost:8081
```

Copy: `https://abc123xyz.ngrok-free.dev`

**Step 4: Update application.properties**

Open `esign-api/src/main/resources/application.properties` and update:

```properties
api.base-url=https://abc123xyz.ngrok-free.dev
```

**Step 5: Rebuild and Restart**

```bash
# Stop the running service (Ctrl+C)
# Rebuild
mvn clean package -DskipTests

# Restart
mvn spring-boot:run
```

**Step 6: Test**

Open browser: `https://abc123xyz.ngrok-free.dev/api/v1/esign/health`

Should show: `{"status":"UP"}`

### ngrok URL Update Workflow

Every time ngrok restarts, you get a new URL. Follow this process:

```
1. Start ngrok → Get new URL
2. Update api.base-url in application.properties
3. Rebuild: mvn clean package -DskipTests
4. Restart: mvn spring-boot:run
```

### ngrok Free Tier Limitations

| Limitation | Workaround |
|------------|------------|
| URL changes on restart | Update config and rebuild |
| Only 1 tunnel at a time | Use localtunnel for second port |
| Browser warning page | SDK handles this automatically |

---

## Production Deployment

### Using Your Own Domain

1. **Get a domain name** from any registrar
2. **Get SSL certificate** (Let's Encrypt is free)
3. **Set up reverse proxy** (nginx recommended)

**Example nginx configuration:**

```nginx
server {
    listen 443 ssl;
    server_name api.yourdomain.com;

    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

    location / {
        proxy_pass http://localhost:8081;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Production Checklist

- [ ] SSL certificate installed
- [ ] Firewall configured
- [ ] Production ESP URLs configured
- [ ] API credentials secured
- [ ] Logging configured
- [ ] Backup strategy in place
- [ ] Monitoring set up

---

## Common Errors & Solutions

### Error: Port Already in Use

```
Web server failed to start. Port 8081 was already in use.
```

**Solution:**

=== "Windows"

    ```cmd
    # Find process using port
    netstat -ano | findstr :8081
    
    # Kill process (replace 12345 with actual PID)
    taskkill /PID 12345 /F
    
    # Or kill all Java processes
    taskkill /F /IM java.exe
    ```

=== "Linux/Mac"

    ```bash
    # Find process using port
    lsof -i :8081
    
    # Kill process (replace 12345 with actual PID)
    kill -9 12345
    ```

---

### Error: Java Not Found

```
'java' is not recognized as an internal or external command
```

**Solution:**

1. Verify Java is installed - run the install script again
2. Close terminal and open a new one (PATH needs refresh)
3. If still not working, manually add Java to PATH

---

### Error: Maven Not Found

```
'mvn' is not recognized as an internal or external command
```

**Solution:**

1. Verify Maven is installed - run the install script again
2. Close terminal and open a new one (PATH needs refresh)
3. If still not working, manually add Maven bin folder to PATH

---

### Error: License Not Found

```
License file not found: ./config/eSignLicense
```

**Solution:**

1. Check file exists: `dir config\` (Windows) or `ls config/` (Linux/Mac)
2. Ensure filename is exactly `eSignLicense` (case-sensitive on Linux/Mac)
3. Verify path in application.properties matches actual location

---

### Error: Certificate Password Wrong

```
keystore password was incorrect
```

**Solution:**

1. Verify `esign.certificate.password` in application.properties
2. Ensure password matches what Capricorn provided (case-sensitive)
3. No extra spaces before or after the password

---

### Error: CORS Blocked

```
Access to fetch has been blocked by CORS policy
```

**Solution:**

Ensure `CorsConfig.java` exists in `esign-api/src/main/java/com/capricorn/esign/api/config/`:

```java
package com.capricorn.esign.api.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.web.cors.CorsConfiguration;
import org.springframework.web.cors.UrlBasedCorsConfigurationSource;
import org.springframework.web.filter.CorsFilter;

@Configuration
public class CorsConfig {
    @Bean
    public CorsFilter corsFilter() {
        CorsConfiguration config = new CorsConfiguration();
        config.addAllowedOrigin("*");
        config.addAllowedMethod("*");
        config.addAllowedHeader("*");
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", config);
        return new CorsFilter(source);
    }
}
```

After adding, rebuild: `mvn clean package -DskipTests`

---

### Error: ngrok HTML Response Instead of JSON

```
Unexpected token '<', received HTML instead of JSON
```

**Solution:**

The SDK already includes the `ngrok-skip-browser-warning` header. If still occurring:

1. Clear browser cache
2. Try in incognito/private window
3. Use paid ngrok plan (no warning page)

---

### Error: Build Failed - No Main Manifest

```
no main manifest attribute, in target/br-esign-sdk-1.0.0.jar
```

**Solution:**

Update `pom.xml` build section:

```xml
<build>
    <plugins>
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
    </plugins>
</build>
```

Then rebuild: `mvn clean package -DskipTests`

---

### Error: ESP Callback Failed

```
ESP callback URL not reachable
```

**Solution:**

1. Verify ngrok is running
2. Check ngrok URL hasn't changed
3. Update `api.base-url` with current ngrok URL
4. Rebuild and restart

---

### Error: Connection Refused

```
Connection refused: localhost:8081
```

**Solution:**

1. Check if service is running: `curl http://localhost:8081/api/v1/esign/health`
2. Start the service if not running
3. Check if port is correct in application.properties

---

## Rebuilding After Configuration Changes

### Quick Reference: When to Rebuild

| What Changed | Rebuild? | Command |
|--------------|----------|---------|
| License files only | ❌ No | Just restart |
| application.properties | ✅ Yes | Rebuild + Restart |
| Java code | ✅ Yes | Rebuild + Restart |
| pom.xml | ✅ Yes | Rebuild + Restart |
| ngrok URL | ✅ Yes | Rebuild + Restart |

### How to Rebuild

=== "Windows - Automatic"

    ```cmd
    # Stop service first (close window or Ctrl+C)
    build.bat
    start.bat
    ```

=== "Windows - Manual"

    ```cmd
    cd esign-api
    mvn clean package -DskipTests
    mvn spring-boot:run
    ```

=== "Linux/Mac - Automatic"

    ```bash
    ./stop.sh
    ./build.sh
    ./start.sh
    ```

=== "Linux/Mac - Manual"

    ```bash
    cd esign-api
    mvn clean package -DskipTests
    mvn spring-boot:run
    ```

### Complete Rebuild Workflow

```bash
# 1. Stop running service
#    (Ctrl+C or close terminal)

# 2. Make your changes
#    (edit application.properties, etc.)

# 3. Rebuild
mvn clean package -DskipTests

# 4. Start again
mvn spring-boot:run

# 5. Verify
curl http://localhost:8081/api/v1/esign/health
```

---

## Quick Reference

### Commands Summary

| Task | Windows | Linux/Mac |
|------|---------|-----------|
| Install prerequisites | `install-prerequisites.bat` (Admin) | `./install-prerequisites.sh` |
| Check prerequisites | `check-prerequisites.bat` | `./check-prerequisites.sh` |
| Build | `build.bat` | `./build.sh` |
| Start services | `start.bat` | `./start.sh` |
| Stop services | Close windows / `taskkill /F /IM java.exe` | `./stop.sh` |
| Start ngrok | `ngrok http 8081` | `ngrok http 8081` |
| Manual build | `mvn clean package -DskipTests` | `mvn clean package -DskipTests` |
| Manual run | `mvn spring-boot:run` | `mvn spring-boot:run` |

### URLs

| Service | URL |
|---------|-----|
| esign-api | http://localhost:8081 |
| Health check | http://localhost:8081/api/v1/esign/health |
| JS SDK test page | http://localhost:8085/test.html |

### Important Files

| File | Purpose |
|------|---------|
| `esign-api/src/main/resources/application.properties` | Main configuration |
| `esign-api/config/eSignLicense` | License file |
| `esign-api/config/privatekey.pfx` | Certificate file |

### Support

- **Capricorn Support:** Contact Capricorn Technologies for license and ESP credential issues
- **Technical Issues:** Check the [Common Errors & Solutions](#common-errors--solutions) section above

---

**Version:** 1.0.0 | **Last Updated:** January 2026
