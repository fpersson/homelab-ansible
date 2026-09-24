# steam

Creates a dedicated `steam` user, downloads and extracts SteamCMD, and
initializes it for use on the target host. On openSUSE Tumbleweed, Slowroll,
and MicroOS, the role also installs the required 32-bit support libraries.

## Example

```bash
ansible-run playbooks/gameserver.yml
```

## Example role

```yaml
---
- name: Install Steam
  hosts: tumbleweed-01
  become: true
  roles:
    - role: steam
```

## Defaults

- `steam_user` sets the account used to run SteamCMD. It defaults to `steam`.
- `steam_user_pwd` sets the password hash for the `steam` user.
- `steam_user_home` sets the account home directory.
- `steam_dir` sets the SteamCMD installation directory.
- `steam_file` sets the SteamCMD download URL.

The default `steam_user_pwd` locks password login and must be replaced with a
secure hash if password login is required.
The variable name is `steam_user_pwd` (not `steam_usr_pwd`). Generate a new
Linux password hash and store it in an encrypted Ansible Vault file:

```bash
openssl passwd -6
ansible-vault create group_vars/all/vault.yml
```

Add the generated hash to the Vault file:

```yaml
steam_user_pwd: "<generated-password-hash>"
```

Run the playbook with the Vault password:

```bash
ansible-playbook \
  -i ../homelab-inventory/inventory/hosts.yml \
  --ask-vault-pass \
  --ask-become-pass \
  playbooks/gameserver.yml \
  --limit="worker-01"
```

For a one-time test, override the hash with extra vars using
`-e steam_user_pwd='<generated-password-hash>'`. Do not commit the hash in an
unencrypted file or expose it in shell history.

The role initializes SteamCMD with `./steamcmd.sh +quit` as the `steam` user.
If initialization fails while downloading SteamCMD files, check network access
to `client-update.steamstatic.com` and the permissions of `steam_dir`.

## Links

* [steamcmd](https://developer.valvesoftware.com/wiki/SteamCMD)
