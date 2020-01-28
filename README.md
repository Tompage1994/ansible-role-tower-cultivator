Ansible Cultivator
==================

Ansible Role to Seed Ansible Tower with some simple initial settings.

NOTE: Currently Organisations, Users, Projects, Inventories and Job Templates are seeded, but the role can be easily expanded to handle all Tower objects.

Users can seed Tower objects by passing a variable `cultivator_tower_objects` that contains a dictionary of objects. The standard `default/main.yml` file, contains an example set of high level objects that can be used as a guide. Each object has a distinct set of variables that it can consume, based upon the appropriate Ansible module employed. For example, the `organisation` object can take `name` and `description` as valid variables. Each object has been documented within the example `cultivator_tower_objects`, use it as a guide for your seeding activities.

An example VARS file has been included to demostrate how to provide the objects.


Requirements
------------

The role heavily utilises the existing `tower_*` Ansible modules, which in turn leverage the `tower-cli`. This role installs `tower-cli` using the `pip` ansible module, into a Python Virtualenv on the target system (I usually target one of the Tower hosts).

The role installs all pre-requisites on the target (eg `python-virtualenv` and `python-setuptools`) to complete the above tasks, but it does require access to the appropriate RHEL/CentOS channels and access to PyPi.


Role Variables
--------------

Available variables are listed below, along with default values (see defaults/main.yml):

    # VirtualEnv that Cultivator uses as a working/install dir on the target host
    cultivator_virtualenv_path: "/opt/cultivator"

    # Base Packages
    cultivator_base_packages:
        - python-virtualenv
        - python-setuptools

    # Packages to install with pip
    cultivator_pip_packages:
        - ansible-tower-cli

    # URL of the Tower API instance/host
    cultivator_tower_server: https://my-tower.example.com

    # User able to login and manage Tower
    cultivator_tower_username: admin

    # Password for the user above
    cultivator_tower_password: towerpassword

    # Should certs be validated? Set false if using self-signed certs
    cultivator_tower_validate_certs: true

    # Dictionary of Tower Objects to seed
    cultivator_tower_objects:
      organisations:
        - name: "example.com"
          description: "Example.com Organisation"
          users:
            - username: "jane.doe"
              first_name: "Jane"
              last_name: "Doe"
              email: "jane.doe@example.com"
              password: "password"
              user_type: "normal"
            - username: "joe.bloggs"
              first_name: "Joe"
              last_name: "Bloggs"
              email: "joe.bloggs@example.com"
              password: "password"
              user_type: "auditor"
          projects:
            - name: "Prefect"
              description: "Ansible Playground Manager"
              scm_url: "https://github.com/danhawker/prefect.git"
              scm_type: git
          inventories:
            - name: "AWS"
              description: "AWS"
          job_templates:
            - name: "Ping"
              job_type: run
              playbook: "playbooks/ping.yml"
              project: "Prefect"
              inventory: "AWS"


Dependencies
------------

    `tower_*` - standard modules included with Ansible >= 2.3


Example Playbook
----------------

The following playbook and accompanying vars file containing the defined seed objects can be invoked in the following manner.

```
$ ansible-playbook seed_tower.yml -e @tower_vars.yml tower
```

```yaml
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
```


Example Vars File
-----------------

```yaml
---
# Vars for objects to seed
cultivator_tower_objects:
  organisations:
    - name: "example.com"
      description: "Example.com Organisation"
      users:
        - username: "jane.doe"
          first_name: "Jane"
          last_name: "Doe"
          email: "jane.doe@example.com"
          password: "password"
          user_type: "normal"
        - username: "joe.bloggs"
          first_name: "Joe"
          last_name: "Bloggs"
          email: "joe.bloggs@example.com"
          password: "password"
          user_type: "auditor"
      credential_type:
        - name: demotype
          inputs:
            fields:
              - id: host
                type: string
                label: Hostname
              - id: password
                type: string
                label: Password
                secret: true
            required:
              - host
              - password
          injectors: "{{ lookup('file', '{{ var_location }}/demo_extra_vars.json') }}"
          # See Ansible Tower docs for example injectors
      credential:
        - name: "cred1"
          kind: demotype
          inputs:
            host: demo.example.com
            password: password
        - name: "cred2"
          kind: ssh
          username: admin
          password: password
      projects:
        - name: "Prefect"
          description: "Ansible Playground Manager"
          scm_url: "https://github.com/danhawker/prefect.git"
          scm_type: git
      inventories:
        - name: "AWS"
          description: "AWS"
      inventory_sources:
        - name: "AWS-source"
          description: "AWS source"
          inventory: "AWS"
          project: "project_1"
          path: "inventory/aws.yml"
      job_templates:
        - name: "Ping"
          job_type: run
          playbook: "playbooks/ping.yml"
          project: "Prefect"
          inventory: "AWS"
      job_template-credentials:
        - job_template: "Ping"
          credential: "cred1"
        - job_template: "Ping"
          credential: "cred2"
      workflow_templates:
        - name: "workflow_1"
          description: "A test workflow"
          schema:
            - job_template: "Ping"
              success_nodes:
              - job_template: "Pong"
```


License
-------

MIT


Author Information
------------------

Dan Hawker [Github](https://github.com/danhawker)
