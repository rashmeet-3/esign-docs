# eSign SDK v1.0.0 - Deployment Checklist

**Production Deployment Checklist**

---

## Pre-Deployment Checklist

### Infrastructure Requirements

- [ ] **Server provisioned** with minimum 4GB RAM, 2 CPU cores
- [ ] **Java 17 JDK** installed and verified
- [ ] **Firewall configured** to allow port 8080 (or custom port)
- [ ] **SSL/TLS certificate** obtained for HTTPS
- [ ] **Reverse proxy** configured (Nginx/Apache) for SSL termination
- [ ] **Domain name** configured and DNS pointing to server
- [ ] **Backup storage** configured for signed documents

### Security Requirements

- [ ] **eSign license file** obtained and validated
- [ ] **ASP certificate (.pfx)** obtained and password secured
- [ ] **Certificate expiry** checked (minimum 6 months validity)
- [ ] **Strong password** set for certificate (12+ characters)
- [ ] **File permissions** restricted on config directory
- [ ] **Firewall rules** configured (allow only necessary ports)
- [ ] **SELinux/AppArmor** configured (if applicable)

### Configuration

- [ ] **application.properties** configured for production
  - [ ] Production ESP URLs set
  - [ ] Public callback URLs configured
  - [ ] ASP ID and credentials set
  - [ ] File paths configured
  - [ ] Upload limits set appropriately
- [ ] **Logging** configured
  - [ ] Log level set to INFO or WARN
  - [ ] Log rotation configured
  - [ ] Log directory with sufficient space
- [ ] **Session timeout** configured appropriately
- [ ] **CORS settings** configured for production domains

### Testing

- [ ] **Application starts** successfully
- [ ] **Configuration validated** (license, certificate)
- [ ] **API endpoints** tested with curl/Postman
- [ ] **End-to-end signing** tested in staging
- [ ] **ESP callbacks** tested and verified
- [ ] **Error handling** tested
- [ ] **Load testing** performed (if high volume expected)

---

## Deployment Steps

### Linux Deployment

#### Step 1: Create Installation Directory
```bash
sudo mkdir -p /opt/esign-sdk
sudo mkdir -p /opt/esign-sdk/config
sudo mkdir -p /opt/esign-sdk/temp
sudo mkdir -p /opt/esign-sdk/signed
sudo mkdir -p /opt/esign-sdk/logs
```

#### Step 2: Copy Files
```bash
sudo cp esign-sdk-1.0.jar /opt/esign-sdk/
sudo cp application.properties /opt/esign-sdk/
sudo cp config/eSignLicense /opt/esign-sdk/config/
sudo cp config/privatekey.pfx /opt/esign-sdk/config/
```

#### Step 3: Set Permissions
```bash
sudo useradd -r -s /bin/false esign
sudo chown -R esign:esign /opt/esign-sdk
sudo chmod 600 /opt/esign-sdk/config/privatekey.pfx
sudo chmod 644 /opt/esign-sdk/config/eSignLicense
```

#### Step 4: Create Systemd Service
```bash
sudo nano /etc/systemd/system/esign-sdk.service
```

```ini
[Unit]
Description=eSign SDK Service
After=network.target

[Service]
Type=simple
User=esign
WorkingDirectory=/opt/esign-sdk
ExecStart=/usr/bin/java -Xms2G -Xmx4G -jar /opt/esign-sdk/esign-sdk-1.0.jar
Restart=on-failure
RestartSec=10
StandardOutput=append:/opt/esign-sdk/logs/esign-sdk.log
StandardError=append:/opt/esign-sdk/logs/esign-sdk-error.log

[Install]
WantedBy=multi-user.target
```

#### Step 5: Enable and Start Service
```bash
sudo systemctl daemon-reload
sudo systemctl enable esign-sdk
sudo systemctl start esign-sdk
sudo systemctl status esign-sdk
```

#### Step 6: Configure Firewall
```bash
# UFW (Ubuntu/Debian)
sudo ufw allow 8080/tcp
sudo ufw reload

# firewalld (CentOS/RHEL)
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --reload
```

#### Step 7: Configure Nginx Reverse Proxy
```bash
sudo nano /etc/nginx/sites-available/esign-sdk
```

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
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/esign-sdk /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

### Windows Deployment

#### Step 1: Create Installation Directory
```powershell
mkdir C:\esign-sdk
mkdir C:\esign-sdk\config
mkdir C:\esign-sdk\temp
mkdir C:\esign-sdk\signed
mkdir C:\esign-sdk\logs
```

#### Step 2: Copy Files
```powershell
Copy-Item esign-sdk-1.0.jar C:\esign-sdk\
Copy-Item application.properties C:\esign-sdk\
Copy-Item config\eSignLicense C:\esign-sdk\config\
Copy-Item config\privatekey.pfx C:\esign-sdk\config\
```

#### Step 3: Create Windows Service
Create `install-service.bat`:
```batch
@echo off
sc create "eSignSDK" binPath= "C:\esign-sdk\run.bat" start= auto
sc description "eSignSDK" "eSign Digital Signature Service"
sc start "eSignSDK"
```

Create `run.bat`:
```batch
@echo off
cd C:\esign-sdk
java -Xms2G -Xmx4G -jar esign-sdk-1.0.jar >> logs\esign-sdk.log 2>&1
```

#### Step 4: Configure Firewall
```powershell
netsh advfirewall firewall add rule name="eSign SDK" dir=in action=allow protocol=TCP localport=8080
```

---

## Post-Deployment Checklist

### Verification

- [ ] **Service running** and auto-starts on reboot
- [ ] **Web interface** accessible at https://yourdomain.com
- [ ] **API endpoints** responding correctly
- [ ] **SSL certificate** valid and trusted
- [ ] **ESP callbacks** working (test with real signing)
- [ ] **Logs** being written correctly
- [ ] **File permissions** correct and secure

### Monitoring Setup

- [ ] **Application logs** monitored
  ```bash
  tail -f /opt/esign-sdk/logs/esign-sdk.log
  ```
- [ ] **System resources** monitored (CPU, RAM, disk)
- [ ] **Uptime monitoring** configured (e.g., UptimeRobot, Pingdom)
- [ ] **Error alerting** configured
- [ ] **Disk space alerts** configured for temp/signed directories

### Backup Configuration

- [ ] **Configuration backup** automated
  ```bash
  # Daily backup script
  #!/bin/bash
  tar -czf /backup/esign-config-$(date +%Y%m%d).tar.gz \
    /opt/esign-sdk/config/ \
    /opt/esign-sdk/application.properties
  ```
- [ ] **Signed documents backup** configured
  ```bash
  # Backup signed PDFs
  rsync -av /opt/esign-sdk/signed/ /backup/signed/
  ```
- [ ] **Database backup** (if using external DB)
- [ ] **Backup retention policy** defined

### Maintenance Tasks

- [ ] **Log rotation** configured
  ```bash
  # /etc/logrotate.d/esign-sdk
  /opt/esign-sdk/logs/*.log {
      daily
      rotate 30
      compress
      delaycompress
      notifempty
      create 0640 esign esign
  }
  ```
- [ ] **Temp file cleanup** scheduled
  ```bash
  # Cron job: daily at 2 AM
  0 2 * * * find /opt/esign-sdk/temp -type f -mtime +7 -delete
  ```
- [ ] **Certificate expiry monitoring** configured
- [ ] **Update procedure** documented

---

## Security Hardening

### Application Security

- [ ] **HTTPS only** (HTTP redirects to HTTPS)
- [ ] **Strong SSL/TLS** configuration (TLS 1.2+)
- [ ] **CORS** restricted to specific domains
- [ ] **Session cookies** marked as secure and httpOnly
- [ ] **File upload** size limits enforced
- [ ] **Input validation** enabled

### System Security

- [ ] **OS updates** applied
- [ ] **Firewall** configured (only necessary ports open)
- [ ] **SSH** hardened (key-based auth, no root login)
- [ ] **Fail2ban** configured (if applicable)
- [ ] **SELinux/AppArmor** enabled and configured
- [ ] **Antivirus** installed and updated (if applicable)

### Access Control

- [ ] **Service user** has minimal permissions
- [ ] **Config files** readable only by service user
- [ ] **Certificate files** protected (chmod 600)
- [ ] **Admin access** restricted to specific IPs
- [ ] **SSH access** restricted to specific users

---

## Performance Optimization

### JVM Tuning

- [ ] **Heap size** configured appropriately
  ```bash
  -Xms2G -Xmx4G
  ```
- [ ] **Garbage collector** optimized
  ```bash
  -XX:+UseG1GC -XX:MaxGCPauseMillis=200
  ```
- [ ] **JVM monitoring** enabled (JMX)

### Application Tuning

- [ ] **Connection pool** sized appropriately
- [ ] **Thread pool** configured
- [ ] **Session timeout** optimized
- [ ] **File cleanup** automated

---

## Disaster Recovery

### Backup Strategy

- [ ] **Daily backups** of configuration
- [ ] **Weekly backups** of signed documents
- [ ] **Monthly backups** archived offsite
- [ ] **Backup restoration** tested

### Recovery Procedures

- [ ] **Recovery plan** documented
- [ ] **Recovery time objective (RTO)** defined
- [ ] **Recovery point objective (RPO)** defined
- [ ] **Failover procedure** documented (if HA setup)

---

## Go-Live Checklist

### Final Verification

- [ ] **All tests passed** in production environment
- [ ] **Monitoring active** and alerts configured
- [ ] **Backups verified** and restoration tested
- [ ] **Documentation complete** and accessible
- [ ] **Support team trained** on the system
- [ ] **Escalation procedures** defined

### Communication

- [ ] **Stakeholders notified** of go-live date
- [ ] **Users informed** of new system
- [ ] **Support contacts** published
- [ ] **Maintenance windows** scheduled and communicated

### Post-Go-Live

- [ ] **Monitor closely** for first 24-48 hours
- [ ] **Review logs** daily for first week
- [ ] **Collect user feedback**
- [ ] **Performance metrics** tracked
- [ ] **Issues documented** and resolved

---

## Rollback Plan

In case of critical issues:

1. **Stop service**
   ```bash
   sudo systemctl stop esign-sdk
   ```

2. **Restore previous version**
   ```bash
   sudo cp /backup/esign-sdk-2.0.0.jar /opt/esign-sdk/esign-sdk-1.0.jar
   ```

3. **Restore configuration**
   ```bash
   sudo tar -xzf /backup/esign-config-backup.tar.gz -C /
   ```

4. **Restart service**
   ```bash
   sudo systemctl start esign-sdk
   ```

5. **Verify functionality**

6. **Notify stakeholders**

---

## Support Contacts

**Technical Support:**
- Email: support@www.esign.network
- Phone: +91-XXX-XXX-XXXX
- Emergency: +91-XXX-XXX-XXXX (24/7)

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

© 2025 Capricorn Technologies
