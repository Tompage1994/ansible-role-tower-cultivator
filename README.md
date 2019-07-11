Role Name
=========

Ansible Role to Seed Ansible Tower with some simple initial settings.

NOTE: Currently only Organisations are seeded, but more objects will be added soon.


Requirements
------------

The role heavily utilises the existing `tower_*` Ansible modules, which in turn leverage the `tower-cli`. This role installs `tower-cli` using the `pip` ansible module, into a Python Virtualenv on the target system (I usually target one of the Tower hosts).

The role installs all pre-requisites on the target (eg `python-virtualenv` and `python-setuptools`) to complete the above tasks, but it does require access to the appropriate RHEL channels and access to PyPi.

Role Variables
--------------

Available variables are listed below, along with default values (see defaults/main.yml):

    # VirtualEnv that Cultivator uses as a working/install dir
    cultivator_virtualenv_path: "/opt/cultivator"

    # Base Packages
    cultivator_base_packages:
        - python-virtualenv
        - python-setuptools

    # Packages to install with pip
    cultivator_pip_packages:
        - ansible-tower-cli

    # Dictionary of Tower orgs to seed.
    cultivator_tower_orgs:
        - { name: "default", description: "Default Organisation" }


    # URL of the Tower API instance/host
    cultivator_tower_server: https://my-tower.example.com

    # User to login to Tower
    cultivator_tower_username: admin

    # Password for the user above
    cultivator_tower_password: towerpassword

    # Should certs be validated? Set false if using self-signed certs
    cultivator_tower_validate_certs: true


Dependencies
------------

    `tower_*` - standard modules included with Ansible >= 2.3


Example Playbook
----------------

---
# Playbook to seed Ansible Tower

- name: "[Ansible Tower] Cultivate Tower"
  hosts: tower
  gather_facts: no

  vars:
    cultivator_tower_server: https://tower.example.com
    cultivator_tower_username: myuser
    cultivator_tower_password: mysecurepassword

  roles:
  - role: tower-cultivator


License
-------

MIT


Author Information
------------------

Dan Hawker [Github](https://github.com/danhawker)
