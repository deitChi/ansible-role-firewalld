Firewalld
=========

[![CI](https://github.com/deitChi/ansible-role-firewalld/workflows/CI/badge.svg?event=push)](https://github.com/deitChi/ansible-role-firewalld/actions?query=workflow%3ACI)

This Ansible Role is created to allow firewalld to be managed programatically - at scale. The role allows for the users to define firewall profiles, and select them using inventory lists - or apply them directly.

Requirements
------------

None.

Role Variables
--------------

Available variables are listed below, along with default values (see defaults/main.yml):

(To Fix)

Dependencies
------------

None.

Example Playbook
----------------

Including an example of how to use your role (for instance, with variables passed in as parameters) is always nice for users too:

    - hosts: servers
      gather_facts: true
      become: true
      roles:
        - ../roles/firewalld

License
-------

MIT

Author Information
------------------

This Role was created in 2023 by Roman Collyer