
# Ping role

Checks whether the target hosts are reachable with Ansible and verifies that
SSH access and permissions are configured correctly.

## Example

```bash
ansible-run playbooks/ping.yml
```

## Example role

```yaml
- name: Check host connectivity
	hosts: all
	gather_facts: false
	roles:
		- role: ping
```
