# Ubuntu Secure System (16.04)

[![Travis CI](https://img.shields.io/travis/fabschurt/ansible-role-ubuntu-secure.svg)](https://travis-ci.org/fabschurt/ansible-role-ubuntu-secure)

This Ansible role aims at securing a Ubuntu box as much as possible, without
hindering performance or flexibility too much. The choices made here are
inevitably biased and opinionated, but I’d gladly study proposals to make
this role more universal.

This role is targeted at **Ubuntu 16.04**, but I guess it should work on any
recent Ubuntu flavor.

There’s still a lot of work to do, but we’ve got a good start here. For now,
this role will basically&nbsp;:

* enable APT unattended upgrades (`security` and `updates` channels)
* completely disable IPv6 support (that will be kind of controversial I guess)
* disable root login and password authentication, and set the listening port
  of the SSH server to an alternative number
* create a non-root *sudoer* user that will become the only entry point to the
  system&nbsp;: the *gatekeeper*
* install some pretty restrictive `iptables` rules, including&nbsp;:
    - `DROP` as default policy
    - basic ICMP/TCP/UDP flood protection
    - only the `sshd` input port left open
    - only some vital output ports left open

**ATTENTION&nbsp;:** immediately after you’ve applied this role to a host, you
will be locked out of it (see the description of applied changes above). I’m
working on a solution in order to have all this handled automatically, but as a
temporary workaround, you will have to edit the connection attributes of the
host in your inventory&nbsp;:

```
[group_name]
…
host_name ansible_port=ubuntu_secure_sshd_port ansible_user=ubuntu_secure_gatekeeper_name ansible_become=true
```

(See below for a description of `ubuntu_secure_sshd_port` and `ubuntu_secure_gatekeeper_name`.)

*Note&nbsp;:* some changes induced by this role will trigger a server reboot.

## Requirements

* Ubuntu remote host(s) with root access
* Ansible >= 2.0

## Role variables

This role is configurable with the following variables&nbsp;:

* `ubuntu_secure_sshd_port`&nbsp;: the TCP port that `sshd` will listen to
* `ubuntu_secure_sshd_idle_timeout`&nbsp;: the number of seconds after which an
  idle SSH session will be closed
* `ubuntu_secure_sshd_max_startups`&nbsp;: the value of the `MaxStartups` config
  directive in `sshd_config`
* `ubuntu_secure_iptables_additional_rules`&nbsp;: additional `iptables` rules
  that will be appended to the default ones (you should use a
  [literal](https://en.wikipedia.org/wiki/YAML#Block_literals) YAML string here)
* `ubuntu_secure_gatekeeper_name`&nbsp;: the gatekeeper’s username
* `ubuntu_secure_gatekeeper_password`&nbsp;: the gatekeeper’s password (mandatory
  if you want to `sudo`)
* `ubuntu_secure_gatekeeper_public_keys`&nbsp;: a list of local paths to public
  key files whose content will be copied to the gatekeeper’s `authorized_keys`
  file

See the **Example playbook** section below for a reference of these variables’
default values.

## Example playbook

This is an example playbook that demonstrates how you would use the role, provided
that you’ve [installed](https://galaxy.ansible.com/intro#download) it already.
The variable values used here reflect the default values declared in `defaults/main.yml`&nbsp;:

```yaml
- hosts: servers
  roles:
    - role: fabschurt.ubuntu-secure
      ubuntu_secure_sshd_port: 222
      ubuntu_secure_sshd_idle_timeout: 900
      ubuntu_secure_sshd_max_startups: '5:50:10' # Be sure to use quotes here
      ubuntu_secure_iptables_additional_rules: ''
      ubuntu_secure_gatekeeper_name: gatekeeper
      ubuntu_secure_gatekeeper_password: '' # You should really override this one, otherwise you won’t be able to sudo
      ubuntu_secure_gatekeeper_public_keys: [~/.ssh/id_rsa.pub]
```

The `ubuntu_secure_gatekeeper_password` value must be in encrypted form,
see [here](http://docs.ansible.com/ansible/faq.html#how-do-i-generate-crypted-passwords-for-the-user-module)
for more information.

## License

This software package is licensed under the [MIT License](https://opensource.org/licenses/MIT).
