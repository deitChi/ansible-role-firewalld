Firewalld
=========

[![CI](https://github.com/deitChi/ansible-role-firewalld/actions/workflows/molecule.yml/badge.svg)](https://github.com/deitChi/ansible-role-firewalld/actions/workflows/molecule.yml)

This Role has been created to manage `firewalld` both locally and at scale.
Features include:
- Profile (Template) definitions
- Profile State - Rules not matching profile are removed.
- Common Profiles - Allows re-use of rule sets.
- Check & Diff Mode compatible
- Profile selection using `Inventory`, `Playbook` or `Role`

Requirements
------------

None.

Playbook Variables
------------------

These variables are intended to be stored in the playbook, as they ultimately decide how this role is consumed, and are intended to be changed on a per-use basis.

```yaml
common:
  rich_rules:
    limited_ssh:
      - 'rule family="ipv4" service name="ssh" accept limit value="1/s"'
    limited_ssh_notice_log:
      - 'rule family="ipv4" service name="ssh" log prefix="SSH Access" level="notice" accept limit value="1/s"'
    icmp:
      - 'rule protocol value="icmp" accept'
  services:
    web:
      - http
      - https
    tls_only:
      - https
    ssh:
      - ssh
  ports:
    checkmk_agent:
      - 6556/tcp
    checkmk_livestatus:
      - 6557/tcp
```

In the `common` map you can declare frequently used services - i.e. ssh limit rules, or services that apply to multiple profiles.

```yaml
profiles:
  - name: public-ssh
    zone: public
    target: DROP
    sources: []
    ports:
      rules: []
    services:
      rules: []
    rich_rules:
      common_profiles: [limited_ssh, icmp]
      rules: []
```

In the `profiles` list of maps, you declare your firewall profiles. The use of `use_common` merges any common profile rules with the same name, to the Zone.

Role Variables
--------------

Available variables are listed below, along with default values (see defaults/main.yml):

```yaml
cfg_DefaultZone: public
```

Default Zone (`iptables` CHAIN) Any rules added manually without declaring `--zone` land here

`Please note`: In RHEL8 the default behaviour is the default nic & default zone are linked, whereas they could be separated in the previous RHEL versions. This role ties the two.

```yaml
cfg_CleanupOnExit: yes
```

If set to no or false the firewall configuration will not get cleaned up on exit or stop of firewalld

```yaml
cfg_Lockdown: no
```

If set to enabled, firewall changes with the D-Bus interface will be limited to applications that are listed in the lockdown whitelist.

```yaml
cfg_IPv6_rpfilter: yes
```

Performs a reverse path filter test on a packet for IPv6. If a reply to the packet would be sent via the same interface that the packet arrived on, the packet will match and be accepted, otherwise dropped.

```yaml
cfg_IndividualCalls: no
```

Do not use combined -restore calls, but individual calls. This increases the time that is needed to apply changes and to start the daemon, but is good for debugging.

```yaml
cfg_LogDenied: all # default: off
```

Add logging rules right before reject and drop rules in the INPUT, FORWARD and OUTPUT chains for the default rules and also final reject and drop rules in zones. Possible values are: all, unicast, broadcast, multicast and off.

```yaml
cfg_FirewallBackend: iptables
```

Selects the firewall backend implementation.
Choices are:
- nftables (default)
- iptables (iptables, ip6tables, ebtables and ipset)

```yaml
cfg_FlushAllOnReload: yes
```

Flush all runtime rules on a reload. In previous releases some runtime configuration was retained during a reload, namely; interface to zone assignment, and direct rules. This was confusing to users. To get the old behavior set this to "no".

```yaml
cfg_RFC3964_IPv4: yes
```

As per RFC 3964, filter IPv6 traffic with 6to4 destination addresses that correspond to IPv4 addresses that should not be routed over the public internet.

```yaml
cfg_AllowZoneDrifting: no
```

Older versions of firewalld had undocumented behavior known as "zone drifting". This allowed packets to ingress multiple zones - this is a violation of zone based firewalls. However, some users rely on this behavior to have a "catch-all" zone, e.g. the default zone. You can enable this if you desire such behavior. It's disabled by default for security reasons.
Note: If "yes" packets will only drift from source based zones to interface based zones (including the default zone). Packets never drift from interface based zones to other interfaces based zones (including the default zone).

```yaml
remove_ufw: True
```

Controls if UFW (uncomplicated firewall - Ubuntu Package) should be removed as part of the installation.

```yaml
disable_timestamp_request_reply: True
```

Add a direct rule, to prevent timestamp reply requests.

```yaml
inventory_var: fwd_zone
```

The variable specified in your inventory for the template `name`

```yaml
skip_prompts: False
```

During the playbook, there are various AYS (Are you sure?) Stages that are there to prevent unintentional firewall modifications. Disabled for CI/CD

Host | Group Vars
-----------------

```yaml
fwd_zone: ['public-ssh']
```

This is the same value that you set with `inventory_var`, but declared in the Inventory file. Essentially selecting which of the templates you declared that you wish to add to your host.


Dependencies
------------

None.

Example Playbook
----------------

Declare your Firewall Profiles, and call the role.

```yaml
- hosts: servers
  gather_facts: true
  become: true
  vars:
    profiles:
      - name: public-http
        zone: public
        target: default
        sources: []
        ports:
          rules: []
        services:
          rules:
            - http
            - https
        rich_rules:
          rules: []

      - name: public-ssh
        zone: public
        target: DROP
        sources: []
        ports:
          rules: []
        services:
          rules:
            - ssh
        rich_rules:
          rules: []

      - name: patching-ssh
        zone: patching
        target: DROP
        sources:
          - 192.168.10.0/24
          - 192.168.9.5
        ports:
          rules: []
        services:
          rules: []
        rich_rules:
          rules:
            - 'rule protocol value="icmp" accept'
            - 'rule family="ipv4" service name="ssh" accept limit value="1/s"'

      - name: mysql
        zone: mysql
        target: Reject
        sources:
          - 172.10.30.5/32
          - 172.10.30.6/32
        ports:
          rules:
            - 3306/tcp
        services:
          rules: []
        rich_rules:
          common_profiles: [limited_ssh, icmp]
          rules: []

  roles:
    - deitchi.firewalld
```

Example Inventory
-----------------

Ensure your hosts have a profile set in the inventory

```ini
[servers:vars]
fwd_zone=['patching-ssh']
[servers:children]
mysql
webserver

[mysql:vars]
fwd_zone=['patching-ssh','mysql']
[mysql]
dbserver1

[webserver:vars]
fwd_zone=['patching-ssh','public-http']
[webserver]
webserver1

[ansible]
ansible-server1 ansible_host=192.168.9.5
ansible-server2 ansible_host=192.168.9.6
```

Going Further
-------------

Using this model, it is also possible to create a dynamic Firewall Profile based on inventory hosts.

Using this [script](https://gist.github.com/deitChi/86ac8ad53379ef977cd97e4bdf4aa134), we can select an inventory group, and add the hosts to a sources list.

```yaml
profiles:
  - name: patching-ssh
    zone: patching
    target: DROP
    # Get ansible inventory var from python script
    sources: "{{ lookup('pipe','../lookup_inventory.py -g ansible -ip') }}" 
    ...
```

In this example, we're using the same Profile as before, but this time using a script to return a list of IP addresses from the `inventory` from the example earlier. `-g ansible` specifies the Inventory Group.

The result is the equivalent of:
```yaml
profiles:
  - name: patching-ssh
    zone: patching
    target: DROP
    sources:
      - 192.168.9.5
      - 192.168.9.6
    ...
```

License
-------

MIT

Author Information
------------------

This Role was created in 2023 by Roman Collyer