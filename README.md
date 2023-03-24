Firewalld
=========

[![CI](https://github.com/deitChi/ansible-role-firewalld/actions/workflows/molecule.yml/badge.svg)](https://github.com/deitChi/ansible-role-firewalld/actions/workflows/molecule.yml)

This Ansible Role is created to allow firewalld to be managed programatically - at scale. The role allows for the users to define firewall profiles, and select them using inventory lists - or apply them directly.

Requirements
------------

None.

Playbook Variables
------------------

These variables are intended to be stored in the playbook, as they ultimately decide how this role is consumed, and are intended to be changed on a per-use basis.

```yaml
common:
  rich_rules:
    - 'rule protocol value="icmp" accept'
    - 'rule family="ipv4" service name="ssh" accept limit value="1/s"'
  services:
    - dhcp
  ports: []
```

In the `common` map you can declare frequently used services - i.e. ssh limit rules, or services that apply to multiple profiles.

```yaml
profiles:
  - name: public-ssh
    zone: public
    target: DROP
    sources: []
    ports:
      use_common: no
      rules: []
    services:
      use_common: no
      rules: []
    rich_rules:
      use_common: yes
      rules: []
```

In the `profiles` list of maps, you declare your firewall profiles. The use of `use_common` merges any common rules with the same name, to the Zone.

Role Variables
--------------

Available variables are listed below, along with default values (see defaults/main.yml):

    remove_ufw: True

Controls if UFW (uncomplicated firewall - Ubuntu Package) should be removed as part of the installation.

    log_denied: True

`firewalld.conf` - log denied & dropped packets (Can be disk intensive)

    disable_timestamp_request_reply: True

Add a direct rule, to prevent timestamp reply requests.

    default_interface_zone: public

Default Zone (Zone that your rules get added to if you do not specify `--zone zonename`) 

`Please note`: In RHEL8 the default behaviour is the default nic & default zone are linked, whereas they could be separated in the previous RHEL versions. This role ties the two.

    inventory_var: fwd_zone

The variable specified in your inventory for the template `name`

    skip_prompts: False

During the playbook, there are various AYS (Are you sure?) Stages that are there to prevent unintentional firewall modifications. Disabled for CI/CD

Host | Group Vars
-----------------

    fwd_zone: ['public-ssh']

This is the same value that you set with `inventory_var`, but declared in the Inventory file. Essentially selecting which of the templates you declared that you wish to add to your host.


Dependencies
------------

None.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

```yaml
- hosts: servers
  gather_facts: true
  become: true
  roles:
    - ../roles/firewalld
```

License
-------

MIT

Author Information
------------------

This Role was created in 2023 by Roman Collyer