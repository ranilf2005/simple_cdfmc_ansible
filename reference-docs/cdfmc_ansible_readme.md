# 🚀 Cisco cdFMC Automation with Ansible — Professional README

## 📌 Overview

This guide provides a **complete, production-ready approach** for automating  
**Cisco Cloud-delivered Firewall Management Center (cdFMC)** using **Ansible**.

It covers:

- ✅ Architecture (CDO → cdFMC → API → Ansible)  
- ✅ Recommended approach (auto token handling)  
- ✅ Advanced approach (manual token handling)  
- ✅ Security and best practices  

---

# 🏗️ Architecture Overview

## 🔷 High-Level Flow

```
+----------------------+
|  Cisco Defense       |
|  Orchestrator (CDO)  |
+----------+-----------+
           |
           v
+----------------------+
|   cdFMC (Cloud FMC)  |
|  fmc-prod-xxxx.cisco |
+----------+-----------+
           |
           v
+----------------------+
|   FMC REST API       |
|  /api/fmc_platform   |
+----------+-----------+
           |
           v
+----------------------+
|    Ansible Control   |
| (cisco.fmcansible)   |
+----------------------+
```

---

## 🔷 Detailed Flow

```
User / Automation Pipeline
        |
        v
Ansible Playbook
        |
        v
cisco.fmcansible Module
        |
        v
(Internal Authentication)
POST /auth/generatetoken
        |
        v
Token Received (Hidden)
        |
        v
API Calls with Token
        |
        v
cdFMC Executes Action
```

---

# ✅ Option 1 — Recommended (Auto Token Handling)

## ✔️ Why This Approach

The Ansible module:

- Automatically authenticates  
- Retrieves API token  
- Uses token internally  
- Refreshes token automatically  

👉 **No manual token handling required**

---

## 🔐 Variables File

```yaml
fmc_hostname: "https://fmc-prod-xxxx.cisco.com"
fmc_username: "your_username"
fmc_password: "your_password"
fmc_verify_ssl: false
domain_uuid: "your-domain-uuid"
```

---

## ▶️ Sample Playbook

```yaml
---
- name: Get Network Objects
  hosts: localhost
  gather_facts: false

  vars_files:
    - ../group_vars/all.yml

  tasks:
    - name: Retrieve network objects
      cisco.fmcansible.fmc_configuration:
        operation: "getAllNetworkObject"
        path_params:
          domainUUID: "{{ domain_uuid }}"
        query_params:
          limit: 5
      register: result

    - debug:
        var: result
```

---

## ⚙️ What Happens Internally

1. Authenticate using credentials  
2. Generate API token  
3. Store token internally  
4. Execute API calls  
5. Refresh token if needed  

---

## 🎯 Benefits

- Clean playbooks  
- No token management  
- Production ready  
- Secure handling  

---

# ⚠️ Option 2 — Manual Token Handling (Advanced)

## 🔍 When to Use

- Debugging API issues  
- Using `uri` module  
- Custom API workflows  
- Learning API internals  

---

## 🔐 Step 1 — Generate Token

```yaml
- name: Get token
  uri:
    url: "{{ fmc_hostname }}/api/fmc_platform/v1/auth/generatetoken"
    method: POST
    user: "{{ fmc_username }}"
    password: "{{ fmc_password }}"
    force_basic_auth: yes
    validate_certs: no
  register: token_response
```

---

## 🔎 Step 2 — Extract Token

```yaml
- set_fact:
    access_token: "{{ token_response.x_auth_access_token }}"
```

---

## 🔁 Step 3 — Use Token

```yaml
- name: Call API using token
  uri:
    url: "{{ fmc_hostname }}/api/fmc_platform/v1/info/domain"
    method: GET
    headers:
      X-auth-access-token: "{{ access_token }}"
    validate_certs: no
```

---

## ⚠️ Risks

- Token expiry issues  
- More complex logic  
- Harder to scale  
- Not ideal for production  

---

# 🧠 Why You DON’T Handle Tokens Manually

## 🔑 Key Concept

| Item | Purpose |
|------|--------|
| Username/password | Identity verification |
| Token | Temporary session |

---

## 🔄 Real Flow

```
Username/Password
        ↓
Token Generated
        ↓
Token Used for API
        ↓
Token Refreshed
```

---

## ❌ Problems with Manual Tokens

### Complexity
- Extra tasks required  

### Errors
- Expired tokens break automation  

### Scalability
- Hard to maintain in pipelines  

---

## ✅ Benefits of Module

- Automatic authentication  
- Automatic token refresh  
- Cleaner code  
- Secure handling  

---

# 🔐 Security Best Practices

## ❌ Avoid

```yaml
fmc_password: "plaintext"
```

---

## ✅ Use Ansible Vault

```bash
ansible-vault create group_vars/vault.yml
```

---

## ✅ Use Environment Variables

```yaml
fmc_password: "{{ lookup('env', 'FMC_PASSWORD') }}"
```

---

## ✅ Use Secret Managers

- HashiCorp Vault  
- AWS Secrets Manager  
- Azure Key Vault  

---

# 🧪 Recommended Execution Order

```bash
ansible --version
ansible-galaxy collection install cisco.fmcansible
ansible all -m ping
ansible-playbook get_network_objects.yml
```

---

# 📊 Summary

| Approach | Recommended | Complexity | Use Case |
|----------|------------|------------|----------|
| Auto Token (Module) | ✅ Yes | Low | Production |
| Manual Token | ⚠️ No | High | Debug/Learning |

---