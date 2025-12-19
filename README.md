# github-actions

This repository provides reusable composite actions for Node.js / React / Django projects.
Use them across your repos to standardize CI/CD workflows.

---

## 📦 Available Actions

### 1. **Lint and Test**

Run linting and tests in a Node.js project.

```yaml
- uses: org/reusable-actions/lint-and-test@v1
  with:
    node-version: 20
```

**Inputs**:

- `node-version` (optional, default `20`) → Node.js version to install.

This action will:

- Checkout source code
- Install dependencies with `npm ci`
- Run `npm run lint` (if defined in `package.json`)
- Run `npm test` in CI mode with coverage

---

### 2. **Build React (CRA)**

Builds a React app and uploads it as a GitHub Actions artifact.

```yaml
- uses: renderbox/github-actions/build-react@v1
  with:
    node-version: 20
    artifact-name: react-build-${{ github.sha }}
```

**Inputs**:

- `node-version` (optional, default `20`)
- `artifact-name` (optional, default `react-build`)

This action will:

- Checkout source
- Install dependencies with `npm ci`
- Run `npm run build`
- Uploads the build output (`build/` for CRA) as an artifact

---

### 3. **Build React (Vite)**

Builds a Vite app and uploads it as a GitHub Actions artifact.

```yaml
- uses: renderbox/github-actions/build-react-vite@v1
  with:
    node-version: 20
    artifact-name: vite-build-${{ github.sha }}
```

**Inputs**:

- `node-version` (optional, default `20`)
- `artifact-name` (optional, default `vite-build`)

This action will:

- Checkout source
- Install dependencies with `npm ci`
- Run `npm run build`
- Uploads the build output (`build/` for Vite) as an artifact

---

### 4. **Deploy to S3**

Download a previously uploaded build artifact and deploy it to Amazon S3.

```yaml
- uses: renderbox/github-actions/deploy-s3@v1
  with:
    artifact-name: react-build-${{ github.sha }}
    bucket: my-bucket-name
    subfolder: my/subfolder
    aws-region: us-east-1
```

**Inputs**:

- `artifact-name` → Name of artifact to deploy
- `bucket` → S3 bucket name
- `subfolder` → Subfolder path inside the bucket
- `aws-region` (optional, default `us-east-1`)

Requires AWS credentials in repo secrets:

- `AWS_ACCESS_KEY_ID`
- `AWS_SECRET_ACCESS_KEY`

---

### 5. **Deploy Latest Release to S3**

Downloads the latest release asset from a GitHub repo and uploads it to an S3 bucket.

```yaml
- uses: renderbox/github-actions/deploy-latest-release-to-s3@v1
  with:
    repository: owner/repo
    asset_pattern: *.zip
    aws_access_key_id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws_secret_access_key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    aws_region: us-east-1
    s3_bucket: my-bucket-name
    subfolder: my/subfolder
```

**Inputs**:

- `repository` → Repository in owner/repo format
- `asset_pattern` → Glob pattern to match the asset name
- `aws_access_key_id` → AWS Access Key ID
- `aws_secret_access_key` → AWS Secret Access Key
- `aws_region` → AWS Region
- `s3_bucket` → S3 bucket name
- `subfolder` → S3 key prefix (folder path in bucket, optional)

---

### 6. **Create Release**

Create a GitHub Release (optionally uploading files).

```yaml
- uses: renderbox/github-actions/create-release@v1
  with:
    tag_name: v1.0.0
    release_name: "Release v1.0.0"
    body: "Release notes here"
    files: dist/**
```

**Inputs**:

- `tag_name` → Tag for the release (e.g., `v1.0.0`)
- `release_name` (optional) → Human-readable name
- `body` (optional) → Release notes
- `files` (optional) → Files or directories to upload

---

### 7. **Publish to PyPI**

Build and publish a Python package to PyPI.

```yaml
- uses: renderbox/github-actions/publish-to-pypi@v1
  with:
    python-version: 3.12
```

**Inputs**:

- `python-version` (optional, default `3.12`) → Python version to use

This action will:

- Checkout source
- Set up Python environment
- Build distribution packages
- Store the distribution packages as artifacts
- Publish packages to PyPI

---

### 8. **Test Django Versions**

Run tests across multiple Django versions to ensure compatibility.

```yaml
- uses: renderbox/github-actions/test-django-versions@v1
  with:
    python-version: 3.12
    django-versions: "3.2,4.0,4.1"
```

**Inputs**:

- `python-version` → Python version to use
- `django-versions` → Comma-separated list of Django versions to test

This action will:

- Set up Python environment
- Install dependencies for each Django version
- Run tests for each version

---

### 9. **Check Django Migrations**

Verify that Django migrations are up-to-date.

```yaml
- uses: renderbox/github-actions/check-django-migrations@v1
  with:
    python-version: 3.12
```

**Inputs**:

- `python-version` → Python version to use

This action will:

- Set up Python environment
- Check for missing migrations

---

### 10. **Lint Python**

Run Python linters to ensure code quality.

```yaml
- uses: renderbox/github-actions/lint-python@v1
  with:
    python-version: 3.12
```

**Inputs**:

- `python-version` → Python version to use

This action will:

- Set up Python environment
- Run linters (e.g., flake8, black)

---

## ⚡ Example Full Workflow

```yaml
name: CI/CD

on:
  push:
    branches: [main]

jobs:
  lint-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: renderbox/github-actions/lint-and-test@v1

  build:
    runs-on: ubuntu-latest
    needs: lint-and-test
    steps:
      - uses: renderbox/github-actions/build-react@v1
        with:
          artifact-name: react-build-${{ github.sha }}

  release:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - uses: actions/download-artifact@v4
        with:
          name: react-build-${{ github.sha }}
          path: dist/
      - uses: renderbox/github-actions/create-release@v1
        with:
          tag_name: v${{ github.run_number }}
          release_name: "Release v${{ github.run_number }}"
          files: dist/**

  deploy:
    runs-on: ubuntu-latest
    needs: release
    steps:
      - uses: renderbox/github-actions/deploy-s3@v1
        with:
          artifact-name: react-build-${{ github.sha }}
          bucket: my-bucket-name
          subfolder: my/subfolder
```

