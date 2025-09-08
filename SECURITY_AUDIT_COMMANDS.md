# Security Audit Commands and Tools

This document contains the commands and tools used during the security audit of the Obsidian Google Drive plugin repository.

## Git History Analysis Commands

```bash
# Search for sensitive files in git history
git log --all --full-history -- "*secret*" "*credential*" "*.env*" "*token*" "*key*" "*id_rsa*" "*.pem"

# Search for leaked credentials in commit content
git log --all --full-history -p | grep -i -E "(client_secret|api_key|access_token|refresh_token|AIza|sk-|ghp_)"

# Search for files that may contain secrets
find . -name "*.env*" -o -name "*secret*" -o -name "*credential*" -o -name "*.pem" -o -name "*.key" -o -name "*id_rsa*" -o -name "*.bak" -o -name "dump.sql"
```

## Code Pattern Analysis

```bash
# Search for dangerous code patterns
grep -r -i -E "(eval|base64|btoa|atob|Function\(|new Function)" --include="*.ts" --include="*.js" --include="*.json" .

# Search for external URLs and network calls
grep -r -i -E "(https?://[^'\")]+)" --include="*.ts" --include="*.js" .

# Search for client credentials patterns
grep -r -i -E "(client_id|client_secret)" --include="*.ts" --include="*.js" .
```

## Dependency Analysis

```bash
# Check for vulnerabilities using yarn
yarn audit --level moderate

# Check for vulnerabilities using npm (requires package-lock.json)
npm audit --audit-level=moderate

# Generate package-lock.json if needed
npm i --package-lock-only
```

## Git Security Tools

Here are additional tools you can use to scan git repositories for secrets:

### git-secrets
```bash
# Install git-secrets
git clone https://github.com/awslabs/git-secrets.git
cd git-secrets && make install

# Configure git-secrets
git secrets --register-aws
git secrets --install

# Scan repository
git secrets --scan
git secrets --scan-history
```

### TruffleHog
```bash
# Install truffleHog
pip install truffleHog

# Scan repository
truffleHog --regex --entropy=False .
truffleHog --regex --entropy=False --branch=main .
```

### gitleaks
```bash
# Install gitleaks
# Download from https://github.com/gitleaks/gitleaks/releases

# Scan repository
gitleaks detect --source . --verbose
gitleaks protect --source . --verbose
```

## Git History Cleanup Commands

If secrets are found in git history, use these commands to clean them:

### git-filter-repo (Recommended)
```bash
# Install git-filter-repo
pip install git-filter-repo

# Remove files containing secrets
git filter-repo --path-glob '*secret*' --invert-paths
git filter-repo --path-glob '*.env*' --invert-paths

# Remove specific strings/patterns
git filter-repo --replace-text <(echo "SECRET_KEY==>SECRET_KEY=[REDACTED]")
```

### BFG Repo-Cleaner
```bash
# Download BFG Repo-Cleaner
wget https://repo1.maven.org/maven2/com/madgag/bfg/1.14.0/bfg-1.14.0.jar

# Remove files
java -jar bfg-1.14.0.jar --delete-files "*.env*"
java -jar bfg-1.14.0.jar --delete-files "*secret*"

# Replace text
java -jar bfg-1.14.0.jar --replace-text replacements.txt
```

## Security Best Practices Commands

### Enable git hooks for secret detection
```bash
# Create pre-commit hook
cat > .git/hooks/pre-commit << 'EOF'
#!/bin/bash
git secrets --pre_commit_hook -- "$@"
EOF

chmod +x .git/hooks/pre-commit
```

### Create .gitignore for sensitive files
```bash
# Add to .gitignore
cat >> .gitignore << 'EOF'
# Sensitive files
*.env
*.env.*
.env
.env.*
*secret*
*credential*
*.pem
*.key
*id_rsa*
*.bak
dump.sql
config.json
secrets.json
client_secret.json
credentials.json
EOF
```

## Dependency Security Scanning

### npm/yarn audit
```bash
# Check for known vulnerabilities
yarn audit
npm audit

# Fix automatically where possible
yarn audit fix
npm audit fix

# Generate detailed report
yarn audit --json > audit-report.json
npm audit --json > audit-report.json
```

### Snyk
```bash
# Install Snyk CLI
npm install -g snyk

# Authenticate
snyk auth

# Test for vulnerabilities
snyk test
snyk test --json > snyk-report.json

# Monitor project
snyk monitor
```

### Retire.js
```bash
# Install retire.js
npm install -g retire

# Scan for vulnerable JavaScript libraries
retire --js --outputformat json --outputpath retire-report.json
```

## Additional Security Tools

### Semgrep
```bash
# Install semgrep
pip install semgrep

# Run security rules
semgrep --config=auto .
semgrep --config=security .
```

### CodeQL
```bash
# Install CodeQL CLI
# Download from https://github.com/github/codeql-cli-binaries/releases

# Create database
codeql database create mydb --language=javascript

# Run queries
codeql database analyze mydb javascript-security-and-quality.qls --format=json --output=results.json
```

## Regular Security Maintenance

Set up these commands to run regularly:

```bash
#!/bin/bash
# security-check.sh - Run weekly security checks

echo "Running dependency audit..."
yarn audit

echo "Scanning for secrets..."
git secrets --scan

echo "Checking for sensitive files..."
find . -name "*.env*" -o -name "*secret*" -o -name "*credential*"

echo "Security check complete."
```

Make it executable and add to cron:
```bash
chmod +x security-check.sh
crontab -e
# Add: 0 9 * * 1 /path/to/security-check.sh
```