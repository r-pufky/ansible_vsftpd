# [Very Secure FTP Daemon (VSFTPd)][h]
VSFTPd service configuration.

## [Requirements][i]
Requires [r_pufky.srv][g] galaxy-ng collection. See
[additional documentation][m] and [reference documentation][n] for
troubleshooting and config variables.

Install size: ~360KB

> Tasks [potentially touching Network Mounted Filesystems][o] will be run as
> the task user and fallback to the service user. Manage these locations
> externally if these fail.

## Role Variables
Detailed variable use documented in defaults. See usage for role operation.

* [defaults][j] - User configurable service options.

* [config][j] - Encapsulated VSFTPd config options.

* [ports][k] - Ports are **not** managed (defined for external use).

## Usage

  Path                   | Usage
 ------------------------|-------
  /etc/vsftpd.d          | Role managed files deployed here.
  /etc/vsftpd.d/user.d   | Controller sourced user files deployed here.
  /var/vsftpd.d/secure.d | Controller sourced user secure files (certs, etc.) deployed here.

### Feature Flags
Tasks are gated by feature flags and executed in the following order.

  Step | Flag               | Notes
 ------|--------------------|-------
  1    | vsftpd_flg_install | Install required packages, users, etc.
  2    | vsftpd_flg_harden  | Harden service.

### Example Playbooks
Read through defaults and [additional documentation][m] before proceeding.
VSFTPd is very powerful but easily mis-configured. Role requires configuration
for deployment.

### FTP server with local users
Setup FTP server allowing `test_local_user` local user login. `test_local_user`
is managed outside of this role.

``` yaml
- name: 'Create VSFTPd instance with local users.'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.vsftpd'
  vars:
    vsftpd_flg_harden: true
    vsftpd_srv_chroot_list:
      - 'test_local_user'
    vsftpd_cfg_write_enable: true
    vsftpd_cfg_chroot_local_user: true
    vsftpd_cfg_chroot_list_enable: true
    vsftpd_cfg_ls_recurse_enable: true
```

### FTP server with virtual users.
Setup FTP server allowing virtual user logins backed by the local `ftp` user
account. Virtual user configuration is automatically managed, see defaults.

``` yaml
- name: 'Create VSFTPd instance with virtual users.'
  ansible.builtin.include_role:
    name: 'r_pufky.srv.vsftpd'
  vars:
    vsftpd_flg_harden: true
    vsftpd_srv_home_mode: '0550'
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
```

## Troubleshooting

### status=2/INVALIDARGUMENT
If service fails to start with `status=2/INVALIDARGUMENT` means the
configuration file is syntactically correct but has conflicting settings. This
is an **end user** configuration issue.

Manually execute the configuration file to determine the exact cause:

``` bash
vsftpd /etc/vsftpd.conf
```

### 500 OOPS: vsftpd: refusing to run with writable root inside chroot ()
Chroot environments [require read-only root directories][l]. Otherwise logins
result in:

``` bash
500 OOPS: vsftpd: refusing to run with writable root inside chroot ()
```

Check your configuration and set the backing user home directory read-only.
See `vsftpd_srv_local_root_mode`, `vsftpd_srv_anon_root_mode`, and
`vsftpd_srv_home_mode`. Ensure `vsftpd_srv_local_root_recursive`,
`vsftpd_srv_anon_root_recursive` are not plowing directory permissions.

Typically `0550` permissions resolve.

## Development
Configure [environment][a].

``` bash
# Run all tests.
molecule test --all
```

### [Releases][b]

  Release | Debian | Ansible | VSFTPd | Notes
 ---------|--------|---------|--------|-------
  5.x.x   | 13     | 2.20    | v3.0.5 | Ansible 2.20, feature flags, and semantic versioning.
  4.x.x   | 12     | 2.18    | v3.0.5 | Migrate to Debian Trixie.
  3.x.x   | 12     | 2.18    | v3.0.5 | Implement data annotations.
  2.x.x   | 12     | 2.18    | v3.0.5 | Use libraries for common operations.
  1.x.x   | 12     | 2.18    | v3.0.5 | Initial release.

## Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License][c] | [direct link][f]

## Author Information
PGP: [466EEC2B67516C7117C85CE3A0BC35D16698BAB9][d] | [github gist][e]

[a]: https://r-pufky.github.io/ansible_docs
[b]: https://semver.org/spec/v2.0.0
[c]: https://www.tldrlegal.com/license/gnu-affero-general-public-license-v3-agpl-3-0
[d]: https://keys.openpgp.org/vks/v1/by-fingerprint/466EEC2B67516C7117C85CE3A0BC35D16698BAB9
[e]: https://gist.github.com/r-pufky/a8df36977c55b5bb20829267c4c49d22

[f]: https://github.com/r-pufky/ansible_vsftpd/blob/main/LICENSE
[g]: https://github.com/r-pufky/ansible_collection_srv
[h]: https://security.appspot.com/vsftpd.html
[i]: https://github.com/r-pufky/ansible_vsftpd/blob/main/meta/main.yml
[j]: https://github.com/r-pufky/ansible_vsftpd/tree/main/defaults/main/main.yml
[k]: https://github.com/r-pufky/ansible_vsftpd/blob/main/defaults/main/ports.yml
[l]: https://www.benscobie.com/fixing-500-oops-vsftpd-refusing-to-run-with-writable-root-inside-chroot
[m]: https://wiki.archlinux.org/title/Very_Secure_FTP_Daemon
[n]: https://linux.die.net/man/5/vsftpd.conf
[o]: https://github.com/r-pufky/ansible_vsftpd/tree/main/defaults/main/config.yml