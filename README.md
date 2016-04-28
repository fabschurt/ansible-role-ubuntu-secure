# Ubuntu Secure System (14.04)

[![Travis CI](https://img.shields.io/travis/rust-lang/rust.svg)](https://travis-ci.org/fabschurt/ansible-role-ubuntu-secure)

This Ansible role aims at securing a Ubuntu box as much as possible, without
hindering performance or flexibility too much. The choices made here are
inevitably biased and opinionated, but I'd gladly study proposals to make
this role more universal.

Full compatibility has been tested on **Ubuntu 14.04**, but I guess it should
work fine with later and (not too old) earlier versions too.

There's still a lot of work to do, but we've got a good start here. For now,
this role will basically:

* enable APT unattended upgrades (`security` and `updates` channels)
* completely disable IPv6 support (that will be kind of controversial I guess)
* create a non-root (but *sudoer*) master user that will become the only entry
  point to the host: the *gatekeeper*
* install some pretty restrictive `iptables` rules, including:
    - `DROP` as default policy
    - basic ICMP/TCP/UDP flood protection
    - only the `sshd` input port left open
    - only some vital output ports left open

**BE CAREFUL**, once you've applied this role, the target host(s) will be accessible
by the gatekeeper user only, with public key authentication only (root login and
password authentication will be disabled). You will have to change your inventory
and/or command line options to take this into account.

*Note:* this role includes a final step that will reboot the target host(s).

## Requirements

* Ubuntu remote host(s) with root access
* Ansible >= 2.0

## Role variables

This role is configurable with the following variables:

* `ubuntu_secure_sshd_port`: the TCP port that `sshd` will listen to
* `ubuntu_secure_sshd_max_startups`: the value of the `MaxStartups` config
  directive in `sshd_config`
* `ubuntu_secure_iptables_additional_rules`: additional `iptables` rules that
  will be appended to the default ones (you should use a [literal](https://en.wikipedia.org/wiki/YAML#Block_literals)
  YAML string here)
* `ubuntu_secure_gatekeeper_name`: the gatekeeper's username
* `ubuntu_secure_gatekeeper_password`: the gatekeeper's password (mandatory if
  you want to `sudo`)
* `ubuntu_secure_gatekeeper_public_keys`: a list of local paths to public key
  files whose content will be copied to the gatekeeper's `authorized_keys` file

See the **Example playbook** section below for a reference of these variables'
default values.

## Example playbook

This is an example playbook that demonstrates how you would use the role, provided
that you've [installed](https://galaxy.ansible.com/intro#download) it already.
The variable values used here reflect the default values declared in `defaults/main.yml`:

```yaml
- hosts: servers
  roles:
    - role: fabschurt.ubuntu-secure
      ubuntu_secure_sshd_port: 22
      ubuntu_secure_sshd_max_startups: '5:50:10' # Be sure to use quotes here
      ubuntu_secure_iptables_additional_rules: ''
      ubuntu_secure_gatekeeper_name: gatekeeper
      ubuntu_secure_gatekeeper_password: '' # You should really override this one, otherwise you won't be able to sudo
      ubuntu_secure_gatekeeper_public_keys: [~/.ssh/id_rsa.pub]
```

The `ubuntu_secure_gatekeeper_password` value must be in encrypted form,
see [here](http://docs.ansible.com/ansible/faq.html#how-do-i-generate-crypted-passwords-for-the-user-module)
for more information.

## License

This software package is licensed under the [MIT License](https://opensource.org/licenses/MIT).
