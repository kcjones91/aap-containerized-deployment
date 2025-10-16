# AAP Containerized Deployment

Minimal, GitHub‑ready guide to install/uninstall **Red Hat Ansible Automation Platform (AAP)** using the **containerized installer**.

---

## Overview

This repo deploys AAP to the local/target host using Ansible playbooks that live under:
`collections/ansible_collections/ansible/containerized_installer/playbooks/`

- **Install:** `install.yml`
- **Uninstall:** `uninstall.yml`

> ⚠️ **Protect secrets**: do **not** commit real passwords/tokens. Prefer Ansible Vault or environment variables in CI.

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

### 2) Create `vars.yml` (minimal example)

```yaml
aap_deployment_type: "standard"           # e.g., "standard", "controller-only"
aap_namespace: "aap"
aap_registry_url: "{{ registry_url }}"
aap_registry_username: "{{ registry_username }}"
aap_registry_password: "{{ registry_password }}"
controller_route_host: "aap.example.com"  # set to your route/hostname
```

> Encrypt if storing secrets locally:
> ```bash
> ansible-vault encrypt vars.yml
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
└─ collections/
   └─ ansible_collections/
      └─ ansible/
         └─ containerized_installer/
            └─ playbooks/
               ├─ install.yml
               └─ uninstall.yml
```

---

## CI/CD & Secrets (optional)

Use environment variables in CI instead of committing passwords:

```yaml
# Example lookup usage inside vars.yml
aap_registry_username: "{{ lookup('env', 'RH_REG_USER') }}"
aap_registry_password: "{{ lookup('env', 'RH_REG_PASS') }}"
```

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
- **Hostname/route not reachable** → Confirm DNS or `/etc/hosts`, and that `controller_route_host` matches your cert/route.
- **Ports/SELinux** → Open required ports; ensure SELinux contexts aren’t blocking container networking.

---

## License

MIT (or your preferred license).# aap-containerized-deployment
