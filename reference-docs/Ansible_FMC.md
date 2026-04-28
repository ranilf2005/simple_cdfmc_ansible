# Ansible Automation with FMC

This use case demonstrates how to automate Cisco FMC using Ansible playbooks.

---

## 🔄 Workflow

Ansible Playbook  
↓  
FMC REST API  
↓  
Read or Create Objects  
↓  
Verify in FMC  

---

## 1️⃣ What USE CASE 7 Proves

If this works, it proves:

- Your Ansible environment is installed correctly  
- The Cisco FMC Ansible collection is working  
- Your credentials and FMC connectivity are working  
- You can automate FMC using both Python and Ansible  

---

## 2️⃣ Where to Run It

Run everything from the project root:

```bash
cd ~/lab/secure-firewall-automation
source fw/bin/activate
```

---

## 3️⃣ Check Ansible Installation

```bash
ansible --version
```

If not installed:

```bash
pip install ansible
```

---

## 4️⃣ Install Cisco FMC Collection

```bash
ansible-galaxy collection install cisco.fmcansible
```

Verify:

```bash
ansible-galaxy collection list | grep fmc
```

---

## 5️⃣ Project Structure

```
ansible/
├── inventory.yml
├── group_vars/
│   └── all.yml
└── playbooks/
    ├── get_domain.yml
    ├── get_network_objects.yml
    ├── get_security_zones.yml
    └── create_network_objects.yml
```

---

## 6️⃣ Inventory File

```yaml
all:
  hosts:
    localhost:
      ansible_connection: local
```

---

## 7️⃣ Variables File

```yaml
fmc_hostname: "https://192.168.30.184"
fmc_username: "admin"
fmc_password: "yourpassword"
fmc_verify_ssl: false
domain_uuid: "your-domain-uuid"
```

---

## 8️⃣ Test Ansible Ping

```bash
ansible all -i ansible/inventory.yml -m ping
```

Expected:

```json
localhost | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

---

## 9️⃣ Get Domains

```bash
ansible-playbook -i ansible/inventory.yml ansible/playbooks/get_domain.yml
```

---

## 🔟 Get Network Objects

```bash
ansible-playbook -i ansible/inventory.yml ansible/playbooks/get_network_objects.yml
```

---

## 1️⃣1️⃣ Get Security Zones

```bash
ansible-playbook -i ansible/inventory.yml ansible/playbooks/get_security_zones.yml
```

---

## 1️⃣2️⃣ Create Test Object

Example payload:

```yaml
objects_payload:
  - name: ANSIBLE_TEST_NET_01
    value: 10.99.99.0/24
    type: Network
    description: Test object created by Ansible
```

Run:

```bash
ansible-playbook -i ansible/inventory.yml ansible/playbooks/create_network_objects.yml
```

---

## 1️⃣3️⃣ Verification

- Check Ansible output  
- Check FMC GUI  
- Verify using Python inventory  

---

## 1️⃣4️⃣ Safe Execution Order

```bash
ansible --version
ansible-galaxy collection install cisco.fmcansible
ansible all -i ansible/inventory.yml -m ping
ansible-playbook -i ansible/inventory.yml ansible/playbooks/get_domain.yml
ansible-playbook -i ansible/inventory.yml ansible/playbooks/get_network_objects.yml
ansible-playbook -i ansible/inventory.yml ansible/playbooks/get_security_zones.yml
ansible-playbook -i ansible/inventory.yml ansible/playbooks/create_network_objects.yml
```

---

## 1️⃣5️⃣ Success Criteria

- No errors in Ansible  
- Read playbooks work  
- Object created successfully  
- Object visible in FMC  

---

## 1️⃣6️⃣ Common Errors

### Collection not found
```bash
ansible-galaxy collection install cisco.fmcansible
```

### Authentication failure
- Check username/password/hostname  

### Domain UUID missing
- Update group_vars/all.yml  

### Duplicate object
- Use unique object name  

### SSL error
```yaml
fmc_verify_ssl: false
```

---

## 1️⃣7️⃣ If Create Fails

- Check error logs  
- Validate payload  
- Ensure unique object name  

---

## 1️⃣8️⃣ Checklist

- [ ] Ansible installed  
- [ ] FMC collection installed  
- [ ] Inventory correct  
- [ ] Variables correct  
- [ ] Ping works  
- [ ] Read playbooks work  
- [ ] Create playbook works  

---

## 1️⃣9️⃣ Recommended Start

```bash
ansible --version
ansible-galaxy collection install cisco.fmcansible
ansible all -i ansible/inventory.yml -m ping
```
