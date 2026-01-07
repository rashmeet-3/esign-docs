# eSign SDK v1.0.0 - Deployment Checklist

**Production Deployment Checklist**

---

## Legend

| Symbol | Category | Description |
|--------|----------|-------------|
| 🔴 | **MANDATORY** | Must be completed before go-live |
| 🟡 | **RECOMMENDED** | Strongly suggested for production |
| 🟢 | **OPTIONAL** | Nice to have, can be done later |

---

## Pre-Deployment Checklist

### Infrastructure Requirements

| Priority | Item |
|----------|------|
| 🔴 | Server provisioned with minimum **8GB RAM, 8 CPU cores** |
| 🔴 | Java 17 JDK installed and verified |
| 🔴 | Firewall configured to allow port **8081** (REST API) |
| 🟡 | Firewall configured to allow port **8080** (Web UI) - if using Web UI |
| 🔴 | SSL/TLS certificate obtained for HTTPS |
| 🟡 | Reverse proxy configured (Nginx/Apache) for SSL termination |
| 🔴 | Domain name configured and DNS pointing to server |
| 🟡 | Backup storage configured for signed documents |

### Files from Capricorn (MANDATORY)

| Priority | Item |
|----------|------|
| 🔴 | `eSignLicense` file received and placed in `config/` folder |
| 🔴 | `privatekey.pfx` certificate received and placed in `config/` folder |
| 🔴 | Certificate password received and noted securely |
| 🔴 | ASP ID received from Capricorn |
| 🔴 | ESP URLs received (Demo/Production) |

### Security Requirements

| Priority | Item |
|----------|------|
| 🔴 | eSign license file validated (not expired) |
| 🔴 | ASP certificate (.pfx) password secured |
| 🔴 | Certificate expiry checked (minimum 6 months validity) |
| 🟡 | Strong password set for certificate (12+ characters) |
| 🟡 | File permissions restricted on config directory |
| 🟡 | Firewall rules configured (allow only necessary ports) |
| 🟢 | SELinux/AppArmor configured |

### Configuration

| Priority | Item |
|----------|------|
| 🔴 | `application.properties` configured for production |
| 🔴 | Production ESP URLs set |
| 🔴 | Public callback URL configured (see options below) |
| 🔴 | ASP ID and credentials set |
| 🔴 | `api.auth.token` and `api.auth.key` set (your own secure values) |
| 🟡 | File paths configured |
| 🟡 | Upload limits set appropriately |
| 🟡 | Log level set to INFO or WARN |
| 🟢 | Log rotation configured |
| 🟢 | Session timeout configured |
| 🟡 | CORS settings configured for production domains |

#### Public URL Options

The ESP server needs to send callbacks to your server. Choose one option:

| Option | Use Case | Requirements |
|--------|----------|--------------|
| **ngrok** | Development, Testing | Free ngrok account, run `ngrok http 8081` |
| **Own Domain** | Development with server, Production | Domain, SSL certificate, Server with public IP |

!!! tip "Production Recommendation"
    For production, use your **own domain** with proper SSL certificate. ngrok is great for development but URLs change with each restart (unless you have paid plan).

### Testing

| Priority | Item |
|----------|------|
| 🔴 | Application starts successfully |
| 🔴 | Configuration validated (license, certificate) |
| 🔴 | Health endpoint responds: `GET /api/v1/esign/health` |
| 🔴 | End-to-end signing tested with real Aadhaar OTP |
| 🔴 | ESP callbacks tested and verified |
| 🟡 | API endpoints tested with curl/Postman |
| 🟡 | Error handling tested |
| 🟢 | Load testing performed (if high volume expected) |

---

## Quick Start (Development/Testing)

For development and testing, you can simply run:

=== "Windows"

    ```cmd
    start.bat
    ```

=== "Linux/Mac"

    ```bash
    ./start.sh
    ```

This starts the server on port 8081. No additional deployment steps needed for testing.

---

## Production Deployment Steps

!!! note "When to Use These Steps"
    The following deployment steps are **RECOMMENDED for production** environments where you need:
    
    - Auto-start on server reboot
    - Running as a system service
    - SSL/HTTPS termination
    - Proper logging and monitoring

---

### Linux Production Deployment

#### Step 1: Create Installation Directory 🔴 MANDATORY

```bash
sudo mkdir -p /opt/esign-api
sudo mkdir -p /opt/esign-api/config
sudo mkdir -p /opt/esign-api/data/temp
sudo mkdir -p /opt/esign-api/data/signed
sudo mkdir -p /opt/esign-api/data/uploads
sudo mkdir -p /opt/esign-api/data/transactions
sudo mkdir -p /opt/esign-api/logs
```

#### Step 2: Copy Files 🔴 MANDATORY

```bash
# Copy JAR file
sudo cp esign-api/target/esign-api-1.0.0.jar /opt/esign-api/

# Copy configuration
sudo cp esign-api/application.properties /opt/esign-api/

# Copy license and certificate from Capricorn
sudo cp esign-api/config/eSignLicense /opt/esign-api/config/
sudo cp esign-api/config/privatekey.pfx /opt/esign-api/config/
```

#### Step 3: Set Permissions 🟡 RECOMMENDED

```bash
# Create dedicated service user
sudo useradd -r -s /bin/false esign

# Set ownership
sudo chown -R esign:esign /opt/esign-api

# Secure certificate file
sudo chmod 600 /opt/esign-api/config/privatekey.pfx
sudo chmod 644 /opt/esign-api/config/eSignLicense
sudo chmod 640 /opt/esign-api/application.properties
```

#### Step 4: Create Systemd Service 🟡 RECOMMENDED

Create service file:
```bash
sudo nano /etc/systemd/system/esign-api.service
```

Add content:
```ini
[Unit]
Description=eSign API Service
After=network.target

[Service]
Type=simple
User=esign
WorkingDirectory=/opt/esign-api
ExecStart=/usr/bin/java -Xms4G -Xmx6G -jar /opt/esign-api/esign-api-1.0.0.jar
Restart=on-failure
RestartSec=10
StandardOutput=append:/opt/esign-api/logs/esign-api.log
StandardError=append:/opt/esign-api/logs/esign-api-error.log

[Install]
WantedBy=multi-user.target
```

#### Step 5: Enable and Start Service 🟡 RECOMMENDED

```bash
sudo systemctl daemon-reload
sudo systemctl enable esign-api
sudo systemctl start esign-api
sudo systemctl status esign-api
```

#### Step 6: Configure Firewall 🔴 MANDATORY

```bash
# UFW (Ubuntu/Debian)
sudo ufw allow 8081/tcp
sudo ufw reload

# firewalld (CentOS/RHEL)
sudo firewall-cmd --permanent --add-port=8081/tcp
sudo firewall-cmd --reload
```

#### Step 7: Configure Nginx Reverse Proxy 🟡 RECOMMENDED

Create Nginx config:
```bash
sudo nano /etc/nginx/sites-available/esign-api
```

Add content:
```nginx
server {
    listen 80;
    server_name esign.yourdomain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name esign.yourdomain.com;

    ssl_certificate /etc/ssl/certs/esign.crt;
    ssl_certificate_key /etc/ssl/private/esign.key;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    client_max_body_size 50M;

    location / {
        proxy_pass http://localhost:8081;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Enable and test:
```bash
sudo ln -s /etc/nginx/sites-available/esign-api /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

### Windows Production Deployment

#### Step 1: Create Installation Directory 🔴 MANDATORY

```powershell
mkdir C:\esign-api
mkdir C:\esign-api\config
mkdir C:\esign-api\data\temp
mkdir C:\esign-api\data\signed
mkdir C:\esign-api\data\uploads
mkdir C:\esign-api\data\transactions
mkdir C:\esign-api\logs
```

#### Step 2: Copy Files 🔴 MANDATORY

```powershell
# Copy JAR file
Copy-Item esign-api\target\esign-api-1.0.0.jar C:\esign-api\

# Copy configuration
Copy-Item esign-api\application.properties C:\esign-api\

# Copy license and certificate from Capricorn
Copy-Item esign-api\config\eSignLicense C:\esign-api\config\
Copy-Item esign-api\config\privatekey.pfx C:\esign-api\config\
```

#### Step 3: Create Startup Script 🟡 RECOMMENDED

Create `C:\esign-api\start-esign.bat`:
```batch
@echo off
cd /d C:\esign-api
java -Xms4G -Xmx6G -jar esign-api-1.0.0.jar >> logs\esign-api.log 2>&1
```

#### Step 4: Create Windows Service 🟢 OPTIONAL

For auto-start, use NSSM (Non-Sucking Service Manager):

```powershell
# Download NSSM from https://nssm.cc/download
# Then run:
nssm install eSignAPI "C:\Program Files\Java\jdk-17\bin\java.exe" "-Xms4G -Xmx6G -jar C:\esign-api\esign-api-1.0.0.jar"
nssm set eSignAPI AppDirectory "C:\esign-api"
nssm set eSignAPI DisplayName "eSign API Service"
nssm set eSignAPI Start SERVICE_AUTO_START
nssm start eSignAPI
```

#### Step 5: Configure Firewall 🔴 MANDATORY

```powershell
netsh advfirewall firewall add rule name="eSign API" dir=in action=allow protocol=TCP localport=8081
```

---

## Post-Deployment Checklist

### Verification

| Priority | Item |
|----------|------|
| 🔴 | Service running and responding |
| 🔴 | API endpoint accessible: `https://yourdomain.com/api/v1/esign/health` |
| 🔴 | SSL certificate valid and trusted |
| 🔴 | ESP callbacks working (test with real signing) |
| 🟡 | Service auto-starts on reboot |
| 🟡 | Logs being written correctly |

### Monitoring Setup

| Priority | Item |
|----------|------|
| 🟡 | Application logs monitored |
| 🟡 | System resources monitored (CPU, RAM, disk) |
| 🟢 | Uptime monitoring configured (e.g., UptimeRobot, Pingdom) |
| 🟢 | Error alerting configured |
| 🟢 | Disk space alerts for data directories |

### Backup Configuration

| Priority | Item |
|----------|------|
| 🟡 | Configuration backup automated |
| 🟡 | Signed documents backup configured |
| 🟢 | Backup retention policy defined |
| 🟢 | Backup restoration tested |

---

## Security Hardening

### Application Security

| Priority | Item |
|----------|------|
| 🔴 | HTTPS only (HTTP redirects to HTTPS) |
| 🟡 | Strong SSL/TLS configuration (TLS 1.2+) |
| 🟡 | CORS restricted to specific domains |
| 🟢 | File upload size limits enforced |

### System Security

| Priority | Item |
|----------|------|
| 🟡 | OS updates applied |
| 🟡 | Firewall configured (only necessary ports open) |
| 🟢 | SSH hardened (key-based auth, no root login) |
| 🟢 | Fail2ban configured |

### Access Control

| Priority | Item |
|----------|------|
| 🟡 | Service user has minimal permissions |
| 🟡 | Config files readable only by service user |
| 🟡 | Certificate files protected (chmod 600) |
| 🟢 | Admin access restricted to specific IPs |

---

## Performance Optimization 🟢 OPTIONAL

### JVM Tuning

```bash
# Recommended JVM settings for 8GB RAM server
-Xms4G -Xmx6G -XX:+UseG1GC -XX:MaxGCPauseMillis=200
```

### Log Rotation

```bash
# /etc/logrotate.d/esign-api
/opt/esign-api/logs/*.log {
    daily
    rotate 30
    compress
    delaycompress
    notifempty
    create 0640 esign esign
}
```

### Temp File Cleanup

```bash
# Cron job: daily at 2 AM
0 2 * * * find /opt/esign-api/data/temp -type f -mtime +7 -delete
```

---

## Go-Live Checklist

### Final Verification

| Priority | Item |
|----------|------|
| 🔴 | All MANDATORY items completed |
| 🔴 | End-to-end signing tested in production |
| 🔴 | Monitoring active |
| 🟡 | RECOMMENDED items reviewed |
| 🟡 | Documentation accessible to team |
| 🟡 | Support contacts known |

### Post-Go-Live

| Priority | Item |
|----------|------|
| 🔴 | Monitor closely for first 24-48 hours |
| 🟡 | Review logs daily for first week |
| 🟢 | Collect user feedback |
| 🟢 | Performance metrics tracked |

---

## Rollback Plan

In case of critical issues:

```bash
# 1. Stop service
sudo systemctl stop esign-api

# 2. Restore previous version
sudo cp /backup/esign-api-previous.jar /opt/esign-api/esign-api-1.0.0.jar

# 3. Restore configuration
sudo cp /backup/application.properties /opt/esign-api/

# 4. Restart service
sudo systemctl start esign-api

# 5. Verify functionality
curl https://yourdomain.com/api/v1/esign/health
```

---

## Support Contacts

**Technical Support:**

- **Email:** support@capricornid.com
- **Website:** https://www.esign.network

**Escalation:**

- Level 1: Technical Support
- Level 2: Senior Engineer
- Level 3: Development Team

---

## Sign-Off

| Role | Name | Signature | Date |
|------|------|-----------|------|
| **System Administrator** | | | |
| **Security Officer** | | | |
| **Project Manager** | | | |
| **Business Owner** | | | |

---

**Version:** 1.0.0 | **Last Updated:** December 2025

© 2025 Capricorn Technologies
