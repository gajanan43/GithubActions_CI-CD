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

jobs:
  build-test-deploy:
    runs-on: ubuntu-latest

    steps:
      # Step 1: Checkout repository
      - name: Checkout code
        uses: actions/checkout@v4

      # Step 2: Setup Node.js
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "18"

      # Step 3: Install dependencies
      - name: Install dependencies
        run: npm install

      # Step 4: Run tests
      - name: Run tests
        run: npm test

      # Step 5: Build app
      - name: Build
        run: npm run build

      # Step 6: Deploy (example: Vercel, AWS, Docker, etc.)
      - name: Deploy to Vercel
        run: npx vercel --prod --token=${{ secrets.VERCEL_TOKEN }}
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


