# Homepage

Deploys Homepage, a self-hosted dashboard, using Docker Compose.

## Requirements

- Docker Engine with the Docker Compose v2 plugin on the target host.
- The `community.docker` Ansible collection on the control node.

## Configuration

The role gets its service catalog from `homepage_services`. The default is an
empty list; provide the dashboard content in your own inventory. `homepage_dir`
defaults to `/opt/homepage` and can also be overridden there.

The Compose-related defaults are `homepage_image`, `homepage_container_name`,
`homepage_port`, `homepage_allowed_hosts`, and `homepage_docker_socket_path`.
Override them in inventory when needed. Set `homepage_allowed_hosts` to the host
and port users will use in their browser.

Example inventory structure:

```text
inventory/
  hosts.yml
  group_vars/
    homepage.yml
```

`inventory/hosts.yml`:

```yaml
---
all:
  children:
    homepage:
      hosts:
        dashboard01:
          ansible_host: 192.0.2.10
```

`inventory/group_vars/homepage.yml`:

```yaml
---
homepage_services:
  - Monitoring:
      - Grafana:
          icon: grafana.svg
          href: https://grafana.example.org
          description: Dashboards and metrics
homepage_allowed_hosts: dashboard.example.org:3000
```

Use `host_vars/<host>.yml` instead when each Homepage instance needs a different
catalog.

## Playbook

```yaml
---
- name: Deploy Homepage
  hosts: homepage
  become: true
  roles:
    - role: homepage
```

Run it with your inventory:

```bash
ansible-playbook -i ../homelab-inventory/inventory/hosts.yml playbooks/services/homepage.yml
```

To pull the configured image and apply it, run the opt-in update tag:

```bash
ansible-playbook -i ../homelab-inventory/inventory/hosts.yml playbooks/services/homepage.yml --tags homepage,homepage_update
```

Normal deployments do not force an image pull. This update tag does not prune
old images.

## Access and security

Homepage is intended for the trusted LAN only. Do not expose or port-forward it
to the public internet without HTTPS and appropriate authentication. The Docker
socket is mounted read-only, but that is not a complete Docker API authorization
boundary.

Service icon catalog: <https://github.com/homarr-labs/dashboard-icons/tree/main/svg>