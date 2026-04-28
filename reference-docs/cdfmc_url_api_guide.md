
#################################################################

To identify the URL or API endpoint for your Cloud-delivered Firewall Management Center (cdFMC), you need to access it through the Cisco Defense Orchestrator (CDO) portal. Unlike an on-premises FMC, cdFMC does not have a static IP address that you manage; instead, it is accessed via a specific cloud-hosted URL.


Here is how you can find the correct information:


1. Access via Cisco Defense Orchestrator (CDO)

The cdFMC is managed as a service within CDO.


Log in to your Cisco Defense Orchestrator portal (or the regional URL provided to your organization).
Once logged in, look for the Firepower Management Center tab or the specific cdFMC instance in your device inventory.
When you click to launch the FMC, your browser will navigate to the specific URL for your tenant. This URL typically follows a format like: https://fmc-prod-xxxx.cisco.com.

2. Identifying the API Endpoint for Automation

When using Ansible or Python to automate your cdFMC, the API endpoint is generally the same base URL you see in your browser when you are logged into the FMC interface.


UI URL: https://fmc-prod-xxxx.cisco.com/ui/login
API Base URL: https://fmc-prod-xxxx.cisco.com

Important: When configuring your group_vars/all.yml for Ansible, you should use the base URL (without the /ui/login suffix).


3. How to verify the API connectivity

If you are unsure if you have the correct URL, you can perform a simple connectivity test using curl from your terminal before running your Ansible playbooks:


bash
Copy Code
# Replace the URL with your specific cdFMC tenant URL
curl -k -v https://fmc-prod-xxxx.cisco.com/api/fmc_platform/v1/info

If successful: You will receive a JSON response containing information about the FMC version and API status.
If it fails: Double-check the URL for any typos or ensure your network/firewall allows outbound access to the Cisco cloud infrastructure.

4. Important Considerations for cdFMC

Region: Depending on your organization's setup, your URL might differ based on the region (e.g., US, EU, or APJ). Always copy the URL directly from your browser's address bar after launching the FMC from the CDO portal.
API Authentication: cdFMC uses the same API authentication process as on-premises FMC. You will need to generate an API token or use your credentials to obtain an authentication header.
Domain UUID: For cdFMC, the domain_uuid is mandatory for all API calls. If you are having trouble finding it, you can retrieve it by hitting the /api/fmc_platform/v1/info/domain endpoint after successfully authenticating.

Note: If you are still unable to locate the URL, please check with your organization's CDO Administrator, as they have the primary visibility into the tenant's provisioning details.

#################################################################



# Cisco cdFMC URL & API Endpoint Guide

## Overview

To identify the URL or API endpoint for your **Cloud-delivered Firewall Management Center (cdFMC)**, you must access it via the **Cisco Defense Orchestrator (CDO)** portal.

Unlike on-prem FMC, cdFMC:
- Does **not have a static IP**
- Is accessed via a **cloud-hosted URL**
- Is **tenant-specific**

---

## 1. Access via Cisco Defense Orchestrator (CDO)

Follow these steps:

1. Log in to your **Cisco Defense Orchestrator (CDO)** portal  
   *(Use your regional URL provided by your organisation)*  

2. Navigate to:
   - **Firepower Management Center tab**
   - Or locate your **cdFMC instance** under device inventory  

3. Click **Launch FMC**

4. Your browser will redirect to a URL like:

```
https://fmc-prod-xxxx.cisco.com
```

---

## 2. API Endpoint for Automation

When using **Ansible or Python**, the API endpoint is based on the same URL used in the browser.

### Example:

- **UI URL:**
```
https://fmc-prod-xxxx.cisco.com/ui/login
```

- **API Base URL:**
```
https://fmc-prod-xxxx.cisco.com
```

### Important

When configuring Ansible:

```yaml
fmc_hostname: "https://fmc-prod-xxxx.cisco.com"
```

❌ Do NOT include:
```
/ui/login
```

---

## 3. Verify API Connectivity

Before running automation scripts, validate connectivity using `curl`.

### Command:

```bash
curl -k -v https://fmc-prod-xxxx.cisco.com/api/fmc_platform/v1/info
```

### Expected Results

✅ **Success:**
- Returns JSON output
- Displays FMC version and API status

❌ **Failure:**
- Check URL for typos
- Ensure outbound access to Cisco cloud is allowed
- Verify firewall/proxy settings

---

## 4. Important cdFMC Considerations

### 🌍 Region Awareness

Your URL may vary based on region:

- US
- EU
- APJ

👉 Always copy the URL directly from your browser after launching FMC via CDO.

---

### 🔐 API Authentication

cdFMC uses the same authentication as on-prem FMC:

- Username/password login
- Token-based authentication

You must:
1. Authenticate
2. Retrieve API token
3. Use token in headers

---

### 🆔 Domain UUID (Mandatory)

For cdFMC, **domain_uuid is required for ALL API calls**.

### How to retrieve:

```bash
GET /api/fmc_platform/v1/info/domain
```

---

## 5. Troubleshooting

### Cannot find URL?

- Confirm access via CDO portal
- Check with your **CDO Administrator**
- Verify tenant provisioning

---

### API not responding?

- Validate:
  - URL format
  - Network access
  - SSL handling (`-k` for testing)

---

## 6. Summary

| Item | Value |
|------|------|
| Access Method | CDO Portal |
| FMC URL | Cloud-hosted |
| API Base URL | Same as FMC URL |
| Authentication | Token / Credentials |
| Domain UUID | Required |
| Connectivity Test | curl |

---

## 7. Recommended Next Step

Run:

```bash
curl -k https://fmc-prod-xxxx.cisco.com/api/fmc_platform/v1/info
```

Then integrate into:

- Ansible (`group_vars/all.yml`)
- Python scripts
- Automation pipelines

---