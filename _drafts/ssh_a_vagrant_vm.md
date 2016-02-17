---
title:  "SSH a Vagrant VM"
categories: tools
tags: tips
---

Vagrant provides a easy-peasy way of connecting to the VM that was created.

    $ vagrant ssh

I'm experimenting [Ansible](ansible.com) - a provisining automation tool. It uses *SSH* to connect to the remote machine and execute the commands it needs to provision a new computer instance. For that we need to know the *SSH* parameters so it is able to connect to our instance. You can check the *SSH* configs with the following command:

    $ vagrant ssh-config
    Host default
      HostName 127.0.0.1
      User vagrant
      Port 2222
      UserKnownHostsFile /dev/null
      StrictHostKeyChecking no
      PasswordAuthentication no
      IdentityFile /Users/arthur/dev/vagrant-ssh-test/.vagrant/machines/default/virtualbox/private_key
      IdentitiesOnly yes
      LogLevel FATAL

The important lines here are:

      HostName 127.0.0.1
      User vagrant
      Port 2222
      IdentityFile /Users/arthur/dev/vagrant-ssh-test/.vagrant/machines/default/virtualbox/private_key

Using the information collected you can now connect using plain *SSH* as shown below:

    ssh vagrant@127.0.0.1 -p 2222 -i /Users/arthur/dev/vagrant-ssh-test/.vagrant/machines/default/virtualbox/private_key
