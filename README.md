# eSign SDK Documentation

This repository contains the documentation for the eSign Java SDK.

## 📖 View Documentation

**Live Documentation:** https://YOUR_USERNAME.github.io/YOUR_REPO_NAME/

---

## 🚀 How to Host on GitHub Pages (Step-by-Step)

### Step 1: Create a New GitHub Repository

1. Go to [github.com](https://github.com) and sign in
2. Click the **+** icon (top right) → **New repository**
3. Fill in:
   - **Repository name:** `esign-docs` (or any name you prefer)
   - **Description:** `eSign SDK Documentation`
   - **Visibility:** Public (required for free GitHub Pages)
   - ✅ Check "Add a README file"
4. Click **Create repository**

### Step 2: Upload Documentation Files

**Option A: Using GitHub Web Interface (Easiest)**

1. In your new repository, click **Add file** → **Upload files**
2. Drag and drop ALL these files/folders:
   ```
   docs/                    (folder with all .md files)
   .github/                 (folder with workflows)
   mkdocs.yml              (configuration file)
   .gitignore              (optional)
   README.md               (this file)
   ```
3. Click **Commit changes**

**Option B: Using Git Command Line**

```bash
# Clone your new repository
git clone https://github.com/YOUR_USERNAME/YOUR_REPO_NAME.git
cd YOUR_REPO_NAME

# Copy all documentation files here
# (copy docs/, .github/, mkdocs.yml, etc.)

# Add, commit, and push
git add .
git commit -m "Add eSign SDK documentation"
git push origin main
```

### Step 3: Enable GitHub Pages

1. Go to your repository on GitHub
2. Click **Settings** (gear icon)
3. Scroll down to **Pages** in the left sidebar
4. Under **Build and deployment**:
   - **Source:** Select **GitHub Actions**
5. Click **Save**

### Step 4: Update Configuration

Edit `mkdocs.yml` and replace placeholders:

```yaml
site_url: https://YOUR_USERNAME.github.io/YOUR_REPO_NAME/
repo_name: YOUR_USERNAME/YOUR_REPO_NAME
repo_url: https://github.com/YOUR_USERNAME/YOUR_REPO_NAME
```

**Example:** If your GitHub username is `johndoe` and repo is `esign-docs`:
```yaml
site_url: https://johndoe.github.io/esign-docs/
repo_name: johndoe/esign-docs
repo_url: https://github.com/johndoe/esign-docs
```

### Step 5: Trigger Deployment

1. Make any small edit to a file (or push a commit)
2. Go to **Actions** tab in your repository
3. You'll see the workflow running
4. Wait for it to complete (green checkmark)

### Step 6: Access Your Documentation

Your documentation will be live at:
```
https://YOUR_USERNAME.github.io/YOUR_REPO_NAME/
```

---

## 📁 Project Structure

```
esign-docs/
├── .github/
│   └── workflows/
│       └── deploy.yml          # GitHub Actions workflow
├── docs/
│   ├── index.md                # Home page
│   ├── quick-start-guide.md    # Quick start
│   ├── esign-api-guide.md      # API guide (UPDATED)
│   ├── api-reference-card.md   # Quick reference (UPDATED)
│   ├── deployment-checklist.md # Deployment guide
│   ├── eSign-sdk-technical-documentation.md      # Part 1
│   └── eSign-sdk-technical-documentation-part2.md # Part 2
├── mkdocs.yml                  # MkDocs configuration
├── .gitignore                  # Git ignore file
└── README.md                   # This file
```

---

## 🔧 Local Development (Optional)

To preview documentation locally before pushing:

```bash
# Install MkDocs
pip install mkdocs-material

# Serve locally
mkdocs serve

# Open http://127.0.0.1:8000 in browser
```

---

## 📝 Making Updates

1. Edit any `.md` file in the `docs/` folder
2. Commit and push to `main` branch
3. GitHub Actions will automatically rebuild and deploy

---

## 🔗 Useful Links

- [MkDocs Documentation](https://www.mkdocs.org/)
- [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/)
- [GitHub Pages Documentation](https://docs.github.com/en/pages)

---

## 📞 Support

For questions about the eSign SDK, contact:
- **Email:** support@esign.network
- **Website:** https://www.esign.network

---

© 2025 Capricorn Technologies. All rights reserved.
