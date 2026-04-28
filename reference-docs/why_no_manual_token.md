# Why You DON’T Manually Handle Tokens (But Still Use Credentials)

## Key Concept

- Username/password = Identity verification
- Token = Temporary session

---

## How It Works

1. Ansible sends credentials
2. FMC generates token
3. Module stores token
4. Token used for API calls
5. Token refreshed automatically

---

## Why Not Manual Tokens

### Complexity
- Extra tasks required
- Token lifecycle management needed

### Errors
- Expired tokens cause failures
- Missing headers break automation

### Scalability Issues
- Hard to maintain in pipelines
- Difficult for teams

---

## Benefits of Using Module

- Automatic authentication
- Automatic token refresh
- Cleaner code
- More secure handling

---

## Best Practice

Use:
- Ansible module for automation

Avoid:
- Manual token handling unless necessary

---

## Enterprise Approach

Use secure credential storage:
- Ansible Vault
- Environment variables
- Secret managers (Vault, AWS, Azure)

---

## Executive Summary

“We use token-based authentication with automated lifecycle management via the Ansible module.”
