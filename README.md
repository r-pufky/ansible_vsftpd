# Very Secure FTP Daemon (VSFTPd)
VSFTPd service configuration.

## Requirements
[supported platforms](https://github.com/r-pufky/ansible_vsftpd/blob/main/meta/main.yml)

[collections/roles](https://github.com/r-pufky/ansible_vsftpd/blob/main/meta/requirements.yml)

## Role Variables
[defaults](https://github.com/r-pufky/ansible_vsftpd/tree/main/defaults/main/)

## Dependencies
Part of the [r_pufky.srv](https://github.com/r-pufky/ansible_collection_srv)
collection.

## Example Playbook
Read through defaults and [arch documentation](https://wiki.archlinux.org/title/Very_Secure_FTP_Daemon)
before proceeding. VSFTPd is very powerful but easily mis-configured. Role
requires configuration for deployment.

### FTP server with local users.
Setup FTP server allowing `test_local_user` local user login. `test_local_user`
is managed outside of this role.

``` yaml
vsftpd_config_write_enable: true
vsftpd_config_chroot_local_user: true
vsftpd_config_allow_writeable_chroot: true
vsftpd_config_chroot_list_enable: true
vsftpd_config_chroot_list:
  - 'test_local_user'
vsftpd_config_ls_recurse_enable: true
vsftpd_service_harden_enable: true
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
vsftpd_user_home_mode: '0550'
vsftpd_service_virtual_users:
  - user: 'test'
    pass: '{{ vault_test }}'
  - user: 'test2'
    pass: '{{ vault_test2 }}'
vsftpd_config_anonymous_enable: false
vsftpd_config_local_enable: true
vsftpd_config_write_enable: true
vsftpd_config_chroot_local_user: true
vsftpd_config_guest_enable: true
vsftpd_config_guest_username: 'ftp'
vsftpd_config_virtual_use_local_privs: true
vsftpd_service_harden_enable: true
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
See `vsftpd_service_local_root_mode`, `vsftpd_service_anon_root_mode`, and
`vsftpd_user_home_mode`. Typically `0550` permissions resolve.

Reference:
* https://www.benscobie.com/fixing-500-oops-vsftpd-refusing-to-run-with-writable-root-inside-chroot/

## Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License](https://github.com/r-pufky/ansible_vsftpd/blob/main/LICENSE)

## Author Information
https://keybase.io/rpufky
