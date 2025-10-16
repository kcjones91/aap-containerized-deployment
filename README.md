# AAP Containerized Deployment

Minimal, GitHub‑ready guide to install/uninstall **Red Hat Ansible Automation Platform (AAP)** using the **containerized installer**.

---

## Overview

This repo deploys AAP to the local/target host using Ansible playbooks that live under:
`collections/ansible_collections/ansible/containerized_installer/playbooks/`

- **Install:** `install.yml`
- **Uninstall:** `uninstall.yml`

> ⚠️ **Protect secrets**: do **not** commit real passwords/tokens. Prefer **Ansible Vault** or **environment variables** (especially in CI).

---

## Prerequisites

- RHEL 9 (or compatible) with `ansible-core` and `podman` or `docker`
- Entitled **Red Hat registry** account (or service account token) for `registry.redhat.io`
- Shell access to the deployment host(s)

---

## Quick Start

### 1) Edit `inventory`

Replace `localhostnamehere` with your FQDN/IP. Keep `ansible_connection=local` when deploying on the same machine.

```ini
[automationcontroller]
localhostnamehere ansible_connection=local

[automationgateway]
localhostnamehere

[database]
localhostnamehere

[all:vars]
# Container registry creds (use your RH account or a service account token)
registry_url=registry.redhat.io
registry_username='redhat developer account here'
registry_password='redhat password here'
```

### 2) Create `vars.yml`

You can start from the **full example** below (also available as `vars.example.yml`). **Change the passwords/hosts** before use.

```yaml
# --- Registry (pull AAP images) ---
aap_registry_url: "{{ registry_url }}"
aap_registry_username: "{{ registry_username }}"
aap_registry_password: "{{ registry_password }}"

# --- General ---
aap_deployment_type: "standard"        # e.g., "standard", "controller-only"
aap_namespace: "aap"                   # k8s namespace/project for AAP

# --- Controller ---
controller_admin_password: "cora0410"  # CHANGE ME
controller_pg_host: "{{ groups['database'][0] }}"
controller_pg_password: "pgpassword"   # CHANGE ME

# --- Gateway ---
gateway_admin_password: "gatewayadminpassword"  # CHANGE ME
gateway_pg_host: "{{ groups['database'][0] }}"
gateway_pg_password: "gatewaypassword"          # CHANGE ME

# If using custom TLS for gateway, set cert/key paths and CA chain
# gateway_tls_cert: "/path/to/certs/gateway.crt"
# gateway_tls_key:  "/path/to/certs/gateway.key"
custom_ca_cert: "add certs path here ^ example above"
gateway_main_url: "https://<hostip>:8446"       # Set to your reachable URL:port

# --- Automation Hub ---
hub_admin_password: "hubpassword"     # CHANGE ME
hub_pg_host: "127.0.0.1"              # Or your DB host
hub_pg_password: "hubpgpassword"      # CHANGE ME

# --- Installer-managed Postgres admin (creates component DBs) ---
postgresql_admin_username: "postgres"
postgresql_admin_password: "pgadminpassword"    # CHANGE ME

# --- Redis ---
redis_mode: "standalone"              # single-node demo; adjust for HA
```

> Encrypt if storing secrets locally:
> ```bash
> ansible-vault encrypt vars.yml
> ```
>
> **Tip (CI/CD):** Instead of committing secrets, map them from environment variables:
> ```yaml
> aap_registry_username: "{{ lookup('env', 'RH_REG_USER') }}"
> aap_registry_password: "{{ lookup('env', 'RH_REG_PASS') }}"
> ```

### 3) Install

```bash
ansible-playbook -i inventory -e @vars.yml   collections/ansible_collections/ansible/containerized_installer/playbooks/install.yml
```

### 4) Uninstall

```bash
ansible-playbook -i inventory -e @vars.yml   collections/ansible_collections/ansible/containerized_installer/playbooks/uninstall.yml
```

---

## Verify

- Check containers running: `podman ps` or `docker ps`
- Browse to your Controller URL (e.g., `https://aap.example.com`) and log in

---

## Suggested Repo Layout

```text
aap-containerized-deployment/
├─ inventory
├─ vars.yml
├─ vars.example.yml
└─ collections/
   └─ ansible_collections/
      └─ ansible/
         └─ containerized_installer/
            └─ playbooks/
               ├─ install.yml
               └─ uninstall.yml
```

---


In GitHub Actions, define `RH_REG_USER` and `RH_REG_PASS` as **Repository Secrets**.

---

## .gitignore (recommended)

```gitignore
# Ansible artifacts
*.retry
*.vault
*.vault.json

# Local secrets
vars.yml
.env
.credentials
```

---

## Troubleshooting

- **Authentication to registry fails** → Validate `registry_username`/`registry_password` and entitlement for AAP images.
- **Hostname/route not reachable** → Confirm DNS or `/etc/hosts`, and that `gateway_main_url`/Controller route matches your certs.
- **Ports/SELinux** → Open required ports; ensure SELinux contexts aren’t blocking container networking.

---

## License

MIT (or your preferred license).
