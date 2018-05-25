# Ansible :: Ubuntu secure system

[![Travis CI](https://img.shields.io/travis/fabschurt/ansible-role-ubuntu-secure/master.svg)](https://travis-ci.org/fabschurt/ansible-role-ubuntu-secure)

This is a largely work-in-progress Ansible role aiming at securing a Ubuntu
installation in the most sensible way possible. There’s still a lot of work to
be done, but I think we’ve got a pretty good start here.

**BE VERY CAREFUL** before applying this role and make sure you understand the
(non-trivial) changes it will implement.

This role is targeted at **Ubuntu LTS 18.04**, but I guess it should work with
any recent Ubuntu flavor.

Here’s a (non-exhaustive) list of the changes that it will implement:

* default SSH port will be changed to a user-defined value
* SSH root login and password authentication will be disabled
* some pretty restrictive *iptables* rules will be set up, including:
    - *DROP* as default policy for all chains
    - basic ICMP/TCP/UDP flood protection
    - only the SSH input port will be left open
    - only some vital output ports will be left open
* a secure *umask* value of 077 will be (tentatively) enforced
* the existence of a sudoer, public-key-authenticating main admin user will be
  guaranteed

**CAUTION:** as mentioned earlier, the default SSH port will be changed, so
immediately after applying this role, you will have to edit the connection port
of the concerned hosts in your inventory, otherwise you won’t be able to get
access to them anymore. I realize that this process is not ideal; I’ll try to
think about a better, more automated solution.

## Requirements

* Ubuntu 18.04 remote host(s) with root access
* Ansible >= 2.4

## Role variables

This role is configurable with the following variables:

* `ssh_alternate_port`: the TCP port that the SSH daemon will listen to
* `iptables_additional_rules`: additional `iptables` rules that will be appended
  to the default ones (you should use a YAML [block literal](https://en.wikipedia.org/wiki/YAML#Indented_delimiting)
  string here)
* `admin_name`: the username of the main admin user; this user may or may not
  already exist on the system
* `admin_password`: the admin user’s (encrypted, see [here](https://docs.ansible.com/ansible/latest/reference_appendices/faq.html#how-do-i-generate-crypted-passwords-for-the-user-module))
  password; mandatory if you want to use `sudo`
* `admin_pubkey_path`: a URL or local path to a SSH public key with which the
  admin user will be allowed to log into the system

See the [Example playbook](#example-playbook) section below for a reference of
these variables’ default values.

## Example playbook

This is an example playbook that demonstrates how you would use the role,
provided that you’ve [installed](https://galaxy.ansible.com/intro#download) it
already. The variable values used here reflect the default values declared in
`defaults/main.yml`:

```yaml
- hosts: …
  roles:
    - role: fabschurt.ubuntu-secure
      ssh_alternate_port: 222 # I highly recommend you override this one with a custom value
      iptables_additional_rules: ''
      admin_name: admin
      admin_password: '' # You should really override this one, otherwise you won’t be able to sudo
      admin_pubkey_path: ~/.ssh/id_rsa.pub
```

## License

This software package is licensed under the [MIT License](https://opensource.org/licenses/MIT).
