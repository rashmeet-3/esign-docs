# Versioning & Compatibility Policy

This document explains our versioning strategy and backward compatibility commitments.

---

## Semantic Versioning

We follow [Semantic Versioning 2.0.0](https://semver.org/):

```
MAJOR.MINOR.PATCH

Example: 1.2.3
         │ │ │
         │ │ └── PATCH: Bug fixes (backward compatible)
         │ └──── MINOR: New features (backward compatible)
         └────── MAJOR: Breaking changes (may require code changes)
```

---

## Version Types

### PATCH Release (1.0.x)
**Example:** 1.0.0 → 1.0.1

- Bug fixes
- Security patches
- Documentation updates
- No API changes

**Your code:** ✅ No changes needed

---

### MINOR Release (1.x.0)
**Example:** 1.0.0 → 1.1.0

- New features added
- New endpoints added
- New optional parameters
- Existing APIs unchanged

**Your code:** ✅ No changes needed (unless you want new features)

---

### MAJOR Release (x.0.0)
**Example:** 1.0.0 → 2.0.0

- Breaking changes
- Removed or renamed APIs
- Changed request/response format
- Migration may be required

**Your code:** ⚠️ May require updates (see migration guide)

---

## API Versioning

Our REST API uses URL-based versioning:

```
/api/v1/esign/upload-pdf    ← Version 1
/api/v2/esign/upload-pdf    ← Version 2 (future)
```

### Version Support

| API Version | Status | Supported Until |
|-------------|--------|-----------------|
| v1 | ✅ Active | Until v3 release |
| v2 | 🔜 Planned | TBD |

---

## Backward Compatibility Promise

### What We Guarantee (Within Same Major Version)

| Aspect | Guarantee |
|--------|-----------|
| Existing endpoints | Will not be removed |
| Request parameters | Required params won't change |
| Response fields | Existing fields won't be removed |
| Error codes | Existing codes won't change meaning |
| SDK methods | Public methods won't be removed |

### What May Change

| Aspect | May Change |
|--------|------------|
| New optional parameters | May be added |
| New response fields | May be added |
| New error codes | May be added |
| New endpoints | May be added |
| Internal implementation | May change |

---

## Deprecation Policy

When we plan to remove or change a feature:

### Step 1: Deprecation Notice
- Feature marked as **deprecated** in documentation
- Deprecation warning in response headers (if applicable)
- Minimum **6 months** notice before removal

### Step 2: Documentation Update
- Migration guide provided
- Alternative approach documented
- Timeline announced

### Step 3: Removal
- Feature removed in next **MAJOR** version
- Old version continues to work until end of support

---

## SDK Compatibility Matrix

### Current Version: 1.0.0

| Component | Version | Compatibility |
|-----------|---------|---------------|
| REST API | v1 | ✅ |
| JavaScript SDK | 1.0.0 | ✅ |
| Android SDK (AAR) | 1.0.0 | ✅ |
| Java SDK | 1.0.0 | ✅ |

### Minimum Requirements

| Requirement | Version |
|-------------|---------|
| Java JDK | 17+ |
| Android SDK | API 21+ (Android 5.0) |
| Node.js (for JS SDK) | 14+ |
| Modern Browsers | Chrome 80+, Firefox 75+, Safari 13+, Edge 80+ |

---

## Support Lifecycle

### Active Support
- Bug fixes
- Security patches
- Documentation updates
- Technical support

### Maintenance Mode
- Security patches only
- No new features
- Limited technical support

### End of Life (EOL)
- No updates
- No support
- Migration required

### Version Support Timeline

| Version | Release Date | Active Support | Maintenance | EOL |
|---------|--------------|----------------|-------------|-----|
| 1.0.0 | 2025-12-15 | ✅ Current | - | - |

---

## Migration Guides

When breaking changes occur, we provide:

1. **What Changed** - List of breaking changes
2. **Why It Changed** - Reason for the change
3. **How to Migrate** - Step-by-step instructions
4. **Code Examples** - Before and after code

Migration guides will be available at: `docs/migration/`

---

## Checking Your Version

### REST API
```bash
curl https://your-server/api/v1/esign/health
# Response includes version info
```

### JavaScript SDK
```javascript
import { ESignClient } from 'esign-sdk';
console.log(ESignClient.VERSION); // "1.0.0"
```

### Android SDK
```kotlin
val version = ESignClient.VERSION // "1.0.0"
```

---

## Staying Updated

### Release Notifications
- Check [Changelog](changelog.md) for updates
- Subscribe to release announcements (if available)

### Recommended Update Strategy

| Release Type | Recommendation |
|--------------|----------------|
| PATCH | Update immediately |
| MINOR | Update within 1 month |
| MAJOR | Plan migration, update within 6 months |

---

## Questions?

If you have questions about versioning or compatibility:

- **Email:** support@esign.network
- **Website:** https://www.esign.network

---

**Version:** 1.0 | **Last Updated:** December 2025

**© 2025 Capricorn Technologies**
