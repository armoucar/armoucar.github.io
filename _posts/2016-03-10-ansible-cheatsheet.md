---
layout: post
title: "Ansible Cheatsheet"
description: ""
category:
tags: []
---

The idea is to always update this post with important concepts I use in Ansible, basically for future reference.

## Hosts file

`filename: hosts`

This file is responsible for put together information about your servers. In this file you will have information about the location of the servers (`ip`) how to access them (`ssh user` & `ssh key`). You also give your servers names.

Example:

    main_server ansible_ssh_host=127.0.0.1 ansible_ssh_port=2222
     ansible_ssh_user=vagrant
     ansible_ssh_private_key_file=.vagrant/machines/default/virtualbox/private_key

As you can see, as you get more servers inside ansible, the verbosity of this file can be enormous. You likely want to have a `ansible.cfg` file which will help you to set as default some of this data.

## Ansible config file

`filename: ansible.cfg`

You can define this file in different directories. They have a precedence load order:

    1. ANSIBLE_CONFIG environment variable
    2. ./ansible.cfg
    3. ~/.ansible.cfg
    4. /etc/ansible/ansible.cfg

I usually want the `ansible.cfg` inside my project.

    [defaults]
    hostfile = hosts
    remote_user = vagrant
    private_key_file = .vagrant/machines/default/virtualbox/private_key

Then you can reduce the definitions of your `hosts` file to something like:

    production_server  ansible_ssh_host=xxx.yyy.zzz.kkk
    staging_server     ansible_ssh_host=xxx.yyy.zzz.kkk
    development_server ansible_ssh_host=xxx.yyy.zzz.kkk

### Grouping servers

You can create group of servers:

    webserver1 ansible_ssh_host=xxx.yyy.zzz.kkk
    webserver2 ansible_ssh_host=xxx.yyy.zzz.kkk
    db ansible_ssh_host=xxx.yyy.zzz.kkk

    [web]
    webserver1
    webserver2

    [db]
    webserver1
    webserver2
    db

Then you can execute ansible commands in a specific server or group:

    ansible web -a "date"

### Host and group variables

{% raw %}
 - `filename: host_vars/{{host_name}}`.
 - `filename: group_vars/{{group_name}}`.
{% endraw %}

Ex:

- `host_vars/webserver1`
- `host_vars/webserver2`
- `group_vars/production`
- `group_vars/db`,
- `group_vars/production/mysql`

These files should be either in the playbook directory or in the same directory as the inventory file (`hosts`). You can define variables inside these files as `.ini` or `.yaml` format. My preferred is `yaml` because it is more flexible: supports `booleans`, `strings`, `lists` and `dictionaries`.

## Playbooks

A playbook is a list of tasks to be executed on hosts. They use the `YAML` format and they are executed by using the command line tool `ansible-playbook`.

### Executing a playbook

Here's a minimalist playbook (filename: `helloworld.yml`):

    - hosts: ubuntu
      tasks:
        - debug: msg="Hello World!"

To execute this playbook:

    ansible-playbook helloworld.yml

### Variables

#### Playbook variables

Inside a playbook, just define a `dictionary` called `vars`. Example:

{% raw %}
    - name: defining variables
        vars:
          greeting: "hello"
        tasks:
          - name: output a message
            debug: msg="{{ greeting }}"
{% endraw %}

#### Registering variables

Register the output of a task to a variable

{% raw %}
    - name: capture output
      command: whoami
      register: output
    - debug: msg="{{ output.stdout }}"
{% endraw %}

#### Command line variables

{% raw %}
    ansible-playbook example.yml -e my_name=arthur
    ansible-playbook example.yml -e 'my_name="my name is arthur"'
{% endraw %}


#### Built-in variables

`hostvars`, `inventory_hostname`, `group_names`, `groups`, `play_hosts`, `ansible_version`

{% raw %}
    - name: Checking built-in variables
      hosts: server1
      tasks:
        - debug: msg="{{ inventory_hostname }}"
        - debug: msg="{{ group_names }}"
        - debug: msg="{{ groups }}"
        - debug: msg="{{ play_hosts }}"
        - debug: msg="{{ ansible_version }}"
{% endraw %}


##### Hostvars - getting global facts from other hosts.

If a host needs to get any information from the others hosts, this can be done through the `hostvars` builtin variable.

{% raw %}
    - debug: msg="{{ hostvars['server1'].ansible_eth1.ipv4.address }}"
{% endraw %}

### Options

#### Serial

Serial limits the number of concurrent executing hosts. On the case below, it'll execute on 2 hosts at a time.

{% raw %}
    - name: Execute only 2 hosts in parallel
      hosts: ubuntu
      serial: 2
      tasks:
        - debug: msg="{{ ansible_version }}"
{% endraw %}

#### Running only once

Sometimes a task should run just once, even if you're executing tasks in multiple hosts.

    - name: execute migrations
      run_once: true
      command: /project/execute_migrations

### Filters

Filters added of register variables can be used to detect the state of a command or module executed.

    - shell: /usr/bin/foo
      register: result
      ignore_errors: True

    - debug: msg="it failed"
      when: result|failed

    - debug: msg="it changed"
      when: result|changed

    - debug: msg="it succeeded"
      when: result|success

    - debug: msg="it was skipped"
      when: result|skipped

Code above was copied from the [filters documentation](http://docs.ansible.com/ansible/playbooks_filters.html).

#### Catching Errors

Register the result to a variable, ignore errors in case it happens and then display a debug message in case an error happened.

{% raw %}
    - hosts: ubuntu
      tasks:
      - name: Run inexistent program to emulate error
        command: /opt/inexistent
        register: err
        ignore_errors: true

      - debug: msg="Stop running the playbook - /opt/existent has failed -- {{ err.msg }}"
        failed_when: err|failed
{% endraw %}

### Lookups

Ansible provide a way of accessing data on external resources. 

file: `lookup.yml`

{% raw %}
    - hosts: all
      vars:
        ssh_key: "{{ lookup('file', '~/.ssh/id_rsa.pub') | default('') }}"
        email: "fake@gmail.com"
      tasks:
        - debug: msg="{{ ssh_key  }}"
        - debug: msg="{{ lookup('pipe', 'git branch') }}"
        - debug: msg="{{ lookup('env', 'PATH') }}"
        - debug: msg="{{ lookup('template', './my_template.j2') }}"
{% endraw %}

file: `my_template.j2`

{% raw %}
    Sending an email to {{ email }}
{% endraw %}

### Loops

Check out the 'Loops' [docs](http://docs.ansible.com/ansible/playbooks_loops.html).

### Modules

#### Executing commands - [docs](http://docs.ansible.com/ansible/command_module.html)

    - command: whoami

### Getting information about the server (or Facts)

    ansible server1 -m setup
    ansible server1 -m setup -a "filter=*ipv4*"

## Vaults

Ansible has a way of dealing with sensible data, like passwords. You can encrypt files using the tool `ansible-vault`, most of the cases variable files which sensible data.

    ansible-vault encrypt encrypted_vars.yml

    ansible-playbook tasks.yml --ask-vault-pass
    ansible-playbook tasks.yml --vault-password-file password.txt

    ansible-vault view encrypted_vars.yml
    ansible-vault edit encrypted_vars.yml
    ansible-vault decrypt encrypted_vars.yml
    ansible-vault rekey encrypted_vars.yml

## Roles

Roles are a way of scaling Ansible up by splitting playbooks in different files. A role has at least a main task named `main.yml`. Optionally, It can also have different files with different responsibilities:

    roles/role_name/tasks/main.yml    # main task
    roles/role_name/files/            # files uploaded to the remote host
    roles/role_name/templates/        # j2 templates
    roles/role_name/handlers/main.yml # handlers
    roles/role_name/vars/main.yml     # variables not to be overridden
    roles/role_name/defaults/main.yml # default variables
    roles/role_name/meta/main.yml     # dependencies

### Importing roles

    - hosts: all
      roles:
        - base
        - webserver
        - database

### Dependencies

file: `meta/main.yml`

    dependencies:
        - { role: web }
        - { role: db }

### Ansible Galaxy

[Ansible Galaxy](https://galaxy.ansible.com/) - reuse and share ansible playbooks.

    # Install & uninstall
    ansible-galaxy install username.rolename
    ansible-galaxy install username.role -p ./roles
    ansible-galaxy remove username.rolename

    # Scaffolding a new role
    ansible-galaxy init rolename

    # Search
    ansible-galaxy search elasticsearch
    ansible-galaxy info username.role_name

## Trial n' Error

List of stuff that I racked my brains, but once got working.

### Create an new user with a password

`user_password` and `user_password_salt` are variables I keep inside a vault.

{% raw %}
    - name: Generating user's password
      command: "/usr/bin/python -c 'import crypt; print crypt.crypt(\"{{ user_password }}\", \"{{ user_password_salt }}\")'"
      register: encrypted_password

    - name: Create application user
      user: name={{ user_name }} password={{ encrypted_password }} group={{ group_name }} groups="sudo" shell=/bin/bash home={{ user_home_path }}
{% endraw %}
