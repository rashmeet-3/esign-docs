# Complete Installation & Deployment Guide

This guide provides step-by-step instructions for setting up the eSign SDK on Windows, Linux, and Mac systems.

---

## Table of Contents

1. [System Architecture](#system-architecture)
2. [What You Get from Capricorn (ESP)](#what-you-get-from-capricorn-esp)
3. [Prerequisites](#prerequisites)
4. [Installation by Operating System](#installation-by-operating-system)
5. [Configuration](#configuration)
6. [Build & Deploy](#build--deploy)
7. [Starting & Stopping Services](#starting--stopping-services)
8. [Testing with ngrok](#testing-with-ngrok)
9. [Production Deployment](#production-deployment)
10. [Common Errors & Solutions](#common-errors--solutions)
11. [Rebuilding After Configuration Changes](#rebuilding-after-configuration-changes)
12. [Quick Reference](#quick-reference)

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           eSign SDK Architecture                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌─────────────┐      ┌─────────────┐      ┌─────────────┐                 │
│   │ JavaScript  │      │             │      │             │                 │
│   │    SDK      │─────▶│  esign-api  │─────▶│   ESP       │                 │
│   │ (Browser)   │      │ (Port 8081) │      │ (Capricorn) │────▶ UIDAI     │
│   ├─────────────┤      │             │      │             │                 │
│   │  Android    │─────▶│   REST API  │─────▶│   eSign     │                 │
│   │    SDK      │      │   + SDK     │      │   Gateway   │                 │
│   │  (Mobile)   │      │             │      │             │                 │
│   └─────────────┘      └─────────────┘      └─────────────┘                 │
│                              │                     │                         │
│                              │◀── HTTPS Callback ──┘                         │
│                              │                                               │
│                        ┌─────▼─────┐                                         │
│                        │  ngrok /  │                                         │
│                        │ Your URL  │                                         │
│                        │  (HTTPS)  │                                         │
│                        └───────────┘                                         │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Components

| Component | Port | Description |
|-----------|------|-------------|
| **esign-api** | 8081 | REST API for JavaScript/Android SDKs |
| **tomcat_esign** | 8080 | Core Java SDK with web interface (optional) |
| **ngrok** | - | Provides HTTPS URL for local testing |

---

## What You Get from Capricorn (ESP)

Before starting, you must obtain the following from **Capricorn Technologies**:

### Required Files

| File | Description | Where to Place |
|------|-------------|----------------|
| `eSignLicense` | License file for SDK activation | `config/eSignLicense` |
| `privatekey.pfx` | Digital certificate for signing | `config/privatekey.pfx` |

### Required Credentials

| Credential | Description | Example |
|------------|-------------|---------|
| **ASP ID** | Application Service Provider ID | `yourcompanyaspid` |
| **Signer ID** | eKYC signer identifier | `user@yourcompany.capricorn` |
| **Certificate Password** | Password for privatekey.pfx | `your_password` |
| **ESP URL (2.1)** | OTP-based signing endpoint | `https://esign.capricorn.com/esign/2.1/signdoc/` |
| **ESP URL (3.2)** | eKYC-based signing endpoint | `https://esign.capricorn.com/esign/3.2/signdoc/` |
| **API Token** | Authentication token | `9EC4709C8A5F10F0D56A72AB5EC` |
| **API Key** | Authentication key | `c1d6e4fc742265650...` |

!!! warning "Important"
    Keep your license files and credentials secure. Never commit them to public repositories.

### How to Get Credentials

1. Contact Capricorn Technologies: [support@capricorn.com](mailto:support@capricorn.com)
2. Complete the ESP registration process
3. Sign the agreement and pay applicable fees
4. Receive license files and credentials via secure channel

---

## Prerequisites

### Required Software

| Software | Minimum Version | Recommended Version | Purpose |
|----------|-----------------|---------------------|---------|
| **Java JDK** | 17 | 17 or 21 | Runtime and compilation |
| **Apache Maven** | 3.6.0 | 3.9.x | Build tool |
| **ngrok** | Any | Latest | HTTPS tunneling (for testing) |
| **Python** | 3.x | 3.10+ | Test page server (optional) |
| **Node.js** | 14.x | 18.x+ | For localtunnel (optional) |

### Version Check Commands

```bash
java -version      # Should show 17 or higher
mvn -version       # Should show 3.6.0 or higher
ngrok version      # Any version
python --version   # 3.x (optional)
node --version     # 14.x or higher (optional)
```

### HTTPS Requirement

!!! note "HTTPS is Required"
    ESP (Capricorn/UIDAI) requires **HTTPS** callback URLs for security. Options:
    
    - **Development:** Use ngrok (free) or localtunnel (free)
    - **Production:** Use your own domain with SSL certificate

---

## Installation by Operating System

### Windows Installation

#### Method 1: Automatic Installation (Recommended)

1. Extract the SDK package
2. Right-click `install-prerequisites.bat` → **Run as administrator**
3. Wait for installation to complete
4. **Close terminal and open a new one**

```cmd
# Verify installation
java -version
mvn -version
ngrok version
```

#### Method 2: Manual Installation

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

#### Method 3: Manual Download

| Software | Download URL |
|----------|-------------|
| Java 17 | https://adoptium.net/temurin/releases/?version=17 |
| Maven | https://maven.apache.org/download.cgi |
| ngrok | https://ngrok.com/download |

After downloading:

1. Install Java using the `.msi` installer
2. Extract Maven to `C:\Program Files\Maven`
3. Extract ngrok to `C:\Program Files\ngrok`
4. Add to PATH:
   - `C:\Program Files\Maven\bin`
   - `C:\Program Files\ngrok`

---

### Linux Installation

#### Ubuntu/Debian

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

# Verify installation
java -version
mvn -version
ngrok version
```

#### Arch Linux

```bash
# Install Java 17
sudo pacman -S --noconfirm jdk17-openjdk

# Install Maven
sudo pacman -S --noconfirm maven

# Install ngrok (from AUR or manual)
curl -s https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-linux-amd64.tgz | sudo tar xzf - -C /usr/local/bin

# Verify installation
java -version
mvn -version
ngrok version
```

#### Automatic Installation (Linux)

```bash
chmod +x install-prerequisites.sh
./install-prerequisites.sh
```

---

### Mac Installation

#### Using Homebrew (Recommended)

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

# Verify installation
java -version
mvn -version
ngrok version
```

#### Automatic Installation (Mac)

```bash
chmod +x install-prerequisites.sh
./install-prerequisites.sh
```

---

## Configuration

### File Locations

```
esign-sdk-complete/
├── esign-api/
│   ├── config/
│   │   ├── eSignLicense          ← Replace with your license
│   │   └── privatekey.pfx        ← Replace with your certificate
│   └── src/main/resources/
│       └── application.properties ← Update credentials
│
├── tomcat_esign/
│   ├── config/
│   │   ├── eSignLicense          ← Replace with your license
│   │   └── privatekey.pfx        ← Replace with your certificate
│   └── src/main/resources/
│       └── application.properties ← Update credentials
```

### Step 1: Replace License Files

Copy your Capricorn license files to both projects:

=== "Windows"

    ```cmd
    copy "C:\path\to\your\eSignLicense" "esign-api\config\"
    copy "C:\path\to\your\privatekey.pfx" "esign-api\config\"
    copy "C:\path\to\your\eSignLicense" "tomcat_esign\config\"
    copy "C:\path\to\your\privatekey.pfx" "tomcat_esign\config\"
    ```

=== "Linux/Mac"

    ```bash
    cp /path/to/your/eSignLicense esign-api/config/
    cp /path/to/your/privatekey.pfx esign-api/config/
    cp /path/to/your/eSignLicense tomcat_esign/config/
    cp /path/to/your/privatekey.pfx tomcat_esign/config/
    ```

### Step 2: Configure esign-api

Edit `esign-api/src/main/resources/application.properties`:

```properties
# ========================================
# Server Configuration
# ========================================
server.port=8081

# ========================================
# Your Public URL (HTTPS required)
# Replace with your ngrok URL or production domain
# ========================================
api.base-url=https://YOUR-URL-HERE.ngrok-free.dev

# ========================================
# API Authentication
# Set your own secure token and key
# ========================================
api.auth.token=YOUR_API_TOKEN
api.auth.key=YOUR_API_KEY

# ========================================
# ESP Configuration (from Capricorn)
# ========================================
esign.2_1.esp.url=https://YOUR-ESP-URL/esign/2.1/signdoc/
esign.3_2.esp.url=https://YOUR-ESP-URL/esign/3.2/signdoc/

# ========================================
# Your Credentials (from Capricorn)
# ========================================
esign.asp.id=YOUR_ASP_ID
esign.3_2.signer.id=youruser@yourcompany.capricorn

# ========================================
# Certificate Configuration
# ========================================
esign.license.path=./config/eSignLicense
esign.certificate.path=./config/privatekey.pfx
esign.certificate.password=YOUR_CERTIFICATE_PASSWORD

# ========================================
# Data Cleanup (optional)
# ========================================
cleanup.enabled=true
cleanup.retention-days=7
cleanup.cron=0 0 2 * * ?
```

### Step 3: Configure tomcat_esign (if using)

Edit `tomcat_esign/src/main/resources/application.properties`:

```properties
# ========================================
# Server Configuration
# ========================================
server.port=8080

# ========================================
# ESP Configuration (from Capricorn)
# ========================================
esign.2_1.esp.url=https://YOUR-ESP-URL/esign/2.1/signdoc/
esign.3_2.esp.url=https://YOUR-ESP-URL/esign/3.2/signdoc/

# ========================================
# Callback URLs (Your public HTTPS URL)
# ========================================
esign.2_1.response.url=https://YOUR-URL-HERE/api/esign/esp-response
esign.3_2.response.url=https://YOUR-URL-HERE/api/esign/3.2/getdocs/
esign.3_2.redirect.url=https://YOUR-URL-HERE/api/esign/3.2/callback/
esign.base.url=https://YOUR-URL-HERE

# ========================================
# Your Credentials (from Capricorn)
# ========================================
esign.asp.id=YOUR_ASP_ID
esign.3_2.signer.id=youruser@yourcompany.capricorn

# ========================================
# Certificate Configuration
# ========================================
esign.license.path=./config/eSignLicense
esign.certificate.path=./config/privatekey.pfx
esign.certificate.password=YOUR_CERTIFICATE_PASSWORD
```

### Configuration Checklist

- [ ] License file (`eSignLicense`) replaced in both projects
- [ ] Certificate file (`privatekey.pfx`) replaced in both projects
- [ ] `api.base-url` updated with your HTTPS URL
- [ ] `api.auth.token` and `api.auth.key` set
- [ ] `esign.asp.id` set with your ASP ID
- [ ] `esign.certificate.password` set correctly
- [ ] ESP URLs configured (get from Capricorn)
- [ ] Callback URLs use HTTPS

---

## Build & Deploy

### Build Commands

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
# Build tomcat_esign
cd tomcat_esign
mvn clean package -DskipTests
cd ..

# Build esign-api
cd esign-api
mvn clean package -DskipTests
cd ..
```

### Build Output

After successful build:

```
esign-sdk-complete/
├── esign-api/target/
│   └── esign-api-1.0.0.jar          ← esign-api executable
└── tomcat_esign/target/
    └── br-esign-sdk-1.0.0.jar       ← tomcat_esign executable
```

---

## Starting & Stopping Services

### Starting Services

=== "Windows"

    **Option 1: Using start.bat**
    ```cmd
    cd esign-sdk-complete
    start.bat
    ```
    
    **Option 2: Manual start (separate terminals)**
    ```cmd
    # Terminal 1 - esign-api
    cd esign-api
    mvn spring-boot:run
    
    # Terminal 2 - tomcat_esign (optional)
    cd tomcat_esign
    mvn spring-boot:run
    ```

=== "Linux/Mac"

    **Option 1: Using start.sh**
    ```bash
    cd esign-sdk-complete
    ./start.sh
    ```
    
    **Option 2: Manual start (separate terminals)**
    ```bash
    # Terminal 1 - esign-api
    cd esign-api
    mvn spring-boot:run
    
    # Terminal 2 - tomcat_esign (optional)
    cd tomcat_esign
    mvn spring-boot:run
    ```

### Stopping Services

=== "Windows"

    - Close the server terminal windows, or
    - Press `Ctrl+C` in each terminal, or
    - Run: `taskkill /F /IM java.exe`

=== "Linux/Mac"

    ```bash
    ./stop.sh
    ```
    
    Or press `Ctrl+C` in each terminal, or:
    
    ```bash
    pkill -f "esign-api"
    pkill -f "tomcat_esign"
    ```

### Verify Services Running

```bash
# esign-api health check
curl http://localhost:8081/api/v1/esign/health

# tomcat_esign (open in browser)
http://localhost:8080
```

---

## Testing with ngrok

### What is ngrok?

ngrok creates a secure HTTPS tunnel to your local server, required for ESP callbacks.

### Step 1: Start ngrok

```bash
ngrok http 8081
```

Output:
```
Forwarding    https://abc123xyz.ngrok-free.dev -> http://localhost:8081
```

### Step 2: Copy the HTTPS URL

Copy the URL like `https://abc123xyz.ngrok-free.dev`

### Step 3: Update Configuration

Update `api.base-url` in `esign-api/src/main/resources/application.properties`:

```properties
api.base-url=https://abc123xyz.ngrok-free.dev
```

### Step 4: Rebuild and Restart

```bash
# Stop services, rebuild, restart
mvn clean package -DskipTests
mvn spring-boot:run
```

### ngrok Free Tier Limitations

| Limitation | Solution |
|------------|----------|
| URL changes on restart | Update config and rebuild |
| Only 1 tunnel at a time | Use localtunnel for second port |
| Rate limited | Upgrade to paid plan for production |
| Browser warning page | SDK handles this automatically |

### Alternative: localtunnel (Free)

```bash
# Install
npm install -g localtunnel

# Run
lt --port 8081

# Output: https://xyz789.loca.lt
```

---

## Production Deployment

### Option 1: Using Your Own Domain

1. Get a domain name
2. Get SSL certificate (Let's Encrypt is free)
3. Set up reverse proxy (nginx recommended)

**nginx configuration:**

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

### Option 2: Cloud Deployment

| Platform | Recommended Service |
|----------|---------------------|
| AWS | Elastic Beanstalk, EC2 |
| Google Cloud | App Engine, Compute Engine |
| Azure | App Service, VM |
| DigitalOcean | Droplet, App Platform |

### Production Checklist

- [ ] HTTPS enabled with valid SSL certificate
- [ ] Firewall configured (allow 443, block direct access to 8081)
- [ ] Environment variables for sensitive credentials
- [ ] Log rotation configured
- [ ] Monitoring and alerts set up
- [ ] Backup strategy in place
- [ ] Data cleanup enabled

---

## Common Errors & Solutions

### Error: Port Already in Use

```
Web server failed to start. Port 8081 was already in use.
```

**Solution:**

=== "Windows"

    ```cmd
    netstat -ano | findstr :8081
    taskkill /PID <PID> /F
    ```

=== "Linux/Mac"

    ```bash
    lsof -i :8081
    kill -9 <PID>
    ```

---

### Error: Java Not Found

```
'java' is not recognized as an internal or external command
```

**Solution:**

1. Verify Java is installed: Download from https://adoptium.net/
2. Add to PATH:
   - Windows: System Properties → Environment Variables → Add Java bin folder
   - Linux/Mac: Add `export PATH=$PATH:/path/to/java/bin` to `~/.bashrc` or `~/.zshrc`

---

### Error: Maven Not Found

```
'mvn' is not recognized as an internal or external command
```

**Solution:**

1. Verify Maven is installed
2. Add Maven bin folder to PATH

---

### Error: License Not Found

```
License file not found: ./config/eSignLicense
```

**Solution:**

1. Ensure license file exists in `config/` folder
2. Check file name is exactly `eSignLicense` (case-sensitive)
3. Verify path in `application.properties`

---

### Error: Certificate Password Wrong

```
keystore password was incorrect
```

**Solution:**

1. Verify `esign.certificate.password` in `application.properties`
2. Ensure password matches what Capricorn provided

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

Then rebuild.

---

### Error: ngrok HTML Response

```
Unexpected token '<', received HTML instead of JSON
```

**Solution:**

The SDK already includes the `ngrok-skip-browser-warning` header. If still occurring:

1. Use paid ngrok plan, or
2. Install browser extension to add the header

---

### Error: ESP Callback Failed

```
ESP callback URL not reachable
```

**Solution:**

1. Ensure ngrok is running
2. Verify callback URL uses HTTPS
3. Check ngrok URL hasn't changed
4. Update `api.base-url` and rebuild

---

### Error: Build Failed - No Main Manifest

```
no main manifest attribute, in target/br-esign-sdk-1.0.0.jar
```

**Solution:**

Update `tomcat_esign/pom.xml` build section:

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

## Rebuilding After Configuration Changes

### When to Rebuild

| Change Type | Action Required |
|-------------|-----------------|
| `application.properties` changed | Rebuild + Restart |
| License files replaced | Restart only |
| Java source code changed | Rebuild + Restart |
| pom.xml changed | Rebuild + Restart |

### Rebuild Steps

=== "Windows"

    ```cmd
    # 1. Stop services (close terminal windows or Ctrl+C)
    
    # 2. Rebuild
    build.bat
    
    # 3. Restart
    start.bat
    ```

=== "Linux/Mac"

    ```bash
    # 1. Stop services
    ./stop.sh
    
    # 2. Rebuild
    ./build.sh
    
    # 3. Restart
    ./start.sh
    ```

### Quick Restart (Without Full Rebuild)

If only `application.properties` changed and using `mvn spring-boot:run`:

```bash
# Just press Ctrl+C to stop, then run again
mvn spring-boot:run
```

---

## Quick Reference

### Terminal Commands Summary

| Task | Windows | Linux/Mac |
|------|---------|-----------|
| Install prerequisites | `install-prerequisites.bat` (Admin) | `./install-prerequisites.sh` |
| Check prerequisites | `check-prerequisites.bat` | `./check-prerequisites.sh` |
| Build | `build.bat` | `./build.sh` |
| Start services | `start.bat` | `./start.sh` |
| Stop services | Close windows / `taskkill /F /IM java.exe` | `./stop.sh` |
| Start ngrok | `ngrok http 8081` | `ngrok http 8081` |

### URLs Summary

| Service | URL |
|---------|-----|
| esign-api | http://localhost:8081 |
| esign-api health | http://localhost:8081/api/v1/esign/health |
| tomcat_esign | http://localhost:8080 |
| JS SDK test page | http://localhost:8085/test.html |

### File Locations

| File | Purpose |
|------|---------|
| `esign-api/src/main/resources/application.properties` | esign-api configuration |
| `tomcat_esign/src/main/resources/application.properties` | tomcat_esign configuration |
| `esign-api/config/eSignLicense` | License file |
| `esign-api/config/privatekey.pfx` | Certificate file |

### Support

- **Documentation:** See other pages in this documentation
- **Capricorn Support:** Contact Capricorn Technologies for license and ESP issues
- **Technical Issues:** Check [Common Errors & Solutions](#common-errors--solutions)

---

**Version:** 1.0.0 | **Last Updated:** January 2026
