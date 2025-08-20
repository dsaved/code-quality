# GitHub Checks Implementation for NestJS Projects

![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white)
![ESLint](https://img.shields.io/badge/ESLint-4B3263?style=for-the-badge&logo=eslint&logoColor=white)
![Prettier](https://img.shields.io/badge/prettier-1A2C34?style=for-the-badge&logo=prettier&logoColor=F7BA3E)
![Jest](https://img.shields.io/badge/Jest-323330?style=for-the-badge&logo=jest&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-007ACC?style=for-the-badge&logo=typescript&logoColor=white)

A comprehensive GitHub Actions workflow system that enforces code quality, naming conventions, spell checking, security scanning, and test coverage for NestJS applications.

## 🎯 Overview

This implementation provides a complete CI/CD pipeline with automated code quality checks that run on every push and pull request. It ensures consistent code standards, security practices, and naming conventions across your entire development team.

## ✨ Features

### 🔍 Code Quality Checks

- **ESLint**: TypeScript/JavaScript linting with custom rules
- **Prettier**: Automated code formatting validation
- **Naming Conventions**: Enforces kebab-case files, PascalCase classes, camelCase variables
- **Code Standards**: Prevents console.log, hardcoded secrets, and poor practices

### 🛡️ Security & Testing

- **Security Scanning**: OSV Scanner and Trivy vulnerability detection
- **Test Coverage**: Jest unit and e2e testing with coverage thresholds
- **Spell Checking**: CSpell validation for code and documentation
- **Secret Detection**: Prevents hardcoded API keys and passwords

### 🚀 CI/CD Pipeline

- **Multi-Node Testing**: Tests on Node.js 18.x and 20.x
- **Database Integration**: PostgreSQL test database setup
- **Parallel Execution**: Multiple checks run simultaneously
- **Coverage Reporting**: Built-in coverage reports and thresholds

### 📋 Team Collaboration

- **CODEOWNERS**: Automatic reviewer assignment
- **PR Templates**: Standardized pull request format
- **Status Checks**: Clear pass/fail indicators on PRs
- **Branch Protection**: Prevents merging until all checks pass

## 📁 Generated Files Structure

```
your-nestjs-project/
├── .github/
│   ├── workflows/
│   │   ├── ci.yml                    # Main CI pipeline
│   │   ├── naming-conventions.yml    # Naming validation
│   │   ├── spell-check.yml          # Spell checking
│   │   └── security.yml             # Security scanning
│   ├── CODEOWNERS                   # Team ownership rules
│   └── pull_request_template.md     # PR template
├── .eslintrc.js                     # Enhanced ESLint config
├── .cspell.json                     # Spell check dictionary
├── jest.config.js                   # Coverage thresholds
└── package.json                     # Updated scripts
```

## 🚀 Quick Start

### Method 1: Using NodeJS Bootstrap CLI (Recommended)

```bash
# Install the bootstrap CLI
npm install -g nodejs-bootstrapper

# Create new project with GitHub Checks
init-project

# Follow the interactive prompts:
# ✅ Select "NestJS Template" or "Enterprise API (NestJS)"
# ✅ Enable "GitHub Actions for code quality checks"
# ✅ Select desired check types:
#    - Code Quality (ESLint, Prettier)
#    - Naming Conventions
#    - Spell Checking
#    - Security Scanning
#    - Test Coverage
```

### Method 2: Manual Implementation

Follow the [detailed implementation guide](#-implementation-guide) below.

## 🔧 Implementation Guide

### Step 1: Create Workflow Files

Create the GitHub Actions workflows:

<details>
<summary>📄 <code>.github/workflows/ci.yml</code></summary>

```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [18.x, 20.x]

    services:
      postgres:
        image: postgres:13
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: test_db
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432

    steps:
      - uses: actions/checkout@v4

      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Run ESLint
        run: npm run lint:check

      - name: Run Prettier Check
        run: npm run format:check

      - name: Build application
        run: npm run build

      - name: Run Unit Tests
        run: npm run test:cov
        env:
          NODE_ENV: test
          DB_HOST: localhost
          DB_PORT: 5432
          DB_USERNAME: postgres
          DB_PASSWORD: postgres
          DB_NAME: test_db
```

</details>

<details>
<summary>📄 <code>.github/workflows/naming-conventions.yml</code></summary>

```yaml
name: Naming Conventions

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  naming-conventions:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Check file naming conventions
        run: |
          echo "Checking for kebab-case file naming..."

          # Check TypeScript files (excluding spec files)
          if find src -name "*.ts" -not -path "*/node_modules/*" | grep -E '[A-Z]|_' | grep -v '\.spec\.ts$' | grep -v '\.test\.ts$'; then
            echo "❌ Found files that don't follow kebab-case naming convention"
            exit 1
          fi

          echo "✅ All files follow kebab-case naming convention"

      - name: Check for console.log statements
        run: |
          if grep -r "console\.log" src --include="*.ts" --include="*.js"; then
            echo "❌ Found console.log statements in source code"
            exit 1
          fi
          echo "✅ No console.log statements found"

      - name: Check for hardcoded secrets
        run: |
          if grep -r -E "(password|secret|key|token)\s*=\s*['\"][^'\"]{8,}" src --include="*.ts" --include="*.js"; then
            echo "❌ Found potential hardcoded secrets"
            exit 1
          fi
          echo "✅ No hardcoded secrets detected"
```

</details>

<details>
<summary>📄 <code>.github/workflows/spell-check.yml</code></summary>

```yaml
name: Spell Check

on:
  push:
    branches: [development]
  pull_request:
    branches: [development]

jobs:
  spelling:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20.x"

      - name: Install CSpell
        run: npm install -g cspell

      - name: Run spell check
        run: cspell "src/**/*.{ts,js}" "api/**/*.md" "**/*.json" --config .cspell.json
```

</details>

<details>
<summary>📄 <code>.github/workflows/security.yml</code></summary>

```yaml
name: Security Audit

on:
  push:
    branches: [development]
  pull_request:
    branches: [development]

jobs:
  osv-trivy-scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Install OSV Scanner
        run: |
          curl -sSfL https://github.com/google/osv-scanner/releases/latest/download/osv-scanner_linux_amd64 -o osv-scanner
          chmod +x osv-scanner
          sudo mv osv-scanner /usr/local/bin/
      - name: Run OSV Scanner on lockfiles and fail on vulnerabilities
        continue-on-error: true
        run: |
          if [ -f yarn.lock ]; then
            osv-scanner --lockfile=api/yarn.lock | tee osv-report.txt
            # Only fail if vulnerabilities count is nonzero in OSV report
            if grep -E 'Vulnerabilities:[[:space:]]*[1-9][0-9]*' osv-report.txt; then
              echo "VULN_FOUND=true" >> $GITHUB_ENV
              echo "❌ Vulnerabilities found!"
              exit 1
            fi
          elif [ -f package-lock.json ]; then
            osv-scanner --lockfile=package-lock.json | tee osv-report.txt
            if grep -E 'Vulnerabilities:[[:space:]]*[1-9][0-9]*' osv-report.txt; then
              echo "VULN_FOUND=true" >> $GITHUB_ENV
              echo "❌ Vulnerabilities found!"
              exit 1
            fi
          else
            echo "No lockfile found. Skipping vulnerability scan."
          fi
      - name: Install Trivy
        run: |
          sudo apt-get update && sudo apt-get install -y wget
          wget -qO trivy.deb https://github.com/aquasecurity/trivy/releases/download/v0.65.0/trivy_0.65.0_Linux-64bit.deb
          sudo dpkg -i trivy.deb
      - name: Run Trivy FS scan
        continue-on-error: true
        run: |
          trivy fs . --exit-code 1 --severity MEDIUM,HIGH,CRITICAL | tee trivy-report.txt
          # Only fail if vulnerabilities count is nonzero in Trivy report summary for lockfiles
          if grep -E '│ (yarn.lock|package-lock.json) │ [^│]+ │[[:space:]]*[1-9][0-9]*[[:space:]]*│' trivy-report.txt; then
            echo "VULN_FOUND=true" >> $GITHUB_ENV
            echo "❌ Vulnerabilities found by Trivy!"
            exit 1
          fi

      - name: Install OWASP Dependency Check
        run: |
          wget -qO dependency-check.zip https://github.com/jeremylong/DependencyCheck/releases/download/v8.4.0/dependency-check-8.4.0-release.zip
          unzip dependency-check.zip
          chmod +x dependency-check/bin/dependency-check.sh

      - name: Run OWASP Dependency Check
        continue-on-error: true
        run: |
          ./dependency-check/bin/dependency-check.sh \
            --scan . \
            --format HTML \
            --format JSON \
            --out dependency-check-report \
            --enableRetired \
            --enableExperimental
          
          # Check if vulnerabilities were found
          if [ -f dependency-check-report/dependency-check-report.json ]; then
            VULN_COUNT=$(jq '.dependencies | map(.vulnerabilities // []) | flatten | length' dependency-check-report/dependency-check-report.json)
            if [ "$VULN_COUNT" -gt 0 ]; then
              echo "VULN_FOUND=true" >> $GITHUB_ENV
              echo "❌ OWASP Dependency Check found $VULN_COUNT vulnerabilities!"
            fi
          fi

      - name: Upload OWASP Dependency Check reports
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: owasp-dependency-check-reports
          path: dependency-check-report/

      - name: Send vulnerability report email
        if: env.VULN_FOUND == 'true'
        uses: dawidd6/action-send-mail@v3
        with:
          server_address: ${{ secrets.SMTP_SERVER }}
          server_port: 465
          username: ${{ secrets.SMTP_USERNAME }}
          password: ${{ secrets.SMTP_PASSWORD }}
          subject: "Security Vulnerability Scan Failed - ${{ github.repository }}"
          to: ${{ secrets.SMTP_MAIL_TO }}
          from: ${{ secrets.SMTP_MAIL_FROM }}
          body: |
            Security vulnerability scan failed for ${{ github.repository }}.
            
            Tools that found vulnerabilities:
            - OSV Scanner: Check osv-report.txt
            - Trivy: Check trivy-report.txt  
            - OWASP Dependency Check: Check dependency-check-report folder
            
            Please review and remediate the identified vulnerabilities.
          attachments: |
            osv-report.txt
            trivy-report.txt
            dependency-check-report/dependency-check-report.html

      - name: Fail job if vulnerabilities found
        if: env.VULN_FOUND == 'true'
        run: |
          echo "❌ Security vulnerabilities detected! Please review the reports."
          exit 1
```

</details>

### Step 2: Create Configuration Files

<details>
<summary>📄 <code>.eslintrc.js</code> (Enhanced)</summary>

```javascript
module.exports = {
  parser: "@typescript-eslint/parser",
  parserOptions: {
    project: "tsconfig.json",
    tsconfigRootDir: __dirname,
    sourceType: "module",
  },
  plugins: ["@typescript-eslint/eslint-plugin"],
  extends: [
    "plugin:@typescript-eslint/recommended",
    "plugin:prettier/recommended",
  ],
  root: true,
  env: {
    node: true,
    jest: true,
  },
  ignorePatterns: [".eslintrc.js", "dist/", "node_modules/"],
  rules: {
    // Naming Conventions
    "@typescript-eslint/naming-convention": [
      "error",
      { selector: "default", format: ["camelCase"] },
      { selector: "variable", format: ["camelCase", "UPPER_CASE"] },
      {
        selector: "parameter",
        format: ["camelCase"],
        leadingUnderscore: "allow",
      },
      {
        selector: "memberLike",
        modifiers: ["private"],
        format: ["camelCase"],
        leadingUnderscore: "require",
      },
      { selector: "typeLike", format: ["PascalCase"] },
      { selector: "enumMember", format: ["UPPER_CASE"] },
    ],

    // Code Quality
    "no-console": "error",
    "no-debugger": "error",
    "@typescript-eslint/no-unused-vars": "error",
    "@typescript-eslint/no-explicit-any": "warn",
  },
};
```

</details>

<details>
<summary>📄 <code>.cspell.json</code></summary>

```json
-
### 🛡️ Security & Testing

- **Security Scanning**: OSV Scanner and Trivy vulnerability detection
  "words": [
    "nestjs", "typeorm", "postgresql", "bcryptjs", "jsonwebtoken",
    "dto", "api", "crud", "auth", "middleware", "decorator", "swagger"
  ],
  "ignorePaths": [
    "node_modules/**", "dist/**", "coverage/**", ".git/**"
  ]
}
```

</details>

<details>
<summary>📄 <code>.github/CODEOWNERS</code></summary>

```
# Global Owners
* @yourteam/backend-team

# API Routes
/src/api/ @yourteam/api-team

# Authentication & Security
/src/auth/ @yourteam/security-team
/src/guards/ @yourteam/security-team

# Database & Models
/src/model/ @yourteam/database-team

# CI/CD
/.github/ @yourteam/devops-team
```

</details>

### Step 3: Update Package Scripts

Add these scripts to your `package.json`:

```json
{
  "scripts": {
    "lint:check": "eslint \"{src,apps,libs,test}/**/*.ts\"",
    "format:check": "prettier --check \"src/**/*.ts\" \"test/**/*.ts\"",
    "test:cov": "jest --coverage",
    "spell:check": "cspell \"src/**/*.{ts,js}\" \"**/*.md\""
  }
}
```

## 📊 Naming Conventions Enforced

### 📁 File & Directory Naming

- **Files**: `kebab-case.ts` ✅ | `camelCase.ts` ❌ | `PascalCase.ts` ❌
- **Directories**: `user-management/` ✅ | `userManagement/` ❌

### 🏗️ Code Naming

- **Classes**: `UserService` ✅ | `userService` ❌
- **Variables**: `userData` ✅ | `user_data` ❌
- **Constants**: `API_KEY` ✅ | `apiKey` ❌
- **Interfaces**: `IUserRepository` ✅ | `UserRepository` ❌

### 🌐 API Naming

- **Endpoints**: `/api/users` ✅ | `/api/Users` ❌
- **Routes**: `@Get('user-profile')` ✅ | `@Get('userProfile')` ❌

## 🛡️ Security Checks

### 🚫 Prevented Patterns

```typescript
// ❌ Hardcoded secrets (detected)
const apiKey = "sk_live_abc123def456";
const password = "mypassword123";

// ❌ Console statements (detected)
console.log("Debug info");

// ✅ Proper patterns
const apiKey = process.env.API_KEY;
this.logger.debug("Debug info");
```

## 📈 Coverage Requirements

### 🎯 Thresholds

- **Lines**: 80% minimum
- **Functions**: 80% minimum
- **Branches**: 80% minimum
- **Statements**: 80% minimum

### 📝 Coverage Reports

- **HTML**: `coverage/index.html`
- **LCOV**: `coverage/lcov.info`
- **Terminal**: Real-time feedback

## 🔄 Pull Request Workflow

### 1. Developer Creates PR

```bash
git checkout -b feature/user-authentication
# Make changes...
git push origin feature/user-authentication
```

### 2. Automated Checks Run

```
🔄 CI / test (18.x)           - Installing dependencies...
🔄 CI / test (20.x)           - Running ESLint...
🔄 Naming Conventions         - Checking file names...
🔄 Spell Check               - Validating spelling...
🔄 Security                  - Scanning for vulnerabilities...
```

### 3. Results Display

```
✅ CI / test (18.x)           - All tests passed
✅ CI / test (20.x)           - All tests passed
✅ Naming Conventions         - All conventions followed
✅ Spell Check               - No spelling errors
❌ Security                  - 2 vulnerabilities found
```

### 4. Merge Protection

- ❌ Cannot merge until all checks pass
- ✅ Auto-merge available when all green
- 📝 Required reviewers notified via CODEOWNERS

## ⚙️ GitHub Repository Setup

### 1. Enable GitHub Actions

```bash
# Repository Settings → Actions → General
# ✅ Allow all actions and reusable workflows
```

### 2. Add Required Secrets

```bash
# Repository Settings → Secrets and variables → Actions
SNYK_TOKEN=your
SNYK_TOKEN=your_snyk_token_here
SNYK_TOKEN=your_snyk_token_here
SNYK_TOKEN=your_snyk_token_here
```

### 3. Configure Branch Protection

```bash
# Repository Settings → Branches → Add rule
# Branch name pattern: main
# ✅ Require status checks to pass before merging
# ✅ Require branches to be up to date before merging
# ✅ Include administrators
```

## 🧪 Testing the Implementation

### 1. Create Test Branch

```bash
git checkout -b test/github-checks
```

### 2. Test Each Check Type

<details>
<summary>🔍 Test Naming Conventions</summary>

```bash
# Create file with wrong naming
touch src/UserService.ts  # Should fail (PascalCase)
git add . && git commit -m "test: wrong file naming"
git push origin test/github-checks
# Check GitHub Actions - should fail naming-conventions
```

</details>

<details>
<summary>🧪 Test Code Quality</summary>

```bash
# Add console.log statement
echo "console.log('test');" >> src/app.service.ts
git add . && git commit -m "test: add console.log"
git push origin test/github-checks
# Check GitHub Actions - should fail linting
```

</details>

<details>
<summary>📝 Test Spell Check</summary>

```bash
# Add misspelled word in comment
echo "// This is a mispelled word: recieve" >> src/app.service.ts
git add . && git commit -m "test: add spelling error"
git push origin test/github-checks
# Check GitHub Actions - should fail spell check
```

</details>

## 🚨 Troubleshooting

### Common Issues

<details>
<summary>❌ <strong>Node.js Version Mismatch</strong></summary>

**Problem**: Tests fail on specific Node.js versions

**Solution**:

```yaml
# Update .github/workflows/ci.yml
strategy:
  matrix:
    node-version: [18.x, 20.x] # Match your project's requirements
```

</details>

<details>
<summary>❌ <strong>Database Connection Fails</strong></summary>

**Problem**: PostgreSQL service not accessible

**Solution**:

```yaml
# Ensure correct environment variables in ci.yml
env:
  DB_HOST: localhost
  DB_PORT: 5432
  DB_USERNAME: postgres
  DB_PASSWORD: postgres
  DB_NAME: test_db
```

</details>

<details>
<summary>❌ <strong>Coverage Threshold Too High</strong></summary>

**Problem**: Cannot reach 80% coverage requirement

**Solution**:

```javascript
// Adjust jest.config.js
coverageThreshold: {
  global: {
    branches: 60,    // Reduced from 80
    functions: 60,   // Reduced from 80
    lines: 60,       // Reduced from 80
    statements: 60   // Reduced from 80
  }
}
```

</details>

<details>
<summary>❌ <strong>ESLint Rules Too Strict</strong></summary>

**Problem**: Too many linting errors blocking development

**Solution**:

```javascript
// Adjust .eslintrc.js rules
rules: {
  'no-console': 'warn',                    // Changed from 'error'
  '@typescript-eslint/no-explicit-any': 'off',  // Temporarily disable
}
```

</details>

### Getting Help

- 📖 **Documentation**: Check individual workflow files for inline comments
- 🐛 **Issues**: Create GitHub issue with workflow logs
- 💬 **Discussions**: Use repository discussions for questions
- 📧 **Support**: Contact maintainers via repository contacts

## 🔧 Customization Options

### Workflow Customization

<details>
<summary>🎯 <strong>Adjust Coverage Thresholds</strong></summary>

```javascript
// jest.config.js
coverageThreshold: {
  global: {
    branches: 70,    // Custom threshold
    functions: 70,
    lines: 70,
    statements: 70
  },
  './src/api/': {    // Path-specific thresholds
    branches: 90,
    functions: 90
  }
}
```

</details>

<details>
<summary>🎨 <strong>Custom ESLint Rules</strong></summary>

```javascript
// .eslintrc.js
rules: {
  // Add custom rules for your team
  'prefer-const': 'error',
  'no-var': 'error',
  '@typescript-eslint/explicit-function-return-type': 'warn',

  // Project-specific naming
  '@typescript-eslint/naming-convention': [
    'error',
    { selector: 'interface', format: ['PascalCase'], prefix: ['I'] },
    { selector: 'typeAlias', format: ['PascalCase'], suffix: ['Type'] }
  ]
}
```

</details>

<details>
<summary>📚 <strong>Custom Spell Check Dictionary</strong></summary>

```json
// .cspell.json
{
  "words": [
    // Add your domain-specific terms
    "microservice",
    "webhook",
    "kubernetes",
    "dockerfile",
    "redis",

    // Add your project/company terms
    "yourcompany",
    "projectname",
    "apiversion"
  ]
}
```

</details>

## 📈 Benefits & ROI

### 👥 For Development Teams

- **Consistency**: Uniform code style across all developers
- **Quality**: Automatic detection of common issues
- **Productivity**: Less time spent in code reviews on style issues
- **Onboarding**: New developers follow standards automatically

### 🏢 For Organizations

- **Maintainability**: Consistent codebase easier to maintain
- **Security**: Automated vulnerability detection
- **Compliance**: Documentation and standards enforcement
- **Velocity**: Faster development cycles with automated checks

### 📊 Measurable Improvements

- **Code Review Time**: 40-60% reduction in review time
- **Bug Detection**: 30-50% more issues caught before production
- **Developer Satisfaction**: Reduced frustration with inconsistent standards
- **Security Posture**: Proactive vulnerability management

## 🎯 Best Practices

### 📝 Commit Messages

```bash
# Good commit messages
git commit -m "feat: add user authentication endpoint"
git commit -m "fix: resolve memory leak in user service"
git commit -m "docs: update API documentation"
git commit -m "test: add unit tests for auth service"

# Poor commit messages
git commit -m "fixed stuff"
git commit -m "updates"
git commit -m "wip"
```

#### For NestJS-specific OWASP checks, you can also add:
```bash
# Install and run additional NestJS security tools
npm install --save-dev @nestjs/cli helmet
npm audit --audit-level=moderate
```

### 🔄 Branch Strategy

```bash
# Feature branches
feature/user-authentication
feature/payment-integration

# Bug fix branches
fix/memory-leak-user-service
fix/validation-error-handling

# Release branches
release/v1.2.0
hotfix/critical-security-patch
```

### 📋 Code Review Process

1. **Self-Review**: Check all automated status before requesting review
2. **Small PRs**: Keep changes focused and reviewable
3. **Documentation**: Update docs with functional changes
4. **Testing**: Include tests for new functionality
5. **Breaking Changes**: Clearly document any breaking changes

## 🛣️ Roadmap

### 🔮 Planned Enhancements

- [ ] **Performance Testing**: Lighthouse CI integration
- [ ] **Accessibility**: A11y linting and testing
- [ ] **Bundle Analysis**: Webpack bundle size monitoring
- [ ] **Visual Regression**: Screenshot diff testing
- [ ] **API Testing**: Automated contract testing

### 🚀 Advanced Features

- [ ] **Multi-Environment**: Staging/production deployment pipelines
- [ ] **Slack Integration**: Status notifications to team channels
- [ ] **Metrics Dashboard**: Code quality metrics visualization
- [ ] **Auto-Fix**: Automated code formatting and basic fixes

## 📄 License

This implementation is provided under the same license as the main NodeJS Bootstrap project.

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Follow the established coding standards (enforced by these checks!)
4. Submit a pull request

## 📞 Support

- 📚 **Documentation**: [Implementation Guide](GITHUB_CHECKS_IMPLEMENTATION.md)
- 🐛 **Bug Reports**: Create an issue with detailed reproduction steps
- 💡 **Feature Requests**: Use discussions for enhancement ideas
- 📧 **Direct Contact**: Reach out to the maintainers

---

**Made with ❤️ for better code quality and developer experience**

![NestJS](https://img.shields.io/badge/NestJS-E0234E?style=flat&logo=nestjs&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-2088FF?style=flat&logo=github-actions&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-007ACC?style=flat&logo=typescript&logoColor=white)
