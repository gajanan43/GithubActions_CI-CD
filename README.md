# GithubActions_CI-CD

## 🔹 What is CI/CD?

* **CI (Continuous Integration):**
  Developers frequently push their code to GitHub. Each push triggers automated processes like building the code, running tests, linting, and static analysis. The goal: catch bugs early.

* **CD (Continuous Delivery / Deployment):**
  Once the code passes CI, it’s automatically deployed (or made ready for deployment) to staging/production environments.

  * **Continuous Delivery:** Code is packaged and ready for deployment but might need manual approval.
  * **Continuous Deployment:** Fully automated, goes directly to production without manual intervention.

---

## 🔹 GitHub Actions (CI/CD on GitHub)

GitHub Actions is GitHub’s built-in CI/CD tool. It uses **YAML workflow files** stored in `.github/workflows/`.

### Workflow basics:

* **Triggers (`on:`)** – defines when the workflow runs:

  ```yaml
  on:
    push:
      branches: ["main"]
    pull_request:
      branches: ["main"]
  ```

  👉 Runs the pipeline when someone pushes code to `main` or creates a PR targeting `main`.

* **Jobs** – logical groups of steps (can run in parallel or sequentially).

* **Steps** – individual actions (e.g., checkout code, install dependencies, run tests).

* **Runners** – environments where jobs run (e.g., `ubuntu-latest`, `windows-latest`, or self-hosted runners).

---

## 🔹 Example CI/CD Workflow

```yaml

name: CI/CD Pipeline

on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]
  workflow_dispatch:  # Manual trigger

jobs:
  # ------------------------------
  # 1. Linting & Static Analysis
  # ------------------------------
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # Language runtime (choose Node, Python, Java, etc.)
      - uses: actions/setup-node@v4
        with:
          node-version: 18
      # - uses: actions/setup-python@v5
      #   with:
      #     python-version: "3.11"
      # - uses: actions/setup-java@v4
      #   with:
      #     java-version: "17"

      - run: npm install   # or pip install -r requirements.txt, mvn install
      - run: npm run lint  # or pylint src/, flake8, checkstyle, etc.

  # ------------------------------
  # 2. Unit Tests
  # ------------------------------
  unit-tests:
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 18
      - run: npm install
      - run: npm run test:unit   # Replace with pytest, mvn test, etc.

  # ------------------------------
  # 3. Integration Tests
  # ------------------------------
  integration-tests:
    runs-on: ubuntu-latest
    needs: unit-tests
    steps:
      - uses: actions/checkout@v4
      - run: npm run test:integration  # or pytest -m integration

  # ------------------------------
  # 4. End-to-End (E2E) Tests
  # ------------------------------
  e2e-tests:
    runs-on: ubuntu-latest
    needs: integration-tests
    steps:
      - uses: actions/checkout@v4
      - run: npm run test:e2e  # or cypress run, playwright test

  # ------------------------------
  # 5. Security Scan
  # ------------------------------
  security:
    runs-on: ubuntu-latest
    needs: e2e-tests
    steps:
      - uses: actions/checkout@v4
      - run: npm audit --audit-level=high || true
      # Or: pip-audit, snyk, trivy, etc.

  # ------------------------------
  # 6. Build
  # ------------------------------
  build:
    runs-on: ubuntu-latest
    needs: security
    steps:
      - uses: actions/checkout@v4
      - run: npm install
      - run: npm run build   # or mvn package, python setup.py build

  # ------------------------------
  # 7. Deploy
  # ------------------------------
  deploy:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - uses: actions/checkout@v4

      # Example: Deploy to Vercel
      - run: npx vercel --prod --token=${{ secrets.VERCEL_TOKEN }}

      # Or: Deploy to AWS
      # - uses: aws-actions/configure-aws-credentials@v4
      #   with:
      #     aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
      #     aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      #     aws-region: ap-south-1
      # - run: aws s3 sync build/ s3://my-bucket --delete
```
---


## 🔹 Key Concepts in Depth

1. **Triggers (Events):**

   * `push`, `pull_request`, `workflow_dispatch` (manual trigger), `schedule` (cron jobs).
   * Example: run tests daily at midnight:

     ```yaml
     on:
       schedule:
         - cron: "0 0 * * *"
     ```

2. **Jobs & Dependencies:**

   * Jobs can depend on each other (`needs:`).
   * Example: Only deploy if tests pass:

     ```yaml
     jobs:
       test:
         runs-on: ubuntu-latest
         steps:
           - run: npm test
       deploy:
         runs-on: ubuntu-latest
         needs: test
         steps:
           - run: echo "Deploying..."
     ```

3. **Secrets & Environment Variables:**

   * Store credentials (API keys, tokens) securely in GitHub `Settings → Secrets`.
   * Example:

     ```yaml
     - name: Deploy
       run: ./deploy.sh
       env:
         API_KEY: ${{ secrets.API_KEY }}
     ```

4. **Artifacts:**

   * Save and share build outputs (e.g., compiled binaries, reports).
   * Example:

     ```yaml
     - name: Upload artifact
       uses: actions/upload-artifact@v4
       with:
         name: build-output
         path: dist/
     ```

5. **Matrix Builds (Test on multiple OS / versions):**

   ```yaml
   strategy:
     matrix:
       os: [ubuntu-latest, windows-latest]
       node: [16, 18]
   runs-on: ${{ matrix.os }}
   steps:
     - uses: actions/setup-node@v4
       with:
         node-version: ${{ matrix.node }}
   ```

---

## 🔹 CI/CD Flow in Real Life

1. **Developer pushes code to GitHub.**
2. **Pipeline starts automatically**:

   * Code is checked out.
   * Dependencies are installed.
   * Code is built & tested.
3. **If tests pass** → build artifacts (Docker images, bundles, binaries).
4. **If build succeeds** → deploy to cloud (AWS, Azure, GCP, Vercel, Netlify, etc.).
5. **If deployment succeeds** → notify team (Slack/Email).

---

## 🔹 Benefits of GitHub CI/CD

* 🚀 Faster releases
* ✅ Automatic testing → higher code quality
* 🔐 Secure deployments using secrets
* 🌍 Cross-platform testing (Linux, Windows, macOS)
* 🔄 Fully integrated with GitHub (issues, PRs, branches)

---
---

