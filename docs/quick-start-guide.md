# eSign SDK v1.0.0 - Quick Start Guide

**Get started with eSign SDK in 5 minutes**

---

## Prerequisites

- [ ] Java 17+ installed
- [ ] eSign license file
- [ ] ASP certificate (.pfx file)
- [ ] Internet connection

---

## Installation (5 Steps)

### Step 1: Extract Package
```bash
unzip esign-sdk-1.0-dist.zip
cd esign-sdk-1.0
```

### Step 2: Configure
Edit `application.properties`:
```properties
# Minimum required configuration
esign.asp.id=your-asp-id
esign.license.path=./config/eSignLicense
esign.certificate.path=./config/privatekey.pfx
esign.certificate.password=your-password

# For local testing with ngrok
esign.2_1.response.url=https://your-ngrok-url.ngrok-free.dev/api/esign/esp-response
```

### Step 3: Place Files
```bash
mkdir -p config
cp /path/to/eSignLicense config/
cp /path/to/privatekey.pfx config/
```

### Step 4: Run Application
```bash
java -jar esign-sdk-1.0.jar
```

### Step 5: Verify
Open browser: `http://localhost:8080`

---

## Your First Signing (Web UI)

1. **Open** `http://localhost:8080` in browser
2. **Select Mode**: Choose "eSign 2.1 (OTP-based)"
3. **Upload PDF**: Click "Choose File" and select a PDF
4. **Enter Details**:
   - Signer Name: `John Doe`
   - Document Info: `Test Document`
5. **Click** "Start Signing Process"
6. **Follow Steps** 1-8 in the UI
7. **Sign at ESP**: Enter Aadhaar and OTP
8. **Download** signed PDF

---

## Your First Signing (API)

```bash
# 1. Set mode
curl -X POST http://localhost:8080/api/esign/set-mode \
  -H "Content-Type: application/json" \
  -d '{"mode":"2.1"}'

# 2. Upload PDF
curl -X POST http://localhost:8080/api/esign/step1/upload \
  -F "file=@test.pdf"

# 3. Set signer details
curl -X POST http://localhost:8080/api/esign/step2/signer-details \
  -H "Content-Type: application/json" \
  -d '{"signerName":"John Doe","docInfo":"Test Document"}'

# 4-8. Execute steps
for i in {3..8}; do
  curl -X POST http://localhost:8080/api/esign/step$i/$(
    case $i in
      3) echo "generate-hash";;
      4) echo "convert-hex";;
      5) echo "generate-txn";;
      6) echo "construct-xml";;
      7) echo "sign-xml";;
      8) echo "generate-esp-form";;
    esac
  )
done
```

---

## Quick Troubleshooting

| Issue | Solution |
|-------|----------|
| Port 8080 in use | Change `server.port=8081` in properties |
| Java version error | Install Java 17: `sudo apt install openjdk-17-jdk` |
| License not found | Check path: `ls -la config/eSignLicense` |
| ESP callback fails | Use ngrok: `ngrok http 8080` |

---

## Next Steps

- Read full [Technical Documentation](eSign-sdk-technical-documentation.md)
- Configure for production
- Review Security Best Practices
- Contact support: support@www.esign.network

---

## Common Commands

**Start server:**
```bash
java -jar esign-sdk-1.0.jar
```

**Start with custom config:**
```bash
java -jar esign-sdk-1.0.jar --spring.config.location=./custom.properties
```

**Start with custom port:**
```bash
java -jar esign-sdk-1.0.jar --server.port=8081
```

**Check logs:**
```bash
tail -f logs/esign-sdk.log
```

---

## Support

- **Documentation**: See full technical documentation
- **Email**: support@www.esign.network
- **Website**: https://www.esign.network/esign-sdk

---

**Ready to integrate?** Check the [API Reference Card](api-reference-card.md) for quick API lookup.
