# Changelog

All notable changes to the eSign SDK will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.0.0] - 2025-12-15

### 🎉 Initial Release

#### Added

**Core SDK (tomcat_esign)**
- eSign 2.1 support (OTP-based Aadhaar signing)
- eSign 3.2 support (eKYC-based signing)
- Multiple authentication modes:
  - `online-aadhaar-otp` - OTP to registered mobile
  - `online-aadhaar-bio` - Fingerprint authentication
  - `online-aadhaar-iris` - Iris scan authentication
  - `online-aadhaar-face` - Face authentication
  - `capricorn-ekyc-account` - Pre-verified eKYC
- Multi-page signature support (all, first, last, range, specific pages)
- Configurable signature appearance (position, size, date format)
- PDF lock modes (no lock, certified, form filling)
- Green tick and custom text in signature
- PKCS7_COMPLETE signature format with TSA timestamp
- LTV (Long Term Validation) enabled signatures
- Web UI for browser-based testing

**REST API (esign-api)**
- RESTful endpoints for SDK integration
- XML and JSON request/response support
- Transaction tracking with unique reference IDs
- Status checking endpoint
- Signed document retrieval endpoint
- Health check endpoint
- CORS support for browser-based clients
- Transaction ID uniqueness enforcement
- Configurable authentication (token + key)

**JavaScript SDK**
- Browser-compatible client library
- Promise-based async API
- TypeScript definitions included
- Minified production build
- ES6 module support

**Android SDK**
- AAR package for easy integration
- Kotlin and Java compatible
- Builder pattern for requests
- Callback-based async operations
- ProGuard rules included

**Documentation**
- Complete installation guide
- Deployment checklist
- Java SDK technical documentation
- Android SDK guide with examples
- JavaScript SDK guide with examples
- REST API reference
- MkDocs-based documentation site

---

## Version History Summary

| Version | Date | Type | Description |
|---------|------|------|-------------|
| 1.0.0 | 2025-12-15 | Initial | First public release |

---

## Upcoming (Planned)

### [1.1.0] - TBD
- Performance improvements
- Additional date format options
- Enhanced error messages

### [2.0.0] - TBD
- Breaking changes (if any)
- Major feature additions

---

## Support Policy

| Version | Status | Support Until |
|---------|--------|---------------|
| 1.0.x | ✅ Active | TBD |

---

## Reporting Issues

If you encounter any issues, please contact:
- **Email:** support@esign.network
- **Website:** https://www.esign.network

---

**© 2025 Capricorn Technologies**
