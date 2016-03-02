---
layout: post
title: "Starting a new Ubuntu Server   Checklist"
description: ""
category: 
tags: []
---

A small checklist when starting a new server. Just to do not waste time boostrapping tests. 

1. <input type="checkbox"> Add a new user for the application:

        # Pick a strong password
        adduser username

2. <input type="checkbox"> Add a new user to the "sudo" group

        gpasswd -a username sudo

3. <input type="checkbox"> Logged in as the new user, generate a ssh key pair - usually used for readonly access of a private hosted project so we're able to fetch and release new versions.

        ssh-keygen -t rsa -b 4096

4. <input type="checkbox"> Add your local public ssh key (workstation) to the remote server by using ssh-copy-id or by copying it inside the `~username/.ssh/authorized_keys`

        ssh-copy-id demo@SERVER_IP_ADDRESS

5. <input type="checkbox"> Disallow remote SSH access to the root.

        sudo vim /etc/ssh/sshd_config

        # Find for the line: PermitRootLogin yes
        # and change it to: PermitRootLogin no

        # Save the file and restart the SSH daemon
        sudo service ssh restart
