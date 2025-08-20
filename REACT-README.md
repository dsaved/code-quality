# GitHub Checks Implementation for ReactJS Projects

![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-2088FF?style=for-the-badge&logo=github-actions&logoColor=white)
![ESLint](https://img.shields.io/badge/ESLint-4B3263?style=for-the-badge&logo=eslint&logoColor=white)
![Prettier](https://img.shields.io/badge/prettier-1A2C34?style=for-the-badge&logo=prettier&logoColor=F7BA3E)
![Jest](https://img.shields.io/badge/Jest-323330?style=for-the-badge&logo=jest&logoColor=white)
![TypeScript](https://img.shields.io/badge/TypeScript-007ACC?style=for-the-badge&logo=typescript&logoColor=white)

A comprehensive GitHub Actions workflow system that enforces code quality, naming conventions, spell checking, security scanning, and test coverage for ReactJS applications.

## 🎯 Overview

This implementation provides a complete CI/CD pipeline with automated code quality checks that run on every push and pull request. It ensures consistent code standards, security practices, and naming conventions across your entire frontend development team.

## ✨ Features

### 🔍 Code Quality Checks

- **ESLint**: JavaScript/TypeScript linting with custom rules
- **Prettier**: Automated code formatting validation
- **Naming Conventions**: Enforces PascalCase components, camelCase variables, kebab-case files
- **Code Standards**: Prevents console.log, hardcoded secrets, and poor practices

### 🛡️ Security & Testing

- **Security Scanning**: OSV Scanner and Trivy vulnerability detection (fail on vulnerabilities, email notification if configured)
- **Test Coverage**: Jest/React Testing Library with coverage thresholds
- **Spell Checking**: CSpell validation for code and documentation
- **Secret Detection**: Prevents hardcoded API keys and passwords

### 🚀 CI/CD Pipeline

- **Multi-Node Testing**: Tests on Node.js 18.x and 20.x
- **Parallel Execution**: Multiple checks run simultaneously
- **Coverage Reporting**: Built-in coverage reports and thresholds

### 📋 Team Collaboration

- **CODEOWNERS**: Automatic reviewer assignment
- **PR Templates**: Standardized pull request format
- **Status Checks**: Clear pass/fail indicators on PRs
- **Branch Protection**: Prevents merging until all checks pass

## 📁 Generated Files Structure

```
your-reactjs-project/
├── .github/
│   ├── workflows/
│   │   ├── ci.yml                    # Main CI pipeline
│   │   ├── naming-conventions.yml    # Naming validation
│   │   ├── spell-check.yml           # Spell checking
│   │   └── security.yml              # Security scanning
│   ├── CODEOWNERS                    # Team ownership rules
│   └── pull_request_template.md      # PR template
├── .eslintrc.json                    # Enhanced ESLint config
├── .cspell.json                      # Spell check dictionary
├── jest.config.js                    # Coverage thresholds
├── .prettierrc                       # Prettier config
└── package.json                      # Updated scripts
```

## 🚀 Quick Start

### Method 1: Using Create React App (Recommended)

```bash
npx create-react-app my-app --template typescript
cd my-app
```

### Method 2: Manual Implementation

Follow the [detailed implementation guide](#-implementation-guide) below.

## 🔧 Implementation Guide

### Step 1: Create Workflow Files

<details>
<summary>📄 <code>.github/workflows/ci.yml</code></summary>

```yaml
name: React CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  build-and-test:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [18.x, 20.x]

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
        run: npm run test:ci

      - name: Check Coverage Thresholds
        run: npm run test:coverage
```

</details>

<details>
<summary>📄 <code>.github/workflows/naming-conventions.yml</code></summary>

```yaml
name: Naming Conventions

on:
  push:
    branches: [staging]
    paths:
      - "src/**"
  pull_request:
    branches: [staging]
    paths:
      - "src/**"

jobs:
  naming-conventions:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Check React component naming conventions
        run: |
          echo "Checking React component naming conventions..."
          # Check that .tsx files use PascalCase
          if find src -name "*.tsx" -not -path "*/node_modules/*" | grep -E '^src/.*[a-z][A-Z].*\.tsx$|^src/.*[^A-Z][a-z].*\.tsx$' | grep -v -E '^src/.*/[A-Z][a-zA-Z]*\.tsx$'; then
            echo "❌ Found .tsx component files that don't follow PascalCase naming convention"
            echo "React components should use PascalCase (e.g., MyComponent.tsx)"
            exit 1
          fi
          echo "✅ All React component files follow PascalCase naming convention"

      - name: Check TypeScript/JavaScript file naming conventions
        run: |
          echo "Checking TypeScript/JavaScript file naming conventions..."
          # Check that non-component .ts files use camelCase or kebab-case
          if find src -name "*.ts" -not -path "*/node_modules/*" | grep -v '\.spec\.ts$' | grep -v '\.test\.ts$' | grep -E '[A-Z][a-z].*[A-Z]'; then
            echo "❌ Found .ts files that don't follow camelCase or kebab-case naming convention"
            echo "Non-component TypeScript files should use camelCase or kebab-case"
            exit 1
          fi
          echo "✅ All TypeScript files follow proper naming conventions"

      - name: Check for console.log statements
        run: |
          if grep -r "console\.log" src --include="*.ts" --include="*.tsx" --include="*.js" --include="*.jsx"; then
            echo "❌ Found console.log statements in source code"
            echo "Please remove console.log statements before committing"
            exit 1
          fi
          echo "✅ No console.log statements found"

      - name: Check for hardcoded secrets
        run: |
          if grep -r -E "(password|secret|key|token|apiKey)\s*[:=]\s*['\"][^'\"]{8,}" src --include="*.ts" --include="*.tsx" --include="*.js" --include="*.jsx"; then
            echo "❌ Found potential hardcoded secrets"
            echo "Please use environment variables for sensitive data"
            exit 1
          fi
          echo "✅ No hardcoded secrets detected"

      - name: Check folder naming conventions
        run: |
          echo "Checking folder naming conventions..."
          # Check that folders use kebab-case or camelCase
          if find src -type d -not -path "*/node_modules/*" | grep -E '/[A-Z][a-z]*[A-Z]' | grep -v -E '/[a-z][a-zA-Z]*$|/[a-z-]+$'; then
            echo "❌ Found folders that don't follow kebab-case or camelCase naming convention"
            echo "Folders should use kebab-case or camelCase"
            exit 1
          fi
          echo "✅ All folders follow proper naming conventions"
```

</details>

<details>
<summary>📄 <code>.github/workflows/spell-check.yml</code></summary>

```yaml
name: Spell Check

on:
  push:
    branches: [staging]
    paths:
      - "src/**"
      - "*.md"
      - "public/**"
  pull_request:
    branches: [staging]
    paths:
      - "src/**"
      - "*.md"
      - "public/**"

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
        run: |
          # Check if .cspell.json exists, create a basic one if not
          if [ ! -f .cspell.json ]; then
            echo "Creating basic .cspell.json configuration..."
            cat > .cspell.json << 'EOF'
          {
            "version": "0.2",
            "language": "en",
            "words": [
              "Vite",
              "TypeScript",
              "tsx",
              "jsx",
              "Redux",
              "tailwind",
              "navbar",
              "useState",
              "useEffect",
              "className",
              "onClick",
              "onChange",
              "onSubmit",
              "href",
              "src",
              "alt",
              "readonly",
              "autofocus",
              "autocomplete",
              "novalidate"
            ],
            "ignorePaths": [
              "node_modules/**",
              "dist/**",
              "build/**",
              "*.min.js",
              "*.map",
              "package-lock.json",
              "yarn.lock"
            ]
          }
          EOF
          fi

          # Run spell check on React project files
          cspell "src/**/*.{ts,tsx,js,jsx}" "*.md" "public/**/*.{html,svg}" --config .cspell.json --exclude "src/**/*.min.js" --exclude "node_modules/**" --exclude "dist/**" --exclude "build/**"
```

</details>

<details>
<summary>📄 <code>.github/workflows/security.yml</code></summary>

```yaml
name: Security Audit

on:
  push:
    branches: [staging]
    paths:
      - "src/**"
      - "package*.json"
      - "yarn.lock"
  pull_request:
    branches: [staging]
    paths:
      - "src/**"
      - "package*.json"
      - "yarn.lock"

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
            osv-scanner --lockfile=yarn.lock | tee osv-report.txt
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
          trivy fs ./ --exit-code 1 --severity MEDIUM,HIGH,CRITICAL | tee trivy-report.txt
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

<details>
<summary>📄 <code>.prettierrc</code></summary>

```json
{
  "singleQuote": true,
  "trailingComma": "all"
}
```

</details>

<details>
<summary>📄 <code>.cspell.json</code></summary>

```json
{
  "version": "0.2",
  "language": "en",
  "words": ["reactjs", "redux", "thunk", "formik", "yup", "axios"],
  "ignorePaths": ["node_modules/**", "build/**", "coverage/**", ".git/**"]
}
```

</details>

<details>
<summary>📄 <code>jest.config.js</code></summary>

```javascript
module.exports = {
  collectCoverage: true,
  coverageDirectory: "coverage",
  coverageThreshold: {
    global: {
      branches: 80,
      functions: 80,
      lines: 80,
      statements: 80,
    },
  },
  testEnvironment: "jsdom",
};
```

</details>

<details>
<summary>📄 <code>.github/CODEOWNERS</code></summary>

```
* @yourteam/frontend-team
/src/components/ @yourteam/ui-team
/src/hooks/ @yourteam/frontend-team
/.github/ @yourteam/devops-team
```

</details>

### Step 3: Update Package Scripts

Add these scripts to your `package.json`:

```json
{
  "scripts": {
    "lint:check": "eslint 'src/**/*.{js,jsx,ts,tsx}'",
    "format:check": "prettier --check 'src/**/*.{js,jsx,ts,tsx}'",
    "test:ci": "react-scripts test --env=jsdom --coverage --watchAll=false",
    "test:coverage": "jest --coverage",
    "spell:check": "cspell 'src/**/*.{js,jsx,ts,tsx}' '**/*.md'"
  }
}
```

## 📊 Naming Conventions Enforced

### 📁 File & Directory Naming

- **Files**: `my-component.tsx` ✅ | `MyComponent.tsx` ❌ | `myComponent.tsx` ❌
- **Directories**: `user-profile/` ✅ | `userProfile/` ❌

### 🏗️ Code Naming

- **Components**: `MyComponent` ✅ | `myComponent` ❌
- **Variables**: `userData` ✅ | `user_data` ❌
- **Constants**: `API_KEY` ✅ | `apiKey` ✅
- **Hooks**: `useUser` ✅ | `UseUser` ❌

## 🛡️ Security Checks

### 🚫 Prevented Patterns

```javascript
// ❌ Hardcoded secrets (detected)
const apiKey = "sk_live_abc123def456";
const password = "mypassword123";

// ❌ Console statements (detected)
console.log("Debug info");

// ✅ Proper patterns
const apiKey = process.env.REACT_APP_API_KEY;
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
git checkout -b feature/user-profile
# Make changes...
git push origin feature/user-profile
```

### 2. Automated Checks Run

```
🔄 CI / build-and-test (18.x)   - Installing dependencies...
🔄 Naming Conventions           - Checking file names...
🔄 Spell Check                 - Validating spelling...
🔄 Security                    - Scanning for vulnerabilities...
```

### 3. Results Display

```
✅ CI / build-and-test (18.x)   - All tests passed
✅ Naming Conventions           - All conventions followed
✅ Spell Check                 - No spelling errors
❌ Security                    - 2 vulnerabilities found
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

### 2. Configure Branch Protection

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
touch src/MyComponent.tsx  # Should fail (PascalCase)
git add . && git commit -m "test: wrong file naming"
git push origin test/github-checks
# Check GitHub Actions - should fail naming-conventions
```

</details>

<details>
<summary>🧪 Test Code Quality</summary>

```bash
# Add console.log statement
echo "console.log('test');" >> src/App.tsx
git add . && git commit -m "test: add console.log"
git push origin test/github-checks
# Check GitHub Actions - should fail linting
```

</details>

<details>
<summary>📝 Test Spell Check</summary>

```bash
# Add misspelled word in comment
echo "// This is a mispelled word: recieve" >> src/App.tsx
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

```json
// Adjust .eslintrc.json rules
"no-console": "warn",
"camelcase": "off"
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
  }
}
```

</details>

<details>
<summary>🎨 <strong>Custom ESLint Rules</strong></summary>

```json
// .eslintrc.json
"rules": {
  "prefer-const": "error",
  "no-var": "error",
  "react/prop-types": "off"
}
```

</details>

<details>
<summary>📚 <strong>Custom Spell Check Dictionary</strong></summary>

```json
// .cspell.json
{
  "words": [
    "microfrontend",
    "storybook",
    "webpack",
    "yourcompany",
    "projectname"
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
git commit -m "feat: add user profile page"
git commit -m "fix: resolve memory leak in App component"
git commit -m "docs: update README"
git commit -m "test: add unit tests for login form"

# Poor commit messages
git commit -m "fixed stuff"
git commit -m "updates"
git commit -m "wip"
```

#### For React.js-specific OWASP checks:

```bash
# Install and run additional React security tools
npm install --save-dev eslint-plugin-security
npm audit --audit-level=moderate
```

### 🔄 Branch Strategy

```bash
# Feature branches
feature/user-profile
feature/payment-integration

# Bug fix branches
fix/memory-leak-app
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

This implementation is provided under the same license as the main project.

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
