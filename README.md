# For New Projects

This project is using the [rails-dev-box](https://github.com/rails/rails-dev-box.git) to get a dev environment setup for a rails app.

## First Time Rails Dev Box users
1. Clone this project into your code folder:
`cd YOUR CODE FOLDER`
`git clone https://github.com/rails/rails-dev-box.git`

1. Move into the new project folder:
`cd rails-dev-box`

1. Get vagrant up and running
`vagrant up`

1. SSH into the vagrant box
`vagrant ssh`

## Creating An App
Now you can create multiple rails apps, each will create a subfolder in the rails-dev-box project. 

1. Make sure vagrant is up and you have ssh'd into the vagrant box
`vagrant up && vagrant ssh`

1. Move to the vagrant folder
`cd /vagrant`

1. Create an app
`rails new APP_NAME`

1. Move into your newly created app
`cd APP_NAME`

At this point I had issues getting my app up and running so I had to perform the following steps:

1. Edit your Gemfile to make sure you're running the correct version of Rails and Puma

Find `gem 'rails', '~> 5.2.3'` and replace it with `gem 'rails', '5.2.1'`

Find `gem 'puma', '~> 3.11' and replace it with `gem 'puma', '3.12'`

Save your changes

1. If you're going to use Postgres add the following to your Gemfile
    
    ```
    # Use Postgres
    gem 'pg'
    ```

1. Edit the config/database.yml file by adding the following
    
    ```
    # Using Postgres, not SQLlite
    default: &default
      adapter: postgresql
      pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
      timeout: 5000

    development:
      <<: *default
      database: db/APP_NAME_dev

    # Warning: The database defined as "test" will be erased and
    # re-generated from your development database when you run "rake".
    # Do not set this db to the same as development or production.
    test:
      <<: *default
      database: db/APP_NAME_test

    production:
      <<: *default
      database: db/APP_NAME_prod
    ```

1. In the console run bundle
`bundle`

1. Create your new databases
`rake db:create:all`

Now you can run your rails app
`rails s` OR `rails c`

-----------------------------------------------------


# A Virtual Machine for Ruby on Rails Core Development

## Introduction

**Please note this VM is not designed for Rails application development, only Rails core development.**

This project automates the setup of a development environment for working on Ruby on Rails itself. Use this virtual machine to work on a pull request with everything ready to hack and run the test suites.

## Requirements

* [VirtualBox](https://www.virtualbox.org)

* [Vagrant 2](http://vagrantup.com)

## How To Build The Virtual Machine

Building the virtual machine is this easy:

    host $ git clone https://github.com/rails/rails-dev-box.git
    host $ cd rails-dev-box
    host $ vagrant up

That's it.

After the installation has finished, you can access the virtual machine with

    host $ vagrant ssh
    Welcome to Ubuntu 18.10 (GNU/Linux 4.18.0-10-generic x86_64)
    ...
    vagrant@rails-dev-box:~$

Port 3000 in the host computer is forwarded to port 3000 in the virtual machine. Thus, applications running in the virtual machine can be accessed via localhost:3000 in the host computer. Be sure the web server is bound to the IP 0.0.0.0, instead of 127.0.0.1, so it can access all interfaces:

    bin/rails server -b 0.0.0.0

## RAM and CPUs

By default, the VM launches with 2 GB of RAM and 2 CPUs.

These can be overridden by setting the environment variables `RAILS_DEV_BOX_RAM` and `RAILS_DEV_BOX_CPUS`, respectively. Settings on VM creation don't matter, the environment variables are checked each time the VM boots.

`RAILS_DEV_BOX_RAM` has to be expressed in megabytes, so configure 4096 if you want the VM to have 4 GB of RAM.

## What's In The Box

* Development tools

* Git

* Ruby 2.5

* Bundler

* SQLite3, MySQL, and Postgres

* Databases and users needed to run the Active Record test suite

* System dependencies for `nokogiri`, `sqlite3`, `mysql2`, and `pg`

* Memcached

* Redis

* RabbitMQ

* An ExecJS runtime

## Recommended Workflow

The recommended workflow is

* edit in the host computer and

* test within the virtual machine.

Just clone your Rails fork into the rails-dev-box directory on the host computer:

    host $ ls
    bootstrap.sh MIT-LICENSE README.md Vagrantfile
    host $ git clone git@github.com:<your username>/rails.git

Vagrant mounts that directory as _/vagrant_ within the virtual machine:

    vagrant@rails-dev-box:~$ ls /vagrant
    bootstrap.sh MIT-LICENSE rails README.md Vagrantfile

Install gem dependencies in there:

    vagrant@rails-dev-box:~$ cd /vagrant/rails
    vagrant@rails-dev-box:/vagrant/rails$ bundle

We are ready to go to edit in the host, and test in the virtual machine.

Please have a look at the [Contributing to Ruby on Rails](http://edgeguides.rubyonrails.org/contributing_to_ruby_on_rails.html) guide for tips on how to run test suites, how to generate an application that uses your local checkout of Rails, etc.

This workflow is convenient because in the host computer you normally have your editor of choice fine-tuned, Git configured, and SSH keys in place.

## Virtual Machine Management

When done just log out with `^D` and suspend the virtual machine

    host $ vagrant suspend

then, resume to hack again

    host $ vagrant resume

Run

    host $ vagrant halt

to shutdown the virtual machine, and

    host $ vagrant up

to boot it again.

You can find out the state of a virtual machine anytime by invoking

    host $ vagrant status

Finally, to completely wipe the virtual machine from the disk **destroying all its contents**:

    host $ vagrant destroy # DANGER: all is gone

Please check the [Vagrant documentation](http://docs.vagrantup.com/v2/) for more information on Vagrant.

## Faster Rails test suites

The default mechanism for sharing folders is convenient and works out the box in
all Vagrant versions, but there are a couple of alternatives that are more
performant.

### rsync

Vagrant 1.5 implements a [sharing mechanism based on rsync](https://www.vagrantup.com/blog/feature-preview-vagrant-1-5-rsync.html)
that dramatically improves read/write because files are actually stored in the
guest. Just throw

    config.vm.synced_folder '.', '/vagrant', type: 'rsync'

to the _Vagrantfile_ and either rsync manually with

    vagrant rsync

or run

    vagrant rsync-auto

for automatic syncs. See the post linked above for details.

### NFS

If you're using Mac OS X or Linux you can increase the speed of Rails test suites with Vagrant's NFS synced folders.

With an NFS server installed (already installed on Mac OS X), add the following to the Vagrantfile:

    config.vm.synced_folder '.', '/vagrant', type: 'nfs'
    config.vm.network 'private_network', ip: '192.168.50.4' # ensure this is available

Then

    host $ vagrant up

Please check the Vagrant documentation on [NFS synced folders](http://docs.vagrantup.com/v2/synced-folders/nfs.html) for more information.

## Troubleshooting

On `vagrant up`, it's possible to get this error message:

```
The box 'ubuntu/yakkety64' could not be found or
could not be accessed in the remote catalog. If this is a private
box on HashiCorp's Atlas, please verify you're logged in via
vagrant login. Also, please double-check the name. The expanded
URL and error message are shown below:

URL: ["https://atlas.hashicorp.com/ubuntu/yakkety64"]
Error:
```

And a known work-around (https://github.com/Varying-Vagrant-Vagrants/VVV/issues/354) can be:

    sudo rm /opt/vagrant/embedded/bin/curl

## License

Released under the MIT License, Copyright (c) 2012–<i>ω</i> Xavier Noria.
