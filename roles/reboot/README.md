# Reboot role

Reboots the target hosts and waits for them to become available again.

## Example

```bash
ansible-run playbooks/update.yml
```

## Example role

```yaml
- name: Reboot hosts
	hosts: all
	gather_facts: false
	roles:
		- role: reboot
```