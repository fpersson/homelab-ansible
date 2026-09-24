# Ansible Inventory Example

This document is a standalone example of the inventory layout used by this repository. It is intended as a starting point for a new environment.

The live inventory is organized into three parts:

- `hosts.yml` defines hosts and groups.
- `group_vars/` contains variables shared by every host in a group.
- `host_vars/` contains variables for one specific host.

Replace the example hostnames, IP addresses, usernames, and service addresses with values from your own environment. Do not copy passwords, API tokens, or other secrets into these files.

## Directory Layout

```text
inventory/
├── hosts.yml
├── group_vars/
│   ├── all.yml
│   ├── monitoring_servers.yml
│   ├── worker_nodes.yml
│   └── deploy_monitoring_agent.yml
└── host_vars/
    ├── monitoring-01.yml
    ├── worker-01.yml
    └── worker-02.yml
```

The file name in `group_vars/` must match the group name in `hosts.yml`. For example, variables for `worker_nodes` belong in `group_vars/worker_nodes.yml`.

## `hosts.yml`

```yaml
---
all:
  children:
    monitoring_servers:
      hosts:
        monitoring-01:

    worker_nodes:
      hosts:
        worker-01:
        worker-02:

    deploy_monitoring_agent:
      hosts:
        worker-01:
        worker-02:
```

A host can belong to more than one group. In this example, the worker hosts are members of both `worker_nodes` and `deploy_monitoring_agent`.

The host names listed here are inventory names. Their network addresses are assigned in matching files under `host_vars/`.

## Variables For All Hosts

Create `inventory/group_vars/all.yml` for settings shared by every managed host:

```yaml
---
ansible_user: ansible
ansible_python_interpreter: /usr/bin/python3
service_base_dir: /opt
```

Use a dedicated automation account when possible. If a host needs a different user, override `ansible_user` in that host's file.

## Group Variables

Create `inventory/group_vars/monitoring_servers.yml` for values shared by monitoring servers:

```yaml
---
monitoring_host: "192.0.2.10"
prometheus_host: "192.0.2.10"
grafana_prometheus_port: 9091
loki_host: "192.0.2.10"
redis_host: "192.0.2.12"
grafana_redis_port: 6379
```

Create `inventory/group_vars/deploy_monitoring_agent.yml` for values used by monitoring agents:

```yaml
---
server_role: monitoring_agent
monitoring_host: "192.0.2.10"
prometheus_host: "192.0.2.10"
loki_host: "192.0.2.10"
redis_host: "192.0.2.12"
```

The addresses in this example use the documentation-only `192.0.2.0/24` range. Replace them with reachable addresses in the target environment.

If a group needs privilege escalation, define it in that group's variables or in the relevant host variables:

```yaml
---
ansible_become: true
```

## Host Variables

Create one file per inventory host. The file name must match the host name in `hosts.yml`.

`inventory/host_vars/monitoring-01.yml`:

```yaml
---
ansible_host: 192.0.2.10
ansible_user: ansible
server_role: monitoring_master
```

`inventory/host_vars/worker-01.yml`:

```yaml
---
ansible_host: 192.0.2.11
ansible_user: ansible
server_role: monitoring_agent
```

`inventory/host_vars/worker-02.yml`:

```yaml
---
ansible_host: 192.0.2.12
ansible_user: ansible
server_role: monitoring_agent
```

`ansible_host` is the address Ansible connects to. It may be different from the inventory name. For example, the inventory name can be `worker-01` while the actual address is supplied by `ansible_host`.

## Secrets

Keep passwords, private keys, tokens, and other sensitive values in an Ansible Vault file instead of `hosts.yml`, `group_vars/`, or `host_vars/`.

Example command to create a vault file:

```bash
ansible-vault create secrets.enc
```

Run a playbook with the vault variables loaded:

```bash
ansible-playbook \
  -i inventory/hosts.yml \
  -e@secrets.enc \
  --ask-vault-pass \
  playbooks/ping.yml
```

Do not commit an unencrypted secrets file.

## Validate A New Inventory

From the repository root, check the parsed inventory before running a playbook:

```bash
ansible-inventory -i inventory/hosts.yml --graph
ansible-inventory -i inventory/hosts.yml --list
ansible -i inventory/hosts.yml all --list-hosts
ansible -i inventory/hosts.yml all -m ping
```

Confirm that:

- Every host in `hosts.yml` has a matching file in `host_vars/` when it needs host-specific settings.
- Every group variable file uses the exact group name from `hosts.yml`.
- `ansible_host` points to the intended machine.
- The SSH user has the required permissions.
- Service and monitoring addresses are reachable from the managed hosts.

## Target A Group Or Host

Run a playbook against one group:

```bash
ansible-playbook \
  -i inventory/hosts.yml \
  -l worker_nodes \
  playbooks/monitoring/agents.yml
```

Run it against one host:

```bash
ansible-playbook \
  -i inventory/hosts.yml \
  -l worker-01 \
  playbooks/monitoring/agents.yml
```

The `-l` option limits the playbook to the selected group or host, which is useful when testing a new inventory.
