# ansible-homelab

Ansible playbooks and roles for managing my homelab. The inventory is kept in
the separate, private `homelab-inventory` repository.

## Requirements

- Ansible installed locally
- A configured inventory at `../homelab-inventory/inventory/hosts.yml`
- The collections listed in `requirements.yml`

Install the required collections from the project root:

```bash
ansible-galaxy collection install -r requirements.yml
```

## Usage

Run all commands from the project root. Check SSH connectivity and permissions
before making changes:

```bash
ansible-playbook \
	-i ../homelab-inventory/inventory/hosts.yml \
	playbooks/ping.yml
```

The update playbook updates packages and then reboots the hosts:

```bash
ansible-playbook \
	-i ../homelab-inventory/inventory/hosts.yml \
	playbooks/update.yml
```

The update operation is selected automatically based on the distribution:

- openSUSE Leap uses `zypper update`.
- openSUSE Tumbleweed and MicroOS use `zypper dup`.

Use `--limit HOST_OR_GROUP` to target a specific host or inventory group.

## Roles

- `ping` checks host connectivity and configured SSH access.
- `update` updates packages or performs a distribution upgrade.
- `reboot` reboots hosts and waits for them to become available again.

## Inventory Example

See [docs/inventoy_example.md](docs/inventoy_example.md) for an example
inventory layout. Do not commit passwords, private keys, or other secrets.