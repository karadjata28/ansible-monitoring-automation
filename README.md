# Ansible Admin Monitoring

Ansible project for deploying a scoped host/container monitoring and VictoriaLogs stack.

Included roles:

- `common`
- `node_exporter`
- `os_updates`
- `cadvisor`
- `victorialogs`

## Repository Layout

```text
.
├── ansible.cfg
├── inventory/
│   └── local/
│       ├── group_vars/
│       │   └── all.yaml
│       ├── host_vars/
│       └── hosts.yaml
├── playbooks/
│   └── monitoring.yaml
├── roles/
│   ├── common/
│   ├── node_exporter/
│   ├── os_updates/
│   ├── cadvisor/
│   └── victorialogs/
└── vault/
    └── secrets.yaml
```

`ansible.cfg` uses `inventory/local/hosts.yaml` and `roles/` by default.

## Inventory

The local inventory contains only these groups:

- `common`
- `node_exporter`
- `os_updates`
- `cadvisor`
- `victorialogs`

The active workstation hosts copied from the source project are assigned to `common`, `node_exporter`, and `os_updates`. The `cadvisor` and `victorialogs` groups are present but empty until you add the container hosts and VictoriaLogs host.

Shared connection/admin variables live in `inventory/local/group_vars/all.yaml`.

## Roles

`common` installs base packages, Docker, and the configured admin user.

`node_exporter` installs node_exporter and enables the textfile collector at:

```text
/var/lib/node_exporter/textfile
```

`os_updates` installs a systemd timer that writes update and host metrics to:

```text
/var/lib/node_exporter/textfile/os_updates.prom
```

`cadvisor` runs the cAdvisor container with Docker and exposes container metrics on `cadvisor_host_port`.

`victorialogs` installs VictoriaLogs, creates its service user/data directories, and runs it under systemd.

## Vault

`vault/secrets.yaml` is copied from the source project as an encrypted Ansible Vault file.

`vault/vault_pass.txt` is intentionally ignored by git. Use `--ask-vault-pass` or pass `--vault-password-file vault/vault_pass.txt` when a command needs to read encrypted vault data.

Useful commands:

```bash
ansible-vault view vault/secrets.yaml
ansible-vault edit vault/secrets.yaml
ansible-vault rekey vault/secrets.yaml
```

## Running

Syntax check:

```bash
ansible-playbook --syntax-check playbooks/monitoring.yaml
```

Run the full scoped playbook:

```bash
ansible-playbook playbooks/monitoring.yaml --ask-become-pass --ask-vault-pass
```

If `vault/vault_pass.txt` exists locally, use `--vault-password-file vault/vault_pass.txt` instead of `--ask-vault-pass`.

## Start The Project

From the repository root:

```bash
cd /home/akaradjov/test_projects/ansible-monitoring-automation
ansible-playbook --syntax-check playbooks/monitoring.yaml
ansible-playbook playbooks/monitoring.yaml --ask-become-pass --ask-vault-pass
```

If you use a local vault password file:

```bash
ansible-playbook playbooks/monitoring.yaml --ask-become-pass --vault-password-file vault/vault_pass.txt
```
