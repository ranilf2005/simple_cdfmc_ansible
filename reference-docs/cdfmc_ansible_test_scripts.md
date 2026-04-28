# Cisco cdFMC Ansible Test Scripts

> **Purpose:** Validate Cisco cdFMC or on-premises FMC automation using Ansible and the `cisco.fmcansible` collection.

---

## 1. Overview

To interact with **Cisco cdFMC** or **on-premises FMC** using Ansible, use the Cisco Ansible collection:

```bash
cisco.fmcansible
```

This collection provides the `fmc_configuration` module, which maps Ansible tasks to FMC REST API operations.

---

## 2. Important Fixes Applied

| Get network objects operation | `getAllNetworkObjects` | `getAllNetworkObject` |
| Create network object operation | `createNetworkObject` | `createMultipleNetworkObject` |
| Result handling | `result.data` | Use `register_as` or display full result safely |
| Variables loading | Relative `vars_files` only | Kept simple, but added safer guidance |
| cdFMC guidance | Basic | Added domain UUID, SSL, API limit and secret handling notes |

---

## 3. Recommended Project Structure

Create this structure:

```text
secure-firewall-ansible/
├── ansible/
│   ├── inventory.yml
│   ├── group_vars/
│   │   └── all.yml
│   └── playbooks/
│       ├── get_domain.yml
│       ├── get_network_objects.yml
│       └── create_network_object.yml
└── README.md
```

---

## 4. Install Prerequisites

### 4.1 Install Ansible

```bash
pip install ansible
```

Verify:

```bash
ansible --version
```

---

### 4.2 Install Cisco FMC Ansible Collection

```bash
ansible-galaxy collection install cisco.fmcansible
```

Verify:

```bash
ansible-galaxy collection list | grep fmc
```

Expected output should include:

```text
cisco.fmcansible
```

---

## 5. Inventory File

**File:** `ansible/inventory.yml`

Because the playbooks call the FMC API from your local machine or jump host, use localhost.

```yaml
all:
  hosts:
    localhost:
      ansible_connection: local
```

---

## 6. Variables File

**File:** `ansible/group_vars/all.yml`

```yaml
---
fmc_hostname: "https://your-fmc-ip-or-fqdn"
fmc_username: "your_username"
fmc_password: "your_password"
fmc_verify_ssl: false
domain_uuid: "your-domain-uuid-here"
```

---

## 7. Security Note for Passwords

For lab testing, we can include usage of encrypted variables.

For customer, production, or shared environments, use **Ansible Vault**.

Create encrypted variables:

```bash
ansible-vault create ansible/group_vars/vault.yml
```

Example secure content:

```yaml
vault_fmc_password: "your_password"
```

Then reference it from `all.yml`:

```yaml
fmc_password: "{{ vault_fmc_password }}"
```

Run playbooks with:

```bash
ansible-playbook --ask-vault-pass -i ansible/inventory.yml ansible/playbooks/get_network_objects.yml
```

---

## 8. First Test — Ansible Local Ping

Run:

```bash
ansible all -i ansible/inventory.yml -m ping
```

Expected output:

```json
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

This confirms Ansible can run locally.

> This does not test FMC connectivity yet.

---

## 9. Playbook 1 — Get FMC Domains

Use this first if you do not know your `domain_uuid`.

**File:** `ansible/playbooks/get_domain.yml`

```yaml
---
- name: Get FMC Domains
  hosts: localhost
  gather_facts: false

  vars_files:
    - ../group_vars/all.yml

  tasks:
    - name: Retrieve FMC domains
      cisco.fmcansible.fmc_configuration:
        operation: "getAllDomain"
        register_as: "domains"

    - name: Display FMC domains
      debug:
        var: domains
```

Run:

```bash
ansible-playbook -i ansible/inventory.yml ansible/playbooks/get_domain.yml
```

Look for the domain UUID in the output, then update:

```yaml
domain_uuid: "your-real-domain-uuid"
```

---

## 10. Playbook 2 — Get Network Objects

This is a safe read-only test.

**File:** `ansible/playbooks/get_network_objects.yml`

```yaml
---
- name: Get Network Objects from FMC
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
          limit: 10
          expanded: true
        register_as: "network_objects"

    - name: Display network objects
      debug:
        var: network_objects
```

Run:

```bash
ansible-playbook -i ansible/inventory.yml ansible/playbooks/get_network_objects.yml
```

---

## 11. Playbook 3 — Create a Network Object

This is a write test. Use a unique object name to avoid duplicate object errors.

**File:** `ansible/playbooks/create_network_object.yml`

```yaml
---
- name: Create a Network Object in FMC
  hosts: localhost
  gather_facts: false

  vars_files:
    - ../group_vars/all.yml

  tasks:
    - name: Create new network object
      cisco.fmcansible.fmc_configuration:
        operation: "createMultipleNetworkObject"
        path_params:
          domainUUID: "{{ domain_uuid }}"
        query_params:
          bulk: false
        data:
          name: "ANSIBLE_TEST_OBJ_01"
          type: "Network"
          value: "192.168.100.0/24"
          description: "Created via Ansible Automation"
          overridable: false
        register_as: "created_network_object"

    - name: Display created object
      debug:
        var: created_network_object
```

Run:

```bash
ansible-playbook -i ansible/inventory.yml ansible/playbooks/create_network_object.yml
```

---

## 12. Safer Version — Create Object Only If Missing

This avoids failure when the object already exists.

**File:** `ansible/playbooks/create_network_object_safe.yml`

```yaml
---
- name: Safely Create a Network Object in FMC
  hosts: localhost
  gather_facts: false

  vars_files:
    - ../group_vars/all.yml

  vars:
    test_object_name: "ANSIBLE_TEST_OBJ_01"
    test_object_value: "192.168.100.0/24"

  tasks:
    - name: Search for existing object by name or value
      cisco.fmcansible.fmc_configuration:
        operation: "getAllNetworkObject"
        path_params:
          domainUUID: "{{ domain_uuid }}"
        query_params:
          filter: "nameOrValue:{{ test_object_name }}"
          limit: 10
          expanded: true
        register_as: "existing_network_objects"

    - name: Show existing matching objects
      debug:
        var: existing_network_objects

    - name: Create object only if no matching object is found
      cisco.fmcansible.fmc_configuration:
        operation: "createMultipleNetworkObject"
        path_params:
          domainUUID: "{{ domain_uuid }}"
        query_params:
          bulk: false
        data:
          name: "{{ test_object_name }}"
          type: "Network"
          value: "{{ test_object_value }}"
          description: "Created safely via Ansible Automation"
          overridable: false
        register_as: "created_network_object"
      when: existing_network_objects | length == 0

    - name: Display created object
      debug:
        var: created_network_object
      when: created_network_object is defined
```

Run:

```bash
ansible-playbook -i ansible/inventory.yml ansible/playbooks/create_network_object_safe.yml
```

> **Note:** Depending on the returned object structure from your FMC or cdFMC version, the `when` condition may need to be adjusted. If this playbook skips or fails unexpectedly, first run the read-only playbook and inspect the returned structure.

---

## 13. Recommended Test Order

Run these commands in order:

```bash
ansible --version
ansible-galaxy collection install cisco.fmcansible
ansible all -i ansible/inventory.yml -m ping
ansible-playbook -i ansible/inventory.yml ansible/playbooks/get_domain.yml
ansible-playbook -i ansible/inventory.yml ansible/playbooks/get_network_objects.yml
ansible-playbook -i ansible/inventory.yml ansible/playbooks/create_network_object.yml
```

---

## 14. cdFMC-Specific Considerations

### 14.1 Domain UUID

cdFMC requires the correct `domainUUID`.

You can get it from:

1. FMC GUI URL
2. FMC API Explorer
3. The `get_domain.yml` playbook

---

### 14.2 SSL Verification

For lab testing with self-signed certificates:

```yaml
fmc_verify_ssl: false
```

For production:

```yaml
fmc_verify_ssl: true
```

---

### 14.3 API Limits

When retrieving objects, always use a limit during testing:

```yaml
query_params:
  limit: 10
```

For large environments, use:

```yaml
query_params:
  limit: 100
  offset: 0
```

---

### 14.4 Object Naming

Avoid generic names such as:

```text
TEST
Network1
Object1
```

Use a clear lab naming convention:

```text
ANSIBLE_TEST_NET_01
ANSIBLE_TEST_HOST_01
ANSIBLE_TEST_GROUP_01
```

---

## 15. How to Verify Success

### 15.1 From Ansible Output

You should see:

```text
ok=...
changed=...
failed=0
```

For read-only tests:

```text
changed=0
```

For create tests:

```text
changed=1
```

---

### 15.2 From FMC GUI

Go to:

```text
Objects > Object Management > Network
```

Search for:

```text
ANSIBLE_TEST_OBJ_01
```

---

### 15.3 From Another Read Test

Run again:

```bash
ansible-playbook -i ansible/inventory.yml ansible/playbooks/get_network_objects.yml
```

Confirm the new object is returned.

---

## 16. Common Errors and Fixes

### Error: Collection Not Found

```text
couldn't resolve module/action 'cisco.fmcansible.fmc_configuration'
```

Fix:

```bash
ansible-galaxy collection install cisco.fmcansible
```

---

### Error: Wrong Operation Name

```text
Invalid operation name
```

Fix:

Use:

```yaml
operation: "getAllNetworkObject"
```

Not:

```yaml
operation: "getAllNetworkObjects"
```

Use:

```yaml
operation: "createMultipleNetworkObject"
```

Not:

```yaml
operation: "createNetworkObject"
```

---

### Error: Authentication Failure

Check:

```yaml
fmc_hostname
fmc_username
fmc_password
```

Also confirm your account has API access.

---

### Error: Domain UUID Missing or Invalid

Fix:

```bash
ansible-playbook -i ansible/inventory.yml ansible/playbooks/get_domain.yml
```

Copy the correct UUID into:

```yaml
domain_uuid: "your-real-domain-uuid"
```

---

### Error: Duplicate Object Already Exists

Fix:

Change:

```yaml
name: "ANSIBLE_TEST_OBJ_01"
```

To something unique:

```yaml
name: "ANSIBLE_TEST_OBJ_02"
```

---

### Error: SSL Certificate Validation Failed

For lab testing:

```yaml
fmc_verify_ssl: false
```

For production, install and trust the correct certificate chain.

---

## 17. Final Checklist

- [ ] Ansible installed
- [ ] `cisco.fmcansible` collection installed
- [ ] `inventory.yml` created
- [ ] `group_vars/all.yml` created
- [ ] FMC hostname configured
- [ ] FMC credentials configured
- [ ] Domain UUID configured
- [ ] Local Ansible ping works
- [ ] `get_domain.yml` works
- [ ] `get_network_objects.yml` works
- [ ] `create_network_object.yml` works
- [ ] Object visible in FMC GUI

---

## 18. Final Recommendation

Start with read-only playbooks first:

```bash
ansible-playbook -i ansible/inventory.yml ansible/playbooks/get_domain.yml
ansible-playbook -i ansible/inventory.yml ansible/playbooks/get_network_objects.yml
```

Only after read-only tests are successful, run the write test:

```bash
ansible-playbook -i ansible/inventory.yml ansible/playbooks/create_network_object.yml
```

---
