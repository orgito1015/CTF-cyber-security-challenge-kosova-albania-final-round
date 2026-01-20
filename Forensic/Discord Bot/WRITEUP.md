# Discord Bot - Forensic Challenge Writeup

## Challenge Information
- **Category:** Forensic
- **Points:** 150
- **Difficulty:** Easy-Medium

## Challenge Description
A developer has created a Discord bot and pushed the code to GitHub. However, they may have accidentally committed sensitive information that was later removed. Your task is to investigate the repository's commit history to find the hidden flag. This challenge demonstrates the importance of proper secret management and the permanence of Git history.

## Tools Required
- **git** - Version control system for examining repository history
- **Web Browser** - To view GitHub repository (optional)
- **GitHub CLI (gh)** - Optional tool for GitHub operations

## Methodology

### Step 1: Understand Git History
Git maintains a complete history of all changes made to a repository. Even if sensitive data is removed in a later commit, it remains accessible in the commit history. This is why secrets should never be committed to version control.

### Step 2: Clone or Access the Repository
If you have a local copy:
```bash
cd discord-bot-directory
```

If you have a GitHub URL:
```bash
git clone <repository-url>
cd repository-name
```

### Step 3: Examine the Commit History
View all commits in the repository:
```bash
git log --oneline --all
```

For more detailed information:
```bash
git log --all --full-history
```

### Step 4: Search Commit Messages for Clues
Look for commits that mention removing secrets, fixing security issues, or similar:
```bash
git log --all --grep="secret"
git log --all --grep="remove"
git log --all --grep="fix"
git log --all --grep="flag"
```

### Step 5: Examine File Changes in History
Check what files have been modified throughout history:
```bash
git log --all --full-history --name-only
```

### Step 6: View Specific Commit Contents
For each suspicious commit, view its contents:
```bash
git show <commit-hash>
```

Or check specific files across all history:
```bash
git log --all --full-history -- config.json
git log --all --full-history -- .env
git log --all --full-history -- secrets.txt
```

### Step 7: Search for the Flag Pattern
Search the entire Git history for the flag format:
```bash
git log -p --all | grep -i "CSC25"
```

Or search for "flag" keyword:
```bash
git log -p --all | grep -i "flag"
```

### Step 8: Check Deleted Files
View all files that were deleted from the repository:
```bash
git log --diff-filter=D --summary
```

Then view the contents before deletion:
```bash
git show <commit-hash>~1:<filename>
```

### Alternative: GitHub Web Interface
If the repository is on GitHub:
1. Navigate to the repository
2. Click on "Commits" to view commit history
3. Browse through commits looking for suspicious changes
4. Click on individual commits to see what was added/removed
5. Look for commits that remove sensitive files or data

## Solution
By examining the Git commit history, we discover that the developer initially committed a configuration file or secrets file containing the Discord bot token or flag, then removed it in a subsequent commit. However, the flag remains in the Git history.

**Flag:** `CSC25{Tr3a5urE_13fT_b3H1nD}`

The flag name itself is a hint - "treasure left behind" - referring to sensitive data left behind in Git history.

## Why This Vulnerability Exists

### The Git History Problem
1. **Immutable History**: Git is designed to preserve complete history for versioning purposes
2. **Distributed Nature**: Once code is pushed, it's cloned to many locations
3. **False Sense of Security**: Developers may think deleting a file removes it completely
4. **Accidental Commits**: Easy to commit sensitive files before adding them to .gitignore

### Common Scenarios
- Committing `.env` files with API keys
- Including `config.json` with database credentials
- Accidentally committing IDE configuration with secrets
- Testing with real credentials in code
- Forgetting to add sensitive files to `.gitignore`

### Why Simple Deletion Doesn't Work
When you commit sensitive data and then delete it:
```bash
git add secrets.txt
git commit -m "Add configuration"
# Later realize the mistake
git rm secrets.txt
git commit -m "Remove secrets"
```

The secrets are still in commit #1's history and can be retrieved by anyone with repository access.

## Key Takeaways
1. **Git history is permanent** - Deleted files can be recovered from commit history
2. **Never commit secrets** - Use environment variables and secret management tools instead
3. **One mistake is enough** - Even if removed immediately, secrets in history are compromised
4. **Public repositories amplify risk** - Anyone can clone and search history
5. **Assume breach** - If secrets were committed, rotate them immediately

## Security Recommendations

### Prevention - Before Committing
- **Use .gitignore** - Add sensitive file patterns before first commit:
  ```
  .env
  .env.local
  config/secrets.yml
  *.key
  *.pem
  credentials.json
  ```
- **Pre-commit Hooks** - Install tools like:
  - `git-secrets` - Prevents committing credentials
  - `detect-secrets` - Scans for high-entropy strings
  - `gitleaks` - Detects hardcoded secrets
  - `trufflehog` - Finds secrets in commit history

### Environment-Based Secret Management
- Use environment variables: `process.env.DISCORD_TOKEN`
- Use secret management services:
  - **HashiCorp Vault**
  - **AWS Secrets Manager**
  - **Azure Key Vault**
  - **Doppler**
- Use `.env` files locally (in .gitignore)
- Use platform-specific secrets (GitHub Secrets, GitLab CI/CD variables)

### If Secrets Are Accidentally Committed

#### Immediate Actions (High Priority)
1. **Rotate compromised secrets immediately** - Consider them public
2. **Revoke API tokens and keys**
3. **Change passwords**
4. **Review access logs** for unauthorized usage

#### Clean Git History (After Rotation)
⚠️ **WARNING**: These commands rewrite history and break existing clones

```bash
# Using git-filter-repo (recommended)
pip install git-filter-repo
git filter-repo --path sensitive-file.txt --invert-paths

# Using BFG Repo-Cleaner
bfg --delete-files sensitive-file.txt

# Force push (breaks existing clones)
git push origin --force --all
```

#### For Public Repositories
- If secrets were in a public repo, **assume they are compromised**
- History rewriting doesn't help (repo may be forked)
- Focus on rotating secrets and monitoring for abuse

### Development Best Practices
- **Separate development and production secrets**
- **Use different API keys for testing**
- **Implement secrets scanning in CI/CD pipeline**
- **Regular security audits of repositories**
- **Security training for developers**

### Git Configuration
Enable git-secrets globally:
```bash
git secrets --install
git secrets --register-aws
```

### Code Review Process
- Review `.gitignore` in every new repository
- Check for secrets before approving pull requests
- Use automated tools in PR checks

## Educational Value
This challenge teaches:
- **Git forensics** - How to investigate repository history
- **Secret management** - Why secrets should never be in version control
- **Security awareness** - Understanding the permanence of Git commits
- **Real-world scenario** - This is a common mistake with serious consequences
- **Incident response** - How to handle accidentally committed secrets

### Real-World Impact
Many major security incidents have occurred due to secrets in Git history:
- API keys for cloud services leading to cryptocurrency mining
- Database credentials causing data breaches
- OAuth tokens enabling account takeovers
- Private keys allowing infrastructure compromise

This challenge demonstrates that even after removing sensitive data, the evidence remains accessible to anyone who knows where to look.
