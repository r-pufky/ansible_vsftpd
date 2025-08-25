# Very Secure FTP Daemon (VSFTPd)
VSFTPd service configuration.

## Requirements
[supported platforms](https://github.com/r-pufky/ansible_vsftpd/blob/main/meta/main.yml)

## Role Variables
[defaults](https://github.com/r-pufky/ansible_vsftpd/tree/main/defaults/main/)

### Ports
All ports and protocols have been defined for the role.

[defaults/ports.yml](https://github.com/r-pufky/ansible_vsftpd/blob/main/defaults/main/ports.yml)

## Dependencies
**galaxy-ng** roles cannot be used independently. Part of
[r_pufky.srv](https://github.com/r-pufky/ansible_collection_srv) collection.

## Example Playbook
Read through defaults and [arch documentation](https://wiki.archlinux.org/title/Very_Secure_FTP_Daemon)
before proceeding. VSFTPd is very powerful but easily mis-configured. Role
requires configuration for deployment.

### FTP server with local users.
Setup FTP server allowing `test_local_user` local user login. `test_local_user`
is managed outside of this role.

``` yaml
vsftpd_cfg_write_enable: true
vsftpd_cfg_chroot_local_user: true
vsftpd_cfg_chroot_list_enable: true
vsftpd_cfg_chroot_list:
  - 'test_local_user'
vsftpd_cfg_ls_recurse_enable: true
vsftpd_srv_harden_enable: true
```

``` yaml
- name: 'FTP local users'
  hosts: '*'
  become: true
  roles:
     - 'r_pufky.srv.vsftpd'
```

### FTP server with virtual users.
Setup FTP server allowing virtual user logins backed by the local `ftp` user
account. Virtual user configuration is automatically managed, see defaults.

host_vars/vsftpd.example.com/vars/vsftpd.yml
``` yaml
vsftpd_srv_user_home_mode: '0550'
vsftpd_srv_virtual_users:
  - user: 'test'
    pass: '{{ vault_test }}'
  - user: 'test2'
    pass: '{{ vault_test2 }}'
vsftpd_cfg_anonymous_enable: false
vsftpd_cfg_local_enable: true
vsftpd_cfg_write_enable: true
vsftpd_cfg_chroot_local_user: true
vsftpd_cfg_guest_enable: true
vsftpd_cfg_guest_username: 'ftp'
vsftpd_cfg_virtual_use_local_privs: true
vsftpd_srv_harden_enable: true
```

``` yaml
- name: 'FTP local users'
  hosts: '*'
  become: true
  roles:
     - 'r_pufky.srv.vsftpd'
```

## Troubleshooting

### status=2/INVALIDARGUMENT
If service fails to start with `status=2/INVALIDARGUMENT`; the configuration
file is syntactically correct but has conflicting settings. This is an end user
configuration issue.

Manually execute the configuration file to determine the exact cause:

``` bash
vsftpd /etc/vsftpd.conf
```

### 500 OOPS: vsftpd: refusing to run with writable root inside chroot ()
Chroot environments require read-only root directories; otherwise logins result
in:

``` bash
500 OOPS: vsftpd: refusing to run with writable root inside chroot ()
```

Check your configuration and set the backing user home directory read-only.
See `vsftpd_srv_local_root_mode`, `vsftpd_srv_anon_root_mode`, and
`vsftpd_srv_user_home_mode`. Ensure `vsftpd_srv_local_root_recursive_enable`,
`vsftpd_srv_anon_root_recursive_enable` are not plowing directory
permissions.

Typically `0550` permissions resolve.

Reference:
* https://www.benscobie.com/fixing-500-oops-vsftpd-refusing-to-run-with-writable-root-inside-chroot/

## Development
Configure [environment](https://github.com/r-pufky/ansible_collection_srv/blob/main/docs/dev/environment/README.md)

Run all unit tests:
``` bash
molecule test --all
```

### Releases
Major release versions track VSFTPd release versions. Branch label tracks
Debian OS release.

* **[13.x.x](https://github.com/r-pufky/ansible_vsftpd)**: 13 Trixie.
* **[12.x.x](https://github.com/r-pufky/ansible_vsftpd/tree/12.x)**: 12 Bookworm.

## Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License](https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0)
 [(direct link)](https://github.com/r-pufky/ansible_vsftpd/blob/main/LICENSE)

## Author Information
PGP Fingerprint: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9](https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9)
| [github gist](https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22)
