# eSign SDK Documentation

This folder contains the documentation for the eSign SDK.

---

## 📖 View Documentation

### Option 1: Serve Locally with MkDocs (Recommended)

```bash
# Install MkDocs (one-time setup)
pip install mkdocs-material

# Navigate to documentation folder
cd documentation

# Serve locally
mkdocs serve

# Open http://127.0.0.1:8000 in browser
```

### Option 2: Read Markdown Files Directly

All documentation is in the `docs/` folder as Markdown files:

| File | Description |
|------|-------------|
| `docs/index.md` | Home page |
| `docs/installation-deployment-guide.md` | Complete installation guide |
| `docs/deployment-checklist.md` | Production deployment checklist |
| `docs/rest-api-reference.md` | REST API quick reference |
| `docs/esign-api-guide.md` | Complete API guide |
| `docs/android-sdk-guide.md` | Android SDK integration |
| `docs/js-sdk-guide.md` | JavaScript SDK integration |

---

## 📁 Project Structure

```
documentation/
├── docs/
│   ├── index.md                           # Home page
│   ├── installation-deployment-guide.md   # Complete installation guide
│   ├── deployment-checklist.md            # Production checklist
│   ├── rest-api-reference.md              # REST API reference
│   ├── esign-api-guide.md                 # Complete API guide
│   ├── android-sdk-guide.md               # Android SDK guide
│   ├── android-sdk-quickstart.md          # Android quick start
│   ├── js-sdk-guide.md                    # JavaScript SDK guide
│   ├── js-sdk-quickstart.md               # JavaScript quick start
│   ├── quick-start-guide.md               # Java SDK quick start
│   ├── eSign-sdk-technical-documentation.md      # Technical docs part 1
│   ├── eSign-sdk-technical-documentation-part2.md # Technical docs part 2
│   ├── web-ui-internal-api.md             # Web UI internal endpoints
│   ├── changelog.md                       # Version history
│   └── versioning-policy.md               # Versioning policy
├── mkdocs.yml                             # MkDocs configuration
└── README.md                              # This file
```

---

## 🔧 Building Documentation

### Build Static Site

```bash
# Build static HTML site
mkdocs build

# Output will be in 'site/' folder
```

### Customization

Edit `mkdocs.yml` to customize:

- Site name and description
- Theme colors
- Navigation structure

---

## 📞 Support

For questions about the eSign SDK, contact:

- **Email:** support@capricornid.com
- **Website:** https://www.esign.network

---

**Version:** 1.0.0 | **Last Updated:** December 2025

© 2025 Capricorn Technologies. All rights reserved.
