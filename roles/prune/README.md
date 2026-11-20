# prune

This role removes dangling Docker images from the target host.

Example playbook (save as `playbooks/prune.yml`):

```yaml
---
- name: Prune dangling Docker images
  hosts: all
  roles:
    - role: prune
      tags: prune
```

    Run it with the private homelab inventory:

    ```bash
    ansible-playbook -i ../homelab-inventory/inventory/hosts.yml --tags prune playbooks/prune.yml
    ```