# Ansible Admin Monitoring

Ansible Admin Monitoring is a role-based automation project for deploying a lightweight host and container monitoring stack on Linux servers.

The project focuses on a small, practical observability setup:

- base host preparation with Docker support
- host metrics through `node_exporter`
- OS update and system status metrics through the node_exporter textfile collector
- container metrics through cAdvisor
- centralized log storage and querying with VictoriaLogs


## Project Goals

This project demonstrates how to structure an Ansible automation repository that is easy to read, extend, and operate:

- separate responsibilities into reusable roles
- keep inventory, shared variables, vault secrets, and playbooks clearly separated
- use systemd-managed services for long-running exporters
- keep node_exporter textfile metrics consistent across roles
- use Ansible Vault for sensitive values
- provide a sanitized inventory suitable for sharing or presentation

## Technology Stack

- Ansible
- Linux systemd
- Docker
- node_exporter
- cAdvisor
- VictoriaLogs
- Ansible Vault

## Repository Layout

```text
.
├── ansible.cfg
├── inventory/
│   └── local/
│       ├── group_vars/
│       │   └── all.yaml
│       ├── host_vars/
│       │   └── .gitkeep
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

`ansible.cfg` points Ansible to:

```ini
inventory = ./inventory/local/hosts.yaml
roles_path = ./roles
```

## Inventory Design

The inventory is stored in `inventory/local/hosts.yaml`.

It uses sanitized project hostnames such as:

```yaml
monitoring-node-01:
  ansible_host: 10.10.10.11
```

The IP addresses are placeholder private addresses and should be replaced with real server IPs before running against a real environment.

The inventory contains these groups:

- `common`
- `node_exporter`
- `os_updates`
- `cadvisor`
- `victorialogs`

Each group maps hosts to the role that should run on them. A host can belong to multiple groups, which allows the same server to receive multiple monitoring components.

Shared variables are stored in:

```text
inventory/local/group_vars/all.yaml
```

Current shared values include:

- SSH user: `admin`
- managed admin account: `admin`
- node_exporter textfile collector path
- OS updates metric output path

## Playbook Flow

The main playbook is:

```text
playbooks/monitoring.yaml
```

It runs the roles in this order:

1. `common`
2. `node_exporter`
3. `os_updates`
4. `cadvisor`
5. `victorialogs`

This order matters because `common` prepares the base operating system, Docker, and user groups before the exporter and service roles are applied.

## Roles

### common

Prepares the target Linux hosts.

Main responsibilities:

- install base packages
- install Docker
- start and enable Docker
- create or manage the configured admin user
- assign admin groups such as `sudo`, `docker`, `adm`, and `systemd-journal`

### node_exporter

Installs and manages node_exporter as a systemd service.

Main responsibilities:

- create the `node_exporter` system user and group
- download and install node_exporter
- expose host metrics on port `9100`
- enable the textfile collector

Textfile collector directory:

```text
/var/lib/node_exporter/textfile
```

### os_updates

Deploys a small custom exporter script for OS update and host status metrics.

Main responsibilities:

- install `/usr/local/bin/export_os_updates.sh`
- deploy a systemd service and timer
- write metrics into the node_exporter textfile collector directory
- collect APT update count, CPU info, memory values, and filesystem usage

Metric output file:

```text
/var/lib/node_exporter/textfile/os_updates.prom
```

Default timer interval:

```text
30m
```

### cadvisor

Runs cAdvisor as a Docker container for container-level metrics.

Main responsibilities:

- start the `gcr.io/cadvisor/cadvisor:latest` container
- expose cAdvisor on host port `9080`
- mount Docker and host filesystem paths needed for container metrics
- keep the container running with the configured restart policy

### victorialogs

Installs and manages VictoriaLogs as a systemd service.

Main responsibilities:

- create the `victorialogs` system user and group
- download the VictoriaLogs Linux AMD64 archive
- install the `victoria-logs-prod` binary
- create data and log directories
- run VictoriaLogs on port `9428`

Default data directory:

```text
/var/lib/victorialogs
```

Default retention:

```text
31d
```

## Secrets And Vault

Sensitive values belong in:

```text
vault/secrets.yaml
```

The file is stored as an Ansible Vault encrypted file. The vault password itself must not be committed.

This repository ignores:

```text
vault/vault_pass.txt
```

Useful vault commands:

```bash
ansible-vault view vault/secrets.yaml --ask-vault-pass
ansible-vault edit vault/secrets.yaml --ask-vault-pass
ansible-vault rekey vault/secrets.yaml
```

If you prefer a local password file:

```bash
ansible-vault view vault/secrets.yaml --vault-password-file vault/vault_pass.txt
```

## How To Run

Go to the project directory:

```bash
cd /home/user/test_projects/ansible-monitoring-automation
```

Check that the inventory parses correctly:

```bash
ansible-inventory --graph
```

Run a syntax check:

```bash
ansible-playbook --syntax-check playbooks/monitoring.yaml
```

Run the full monitoring deployment:

```bash
ansible-playbook playbooks/monitoring.yaml --ask-become-pass --ask-vault-pass
```

If using a local vault password file:

```bash
ansible-playbook playbooks/monitoring.yaml --ask-become-pass --vault-password-file vault/vault_pass.txt
```

## Customization

Before using this project in a real environment:

1. Replace placeholder IPs in `inventory/local/hosts.yaml`.
2. Confirm the SSH user in `inventory/local/group_vars/all.yaml`.
3. Add host-specific overrides in `inventory/local/host_vars/` when needed.
4. Review role defaults under `roles/*/defaults/main.yaml`.
5. Rekey or recreate `vault/secrets.yaml` for the new environment.

## Validation

The project is considered ready to run when both commands pass:

```bash
ansible-inventory --graph
ansible-playbook --syntax-check playbooks/monitoring.yaml
```

## Summary

This project demonstrates practical configuration automation with Ansible: role-based Linux configuration, exporter deployment, Docker-based container metrics, VictoriaLogs service management, encrypted secret handling, and a clean inventory structure suitable for multi-host monitoring environments.
