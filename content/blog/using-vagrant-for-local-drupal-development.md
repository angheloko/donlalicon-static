---
title: "Using Vagrant for local Drupal development"
subtitle: "How to easily set up a Drupal development environment with Drupal VM"
lead: "How to easily set up a Drupal development environment with Drupal VM"
description: ""
createdAt: 2020-03-05T07:38:18.105Z
updatedAt: 2020-03-27T01:56:22.053Z
cover: 
  image: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/images%2Fkelli-mcclintock-GopRYASfsOc-unsplash.jpg?alt=media&token=8c0cf817-ea11-444b-9b60-246387c7db22"
  alt: "Using Vagrant for local Drupal development cover image"
  caption: "Photo by Kelli McClintock on Unsplash"
  thumb: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/images%2Fkelli-mcclintock-GopRYASfsOc-unsplash_thumb.jpg?alt=media&token=74933fc4-6a13-4472-8364-e7958e7f28b9"
tags: 
  - Drupal
  - Vagrant
  - DrupalVM
---
In a previous article, I wrote about [developing Drupal with Docker](https://donlalicon.dev/blog/simple-drupal-docker-development-environment). In this article, we'll look into another popular option for your Drupal development environment.

Drupal VM
---------

[Drupal VM](https://www.drupalvm.com/) is a tool for setting up a development environment in a virtual machine using [Vagrant](https://www.vagrantup.com/intro/index.html). In addition to simplifying the setup process, it also automatically installs and sets up common tools and libraries found in most Drupal-based projects such as [Drush](http://www.drush.org/), [Drupal Console](https://drupalconsole.com/), and [Node.js](https://nodejs.org/en/).

Getting started
---------------

The only prerequisite is to install [Vagrant](https://www.vagrantup.com/downloads.html), [VirtualBox](https://www.virtualbox.org/wiki/Downloads), and [Composer](https://getcomposer.org/). It is also assumed that you have a Composer-based Drupal project or is using Composer to manage your dependencies.

### Install as a Composer dependecy

There are [multiple ways](https://github.com/geerlingguy/drupal-vm#quick-start-guide) on how to use Drupal VM in your project. My favorite is installing Drupal VM as a Composer dependency.

```
composer require --dev geerlingguy/drupal-vm
```

### Create a configuration file

You can configure how your VM is setup via a configuration file. The configuration file must be named `config.yml` although you can save it anywhere you want.

Below is an example of a `config.yml` file. The full configuration options that you can override can be found in [default.config.yml](https://github.com/geerlingguy/drupal-vm/blob/master/default.config.yml). It's recommended to only include the values you want to override.

```
# Update the hostname to the local development environment hostname.
vagrant_hostname: local.headless-lightning.com
vagrant_machine_name: headless-lightning

# Set the IP address so it doesn't conflict with other Drupal VM instances.
vagrant_ip: 192.168.72.228

# Use Ubuntu 16.04 LTS
vagrant_box: geerlingguy/ubuntu1604

# Set drupal_site_name to the project's human-readable name.
drupal_site_name: "Headless Lightning"

# The installation profile.
drupal_install_profile: headless_lightning

# Provide the path to the project root to Vagrant.
vagrant_synced_folders:
  - local_path: .
    destination: /var/www/headless-lightning
    type: nfs

# Set to false since we're using Composer's create-project as a site deployment strategy.
drupal_build_composer_project: false
drupal_composer_path: false
drupal_composer_install_dir: "/var/www/headless-lightning"
drupal_core_path: "/var/www/headless-lightning/docroot"

# MySQL settings.
drupal_db_user: drupal
drupal_db_password: drupal
drupal_db_name: drupal
drupal_db_host: localhost
```

### Create a delegating Vagrantfile

A delegating `Vagrantfile` catches all `vagrant` commands that you run in your project directory and passes them to Drupal VM's own `Vagrantfile`.

Below is how a delegating `Vagrantfile` would look like:

```
# The absolute path to the root directory of the project. Both Drupal VM and
# the config file need to be contained within this path.
ENV['DRUPALVM_PROJECT_ROOT'] = "#{__dir__}"

# The relative path from the project root to the config directory where you
# placed your config.yml file.
ENV['DRUPALVM_CONFIG_DIR'] = "box"

# The relative path from the project root to the directory where Drupal VM is located.
ENV['DRUPALVM_DIR'] = "vendor/geerlingguy/drupal-vm"

# Load the real Vagrantfile
load "#{__dir__}/#{ENV['DRUPALVM_DIR']}/Vagrantfile"
```

### Build your VM

Finally, in your project's root directory, run:

```
vagrant up
```

Note that this process will take a few minutes. At the end you should see something similar to this:

```
==> headless-lightning: Machine 'headless-lightning' has a post `vagrant up` message. This is a message
==> headless-lightning: from the creator of the Vagrantfile, and not from Vagrant itself:
==> headless-lightning: 
==> headless-lightning: Your Drupal VM Vagrant box is ready to use!
==> headless-lightning: * Visit the dashboard for an overview of your site: http://dashboard.local.headless-lightning.com (or http://192.168.72.228)
==> headless-lightning: * You can SSH into your machine with `vagrant ssh`.
==> headless-lightning: * Find out more in the Drupal VM documentation at http://docs.drupalvm.com
```

You can now access your site!

Final words
-----------

I intentionally didn't compare using Drupal VM and Docker. Honestly, I liked both and I usually include both options in my projects.