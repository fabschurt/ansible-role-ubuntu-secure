# Ubuntu Secure System (14.04)

[![Travis CI](https://img.shields.io/travis/rust-lang/rust.svg)](https://travis-ci.org/fabschurt/ansible-role-ubuntu-secure)

This role will create a user that will become the only point of entry on the
targeted hosts. For convenience' sake (and because it's awesome), this user will
be referred to as the *gatekeeper* as of now.

**BE CAREFUL**, once you've applied this role, the targeted hosts will be accessible
by the gatekeeper user only, with public key authentication only (root login and
password authentication will be disabled). You will have to change your inventory
and/or command line options to take this into account.

This role is continuously integrated on [Travis](https://travis-ci.org/fabschurt/ansible-role-ubuntu-secure).
For now, it's simply syntax-checked against multiple Ansible versions, to check
for basic compatibility.

It's been tested against **Ubuntu 14.04**, but should work on earlier and later
versions, as there's nothing really 14.04-specific in there.

## Requirements

* Ubuntu remote host(s) with root access
* Ansible >= 2.0

## Role variables

This role is configurable with the following variables:

* `gatekeeper_name`: the gatekeeper's username
* `gatekeeper_password`: the gatekeeper's password (to be able to `sudo`)
* `gatekeeper_public_keys`: a list of local paths to public key files that will
  be copied to the gatekeeper's `authorized_keys` file

See the **Example playbook** section below for a reference of these variables'
default values.

## Example playbook

This is an example playbook that demonstrates how you would use the role, provided
that you've [installed](https://galaxy.ansible.com/intro#download) it already.
The variable values used here reflect the default values declared in `defaults/main.yml`:

```yaml
- hosts: servers
  roles:
    - role: ansible-role-ubuntu-secure
      gatekeeper_name: gatekeeper
      gatekeeper_password: '' # You should really override this one, otherwise you won't be able to sudo
      gatekeeper_public_keys: [~/.ssh/id_rsa.pub]
```

The `gatekeeper_password` value should be has to be in an encrypted form,
see [here](http://docs.ansible.com/ansible/faq.html#how-do-i-generate-crypted-passwords-for-the-user-module)
for help.

## License

This package is licensed under the [MIT license](https://opensource.org/licenses/MIT).
