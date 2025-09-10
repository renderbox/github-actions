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

### 2. **Build React (CRA or Vite)**

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
- Uploads the build output (`dist/` for Vite, `build/` for CRA) as an artifact

---

### 3. **Deploy to S3**

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

### 4. **Create Release**

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

---

👉 Do you want me to also include **examples for Vite vs CRA builds** in the docs (since their build outputs differ), or just keep it generic with a note about adjusting the output folder?
