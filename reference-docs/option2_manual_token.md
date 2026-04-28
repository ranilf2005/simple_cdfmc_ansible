# Option 2 — Manual Token Handling (Advanced / Debug / Custom API)

## Overview
Manual token handling is used when:
- Debugging API issues
- Using `uri` module
- Learning API workflow
- Unsupported operations in Ansible module

---

## Step 1 — Get Token

```yaml
- name: Authenticate and get token
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

## Step 2 — Extract Token

```yaml
- set_fact:
    access_token: "{{ token_response.x_auth_access_token }}"
    refresh_token: "{{ token_response.x_auth_refresh_token }}"
```

---

## Step 3 — Use Token

```yaml
- name: Use token in API call
  uri:
    url: "{{ fmc_hostname }}/api/fmc_platform/v1/info/domain"
    method: GET
    headers:
      X-auth-access-token: "{{ access_token }}"
    validate_certs: no
  register: result
```

---

## Step 4 — Refresh Token (Optional)

```yaml
- name: Refresh token
  uri:
    url: "{{ fmc_hostname }}/api/fmc_platform/v1/auth/refreshtoken"
    method: POST
    headers:
      X-auth-access-token: "{{ access_token }}"
      X-auth-refresh-token: "{{ refresh_token }}"
```

---

## Considerations

- More complex
- Requires manual handling
- Risk of token expiry issues
- Not recommended for production
