# vsftpd
vsftpd installation.

## Requirements
No additional requirements.

## Role Variables
Settings have been throughly documented for usage.

[defaults/main.yml](https://github.com/r-pufky/ansible_vsftpd/blob/main/defaults/main.yml).

## Dependencies
N/A

## Example Playbook
This example sets up a FTP server only allowing virtual users to login and
upload files, chrooting them to that directory. Virtual user information is
encrypted and stored in an htpasswd formatted file.

host_vars/vsftpd.example.com/vars/vsftpd.yml
``` yaml
vsftpd_virtual_users:
  - {user: 'virtual_user', pass: 'some password in vault'}
vsftpd_local_root_perms:        '0770'
vsftpd_chroot_local_user:       true
vsftpd_guest_enable:            true
vsftpd_hide_ids:                true
vsftpd_local_enable:            true
vsftpd_virtual_use_local_privs: true
vsftpd_write_enable:            true
vsftpd_local_root:              '/srv/ftp'
vsftpd_allow_writeable_chroot:  true
```

site.yml
``` yaml
- name:   'vsftpd server'
  hosts:  'vsftpd.example.com'
  become: true
  roles:
     - 'r_pufky.vsftpd'
```

## Issues
Create a bug and provide as much information as possible.

Associate pull requests with a submitted bug.

## License
[AGPL-3.0 License](https://github.com/r-pufky/ansible_vsftpd/blob/main/LICENSE)

## Author Information
https://keybase.io/rpufky
