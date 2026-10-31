# setdns

This role configures DNS in NetworkManager Ethernet profiles using
`community.general.nmcli`. It requires the `community.general` collection,
nmcli, NetworkManager, and privilege escalation for changes. It does not install
packages, edit resolver files, or explicitly restart NetworkManager.

Supply the exact names of existing Ethernet connection profiles, which are not
necessarily the same as interface names. With `state: present`, a missing profile
can be created and activated by the module. The module's defaults for other
profile settings, such as autoconnect and automatic routes, also apply; this
minimal role does not override or preserve every non-DNS setting.

Variables:

- `setdns_connections`: existing connection names to configure (required).
- `setdns_nameservers`: IPv4 DNS servers (required).
- `setdns_search_domains`: DNS search domains (default: `[]`).
- `setdns_options`: resolver options (default: `[]`).
- `setdns_ignore_auto_dns`: ignore automatically supplied IPv4 and IPv6 DNS
  on selected connections (default: `true`). Static IPv6 DNS is left unchanged.

Keep DNS addresses in your private inventory's group variables and connection
names in host variables if they differ. Example using a documentation-only IP:

```yaml
setdns_nameservers:
  - "192.0.2.53"
setdns_connections:
  - "Wired connection 1"
```

For existing profiles, settings are saved without explicitly reactivating the
connection. Active DNS may remain
unchanged until the connection is next activated by the system or administrator.

The module reports changes when the managed profile settings differ. Use
`--check` to preview changes without modifying profiles or restarting networking.
Test on one server before deploying more widely.

## Optional Network Restart

Network activation belongs to the calling playbook, not this role. In
`playbooks/setdns.yml`, an optional network service restart runs in
`post_tasks`, after all roles and normal tasks on each host:

- `setdns_restart_network`: enable the final restart (default: `false`).
- `setdns_network_service`: service to restart (default: `NetworkManager`).

Set these in your private inventory or with extra variables. Use the service
that actually manages networking on the target; do not assume it is `network`.
The restart is asynchronous, followed by an SSH connection check and a check
of the restart job's result. It is skipped with `--check` and runs even if DNS
was already saved by a previous attempt.

Restarting networking can interrupt SSH and other traffic. Waiting for SSH does
not guarantee connectivity will recover if the configuration is wrong. Verify
active DNS after activation and test on one host with console access available.

Save DNS without restarting anything:

```bash
ansible-playbook -i ../homelab-inventory/inventory/hosts.yml playbooks/setdns.yml --limit slowroll-01 -K
```

Save DNS and explicitly request a final network restart:

```bash
ansible-playbook -i ../homelab-inventory/inventory/hosts.yml playbooks/setdns.yml --limit slowroll-01 -K -e setdns_restart_network=true
```

When composing another playbook, place other roles and tasks before its final
activation step. The setdns role itself never requests a restart.

## Existing Reboot Role

Instead of a network restart, a calling playbook can finish with the existing
`reboot` role. Do not enable both activation methods for the same run. Example:

```yaml
---
- name: Configure services and optionally reboot last
  hosts: all
  roles:
    - setdns
    - ping
  post_tasks:
    - name: Optionally reboot after all configuration tasks
      ansible.builtin.include_role:
        name: reboot
      when:
        - setdns_reboot | default(false) | bool
        - not ansible_check_mode
```

Here `setdns_reboot` is an opt-in variable for that example playbook. The
existing role uses `ansible.builtin.reboot`, which waits for the host to return.
No changes to the reboot role are required.