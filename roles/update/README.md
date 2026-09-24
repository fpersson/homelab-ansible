# Update role

Updates packages on SUSE hosts with `zypper`. The operation is selected
automatically based on the distribution.

## Example

```bash
ansible-run playbooks/update.yml
```

## Example role

```yaml
- name: Update hosts
	hosts: all
	gather_facts: false
	roles:
		- role: update
```

## Update behavior

- openSUSE Leap uses `zypper update` to install the latest packages.
- openSUSE Tumbleweed and MicroOS use `zypper dup` for a distribution upgrade.

Run the playbook without tags:

```bash
ansible-playbook playbooks/update.yml
```