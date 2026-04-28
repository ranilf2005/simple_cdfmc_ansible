# Option 1 — Recommended (Ansible Handles Token Automatically)

## Overview
When using the `cisco.fmcansible.fmc_configuration` module, **you do NOT need to manually handle API tokens**.

The module:
- Authenticates using username/password
- Retrieves API token automatically
- Uses token for all API calls
- Refreshes token when required

---

## Variables File

```yaml
fmc_hostname: "https://fmc-prod-xxxx.cisco.com"
fmc_username: "your_username"
fmc_password: "your_password"
fmc_verify_ssl: false
domain_uuid: "your-domain-uuid"
```

---

## Sample Playbook

```yaml
---
- name: Get Network Objects (Auto Authentication)
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

    - name: Show output
      debug:
        var: result
```

---

## What Happens Internally

1. Sends authentication request
2. Retrieves API token
3. Stores token securely
4. Uses token for API calls
5. Refreshes token automatically

---

## Benefits

- No manual token management
- Cleaner playbooks
- Reduced errors
- Production-ready approach
