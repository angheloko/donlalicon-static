---
title: "Simple Drupal Docker Development Environment"
subtitle: "A simple Docker development environment for Drupal"
lead: "Using Docker for your development environment provides ease of setup, consistency among developers, and, most of all, environment parity."
description: "A simple Docker development environment for Drupal"
createdAt: 2020-01-29T06:32:53.155Z
updatedAt: 2020-04-12T05:32:24.395Z
cover: 
  image: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/images%2Fdrupal-docker(2).png?alt=media&token=234816bc-6a13-4051-a359-2eb0c5829644"
  alt: "Simple Drupal Docker Development Environment cover image"
  caption: ""
  thumb: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/images%2Fdrupal-docker(2)_thumb.png?alt=media&token=40f5dd3d-afe0-48a9-8793-165ad1f88d7f"
tags: 
  - Drupal
  - Docker
---
Using Docker as part of your development environment is a smart and progressive move. It lets you, and other developers, easily and consistently setup a local development environment anytime without the hassle of going through multiple steps and documentation, and most of all, provides a high degree of [Environment Parity](https://12factor.net/dev-prod-parity).

I found a lot of great examples on how to leverage [Docker for Drupal development](https://www.drupal.org/docs/develop/local-server-setup/docker-development-environments), some of which I've used personally, so much so that I've decided to try and come up with the simplest and most straightforward setup I can. The idea is to reuse and extend existing solutions while having minimal custom code as much as possible. In doing so, I hope to provide a simple starting point for myself, and hopefully others as well, to build future projects on.

If you prefer to jump into the code, check out an [example Drupal project](https://github.com/angheloko/drupal-docker-sample) I created that uses this approach.

Composer template for Drupal projects
-------------------------------------

This setup assumes that the [Composer template for Drupal](https://github.com/drupal-composer/drupal-project) will be used. The template provides a starter kit for managing your site dependencies with Composer. It also comes with sensible default packages, structure, and tools, such as [drush](https://www.drush.org/) and [Drupal console](https://drupalconsole.com/).

Drupal image
------------

For this setup, we'll only need one `Dockerfile`, which will be used to build the main image for our Drupal container.

```
FROM drupal:latest

# Install MySQL client so we can use drush from inside the container, among other things.
RUN apt-get update && apt-get install -y mariadb-client git

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Make a minor adjustment to the DocumentRoot so that it points to where our application codes are.
RUN sed -i 's/DocumentRoot \/var\/www\/html/DocumentRoot \/var\/www\/html\/web/g' /etc/apache2/sites-available/000-default.conf

# Mount point for the source codes.
VOLUME /var/www/html
```

As I've mentioned earlier, I aim to reuse existing solutions where possible so where better to start than to use the [official Docker images for Drupal](https://hub.docker.com/_/drupal/). This image installs the basic Drupal requirements as well as apply some recommended settings, which means less code for us too.

In summary, this `Dockerfile` does the following:

*   Use the latest official Docker image for Drupal.
    
*   Install `mariadb-client` so that we can connect to the database using the command line from inside the container. This also allows us to use `drush` commands properly from inside the container.
    
*   Finally, since we're using the Composer template, Drupal will be located under the `web` sub-directory so we'll need to make a minor adjustment to the default `DocumentRoot` that comes with most Debian-based OS.
    

Docker Compose
--------------

Now that we have our basic `Dockerfile` to define our Drupal container, we then need to define other services that we'll need for our application, such as a database service, by _sourcing_ other containers.

If you haven't come across, [Docker Compose](https://docs.docker.com/compose/) basically allows you to piece together multiple containers that work together to form your application. It does so via a YAML file where we define and configure the necessary services that our application needs.

For our setup, our Compose file, named `docker-compose.yml`, looks like this:

```
version: '3.3'

services:
  # The database service.
  db:
    image: mariadb:latest
    volumes:
      - ./data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: drupal
      MYSQL_USER: drupal
      MYSQL_PASSWORD: drupal
  # The Drupal service.
  drupal:
    build: .
    depends_on:
      - db
    ports:
      - "8080:80"
    volumes:
      - .:/var/www/html
    restart: always
    environment:
      MYSQL_DATABASE: drupal
      MYSQL_USER: drupal
      MYSQL_PASSWORD: drupal
      MYSQL_HOST: db
```

From the code, you might have guessed that we have 2 services - one that serves our application(`drupal`) and a database service (`db`).

### The database service

In the spirit of reusing existing solutions where possible, we use the latest [official MariaDB Docker image](https://hub.docker.com/_/mariadb) instead of creating a custom image of the database ourselves. Furthermore, the image already allows us to set certain settings necessary for our application, such as the database credentials, without the need to further extend and customize the image.

Another option worth noting is the `volumes` option. This lets us persist the database by mounting a path located in the host machine into the container. If the container is destroyed or rebuilt, our database is safe since it resides in the host machine. For this setup, our database files will be located in the `data` sub-directory of the application root.

### The Drupal service

Instead of using an existing image, like what we did with the database service, we want to use our own `Dockerfile` to build the Drupal container, which we've defined earlier.

You may have also noticed that we defined the `environment` option in a similar way as what we did in the database service. Being able to set our containers' [environment variables](https://docs.docker.com/compose/environment-variables/) is one of the best benefits of using Docker. This allows us to pass the same database credentials we used in our database service into our Drupal service. We can then change the database variable initialization in the `settings.php` to:

```
<?php

$databases['default']['default'] = [
  'database' => getenv('MYSQL_DATABASE'),
  'username' => getenv('MYSQL_USER'),
  'password' => getenv('MYSQL_PASSWORD'),
  'host' => getenv('MYSQL_HOST'),
  'prefix' => '',
  'port' => '3306',
  'namespace' => 'Drupal\\Core\\Database\\Driver\\mysql',
  'driver' => 'mysql',
];
```

In most Drupal projects, you wouldn't want to commit the `settings.php` file since this file will contain sensitive information such as your database credentials along with other non-sensitive settings. By using environment variables and stripping away only the sensitive parts and replacing them with environment variables, you can now safely commit this file and be assured that the rest of your team will have the same settings.

We've used other options in our Composer file such as `ports`, `volumes`, and `restart`. Some may look self-explanatory but you can find out more about them from the [Compose file reference](https://docs.docker.com/compose/compose-file).

### Networking

You might be curious how can containers communicate with each other? In our case, how can the Drupal service access the database service? According to the docs:

> By default Compose sets up a single network for your app. Each container for a service joins the default network and is both reachable by other containers on that network, and discoverable by them at a hostname identical to the container name.

It means that we can use the service names we defined in the Compose file as a hostname. In our `drupal` service, we pass the hostname `db` using the environment variable `MYSQL_HOST`, which is then used in the `settings.php`.

Setting up a local environment
------------------------------

With our Docker configurations out of the way, any developer can spin up a local development environment with just 2 steps!

### 1\. Run the app

```
$ docker-compose up
```

This command will build and launch your application and database containers.

### 2\. Install Drupal

This step only needs to be done once. Once you've installed Drupal, the database will persist even when the containers are destroyed or rebuilt since the database files are located in the host machine in the `data` directory.

#### SSH into the Drupal container

First you'll need to know the name of the Drupal container. You can do so by running `docker ps` command. The name of the container will be indicated under the `NAMES` column. You'll see an output similar to the one below:

```
$ docker ps
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS                  NAMES
b94ec144233a        drupal-docker_drupal   "docker-php-entrypoi…"   2 hours ago         Up 2 hours          0.0.0.0:8080->80/tcp   drupal-docker_drupal_1
d384c1b920ab        mariadb:latest         "docker-entrypoint.s…"   6 hours ago         Up 2 hours          3306/tcp               drupal-docker_db_1
```

In the example output above, the name of the Drupal container is `drupal-docker_drupal_1`. So to SSH into the Drupal container, you'll need to run `docker exec -it drupal-docker_drupal_1 /bin/bash`.

Once inside the container, you should immediately see yourself in the `/var/www/html` directory. From here you can either do a fresh install or import a database dump.

#### Install dependencies via Composer

First of all, install all dependencies via `composer` by running:

```
composer install
```

#### Fresh install via Drush

To do a fresh install, we can use `drush` by running:

```
./vendor/bin/drush si standard --site-name="My Drupal 8 site"
```

Once done, you'll see an output similar to:

```
root@b94ec144233a:/var/www/html# ./vendor/bin/drush si standard --site-name="My Drupal 8 site"

 You are about to DROP all tables in your 'drupal' database. Do you want to continue? (yes/no) [yes]:
 > 

 [notice] Starting Drupal installation. This takes a while.
 [success] Installation complete.  User name: admin  User password: kaEzmrBwAV
```

#### Importing a database dump

To import a database dump, you can run:

```
./vendor/bin/drush sql-cli < mysql-dump.sql
```

Finally, to access your local site, go to http://localhost:8080.

Closing
-------

Gone are the days when a developer has to go through multiple lines of documentation and steps in order to set up a local development environment. In some cases, this can even take a full day. With the options we have today, such as Docker, setting up a local development environment has never been easier.

My original aim was a bare minimum setup that can be easily changed to suite the needs of a project and, so far, this is what I can come up with. With this setup, I hope that one can easily setup a local Drupal development environment anytime.

While I liked working with Docker, another viable option that I've used before and liked is [DrupalVM](https://www.drupalvm.com/), which uses [Vagrant](https://www.vagrantup.com/). I plan to write about this approach too so stay tuned!