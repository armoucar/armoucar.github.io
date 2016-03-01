---
layout: post
title: "Rails - production ready in few steps"
description: ""
category: 
tags: []
---

Want to skip my whining tale and go straight to the tutorial? Go to next topic clicking [here](#quickguide).

## A small tale - why learn Ansible

While ago, I was pretty busy learning the basics of Ruby and Rails - I had just got in a company and my background was in Java (mostly). A more experienced developer, a mentor for me and only teammate, was steps ahead working with automated provisioning of our servers using [Chef](https://www.chef.io/).

It is a small company, it was just me and him on the Tech team. Some time after this developer left the company and I had to take care of our infrastructure as well. So I learned a bit of Chef and short time later I realized I was spending too much time on it. It is a great tool, but I wasn't able to spend all this time on it.

It has lots of small command line tools. You start learning some with their official tutorials to after understand that those are just concepts and you'll work with different tools (e.g. chef-apply, chef-solo etc.). You have to configure a Chef server to manage you servers and if you're not willing to pay for their service, either you create your own chef server - and spend thousands of hours getting crazy digging the opensource project - or you're limited by having only 5 servers.

Chef was the first provisioning tool I was dealing with and I didn't know other options - had heard of Puppet once but not more than that. We're still using Chef in there, but I had to try to find something simpler - mainly for small projects where I would consider Chef overengineering. I found then [Ansible](https://www.ansible.com/) and in many comparison articles, it was recommended as a simpler tool. Maybe with more restrictions then Chef (where the base for recipes is pure Ruby), but a simplicity that would pay off at the end.

I have set up my development environment and even production environment step by step quite a few times. With Chef, the automation of this process would be called *cookbooks*, while in Ansible those are called *playbooks*.

I read a book about Ansible, by [O'Reilly](http://shop.oreilly.com/product/0636920035626.do), and wrote my own playbooks to automate my process of setting up a Rails environment.

This isn't a blogpost where I go through Ansible concepts though, I just had to make a point about why it was necessary and how did I come to learn how to setup this rails development/production environment very quick.

Doing more and more researches on *interwebs* about Ansible, I find this really cool project called [railsbox](https://railsbox.io/) and this just makes a bootstrap much easier. It uses Ansible to provision your servers with a complete Rails environment. Have a look on this project, it is very useful. At the end of the day, after learn a provision tool, it is not there where you want to spend your time... You want to bootstrap your environment as soon as you can and start focusing on the business side.

Railsbox will guide you through the packages and configurations you want to install. So you can pick a ruby installer (rvm, rbenv), add different environments (remote server, vagrant), database (mysql, postgres, mongo...), and background jobs (sidekiq, resque).

<a name="quickguide"/>

## Bootstraping your Rails environment quickly

If you run into any problem, make sure you check the [Troubleshooting session](#troubleshooting) of this article. Now, let's get cracking.

1. Start a rails application

        $ rails new library -d mysql

2. Create a git repository and push it to a remote server. You'll need the git URL in the next step.

3. Drop the project's Gemfile on *[railsbox](https://railsbox.io/)* so it'll help you to pre-populate the form. Fill all the form with your set up and then click 'Create box' (I recommend you to not put a database password at this time. We're going to configure one manually to make sure it doesn't leak). I'll setup my environment with:

    - Ubuntu Trusty 14.04 LTS 64
    - rbenv
    - Ruby 2.2.1
    - VirtualBox development environment
    - Remote Server production environment
    - nginx + passenger
    - MySQL database - I'll do my own configuration

4. Download the created box. Unzip it inside the rails project we just created. Add `railsbox` folder to git and commit it.

### development env

    cd railsbox/development
    vagrant up

After finished, check that you have Rails 'Welcome Aboard' page running in <a href="http://localhost:8080" target="_blank">http://localhost:8080</a>

Inside the Vagrant VM, the project is available under the `/project_name` directory. In the case of this tutorial, `/library`.

You can restart passenger by running `passenger-config restart-app` and selecting your app.

### production env

Before execute any command, let's inform railsbox some data about our production server. Open the file `railsbox/ansible/group_vars/production/config.yml` and fill all the variables.

For this test, I'm using a [DigitalOcean](https://www.digitalocean.com/) droplet with *512 MB Memory* and *20 GB Disk*.

So thats my final config file:

{% highlight yaml %}
rails_env: production
target: server
vm_memory: 512
vm_swap: 512 # https://help.ubuntu.com/community/SwapFaq#How_much_swap_do_I_need.3F
vm_cores: 1
vm_share_type: 
vm_ip: 
host: ***.***.***.*** # Your server's public IP goes here.
port: 22
username: root
{% endhighlight %}

Now:

    cd railsbox/production
    sh provision.sh

Some tasks might fail but don't worry, if they are ignored they are supposed to fail - like execute `rails db:migrate` for the current version when the app is still not cloned.

Inside the same folder, there's a second script called `deploy.sh` and you might guess what it is. Before execute this script, make sure you generate a server SSH key pair which will authenticate your server with your git repository and then be able to clone your repository and fetch new versions. Make sure that you authenticate with the new applications' user, which was created during the provisioning script. The user created has the same name as your application:

    sudo su - library

[Generate the SSH key is out of this tutorial's scope](https://help.github.com/articles/generating-an-ssh-key/) but if you have any problems, give a shout.

#### secrets.yml

We shall now generate the rails production secret code with rake. Make sure you're logged in as the app user.

    sudo su - library
    cd /library/current
    bundle exec rails secret  # copy the generated key
    touch ../.envrc           # railsbox courtesy

Then edit ../.envrc

    export SECRET_KEY_BASE=paste_the_generated_key_here

And finally, restart Passenger:

    passenger-config restart-app

Now try to access your production server in the browser and see that an error will come up. It's normal, the error happens because we haven't defined a root controller in Rails.

#### Database password

We have installed mysql but we haven't defined a password. I preferred not input it in railsbox or even inside our codebase to keep it secure only inside our production server. So, inside the server let's define one and then define a environment variable to store it.

    # pick a strong password
    mysqladmin -u root password VD#mAe*yz@q8

    # then edit the /library/.envrc file
    # Pay attention to the variable name. It start with the APP_NAME
    export LIBRARY_DATABASE_PASSWORD=VD#mAe*yz@q8

    # Save the file and then load it
    source /library/.envrc

To test that the database authentication is working let's use the rails console:

{% highlight ruby %}
ActiveRecord::Base.connection.execute("SHOW DATABASES;")
{% endhighlight %}

If the above command executes fine, we're done with all the configuration. Now we have a basic Rails environment ready for production. Provisioning of development and production is automated which will give a boost whenever a new person comes to the team or a new server has to be added to a cluster chain. The current architecture is not ready for clustering, so add a new server is not straightforward. Even so, Ansible is a powerful tool and will help a lot when the architecture is ready to work in cluster.

<a name="troubleshooting" />

## Troubleshooting

### 1. MySQL

#### Socket

    Mysql2::Error: Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)

Connect to the instance and check where the socket file is:

    vagrant@localhost:~$ mysql_config --socket
    /var/run/mysqld/mysqld.sock

Copy the directory location and inside your Rails project, open `config/database.yml`, under `development` configuration write:

{% highlight yaml %}
default: &default
  socket: /var/run/mysqld/mysqld.sock
{% endhighlight %}

### 2. SSH Auth

    Permission denied (publickey).
    fatal: Could not read from remote repository.

    Please make sure you have the correct access rights
    and the repository exists.

Make sure your server has a SSH key and that this SSH key is configured in your *git* server.
