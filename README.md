name: CI-CD Pipeline
 
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
 
jobs:
 
  # ---------- CONTINUOUS INTEGRATION ----------
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
 
      - name: Set up runtime
        uses: actions/setup-node@v4
        with:
          node-version: '20'
 
      - name: Install dependencies
        run: npm ci
 
      - name: Run build
        run: npm run build
 
      - name: Run automated tests
        run: npm test
 
  static-analysis:
    runs-on: ubuntu-latest
    needs: build-and-test
    steps:
      - uses: actions/checkout@v4
 
      - name: Initialize CodeQL
        uses: github/codeql-action/init@v3
        with:
          languages: javascript
 
      - name: Run CodeQL analysis
        uses: github/codeql-action/analyze@v3
 
  security-scan:
    runs-on: ubuntu-latest
    needs: build-and-test
    steps:
      - uses: actions/checkout@v4
 
      - name: Install dependencies
        run: npm ci
 
      - name: Run dependency vulnerability audit
        run: npm audit --audit-level=moderate
 
  # ---------- CONTINUOUS DEPLOYMENT ----------
  deploy:
    runs-on: ubuntu-latest
    needs: [static-analysis, security-scan]
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
 
      - name: Configure cloud credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: eu-west-1
 
      - name: Deploy application
        run: |
          # replace with your actual deploy command,
          # e.g. eb deploy, aws s3 sync, sam deploy, etc.
          echo "deploy step goes here"
 
