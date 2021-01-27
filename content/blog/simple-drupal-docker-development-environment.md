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
<p>Using Docker as part of your development environment is a smart and progressive move. It lets you, and other developers, easily and consistently setup a local development environment anytime without the hassle of going through multiple steps and documentation, and most of all, provides a high degree of <a href="https://12factor.net/dev-prod-parity" rel="noopener noreferrer nofollow">Environment Parity</a>.</p><p>I found a lot of great examples on how to leverage <a href="https://www.drupal.org/docs/develop/local-server-setup/docker-development-environments" rel="noopener noreferrer nofollow">Docker for Drupal development</a>, some of which I've used personally, so much so that I've decided to try and come up with the simplest and most straightforward setup I can. The idea is to reuse and extend existing solutions while having minimal custom code as much as possible. In doing so, I hope to provide a simple starting point for myself, and hopefully others as well, to build future projects on.</p><p>If you prefer to jump into the code, check out an <a href="https://github.com/angheloko/drupal-docker-sample" rel="noopener noreferrer nofollow">example Drupal project</a> I created that uses this approach.</p><h2>Composer template for Drupal projects</h2><p>This setup assumes that the <a href="https://github.com/drupal-composer/drupal-project" rel="noopener noreferrer nofollow">Composer template for Drupal</a> will be used. The template provides a starter kit for managing your site dependencies with Composer. It also comes with sensible default packages, structure, and tools, such as <a href="https://www.drush.org/" rel="noopener noreferrer nofollow">drush</a> and <a href="https://drupalconsole.com/" rel="noopener noreferrer nofollow">Drupal console</a>.</p><h2>Drupal image</h2><p>For this setup, we'll only need one <code>Dockerfile</code>, which will be used to build the main image for our Drupal container.</p><pre><code>FROM drupal:latest

# Install MySQL client so we can use drush from inside the container, among other things.
RUN apt-get update &amp;&amp; apt-get install -y mariadb-client git

# Install Composer
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

# Make a minor adjustment to the DocumentRoot so that it points to where our application codes are.
RUN sed -i 's/DocumentRoot \/var\/www\/html/DocumentRoot \/var\/www\/html\/web/g' /etc/apache2/sites-available/000-default.conf

# Mount point for the source codes.
VOLUME /var/www/html</code></pre><p>As I've mentioned earlier, I aim to reuse existing solutions where possible so where better to start than to use the <a href="https://hub.docker.com/_/drupal/" rel="noopener noreferrer nofollow">official Docker images for Drupal</a>. This image installs the basic Drupal requirements as well as apply some recommended settings, which means less code for us too.</p><p>In summary, this <code>Dockerfile</code> does the following:</p><ul><li><p>Use the latest official Docker image for Drupal.</p></li><li><p>Install <code>mariadb-client</code> so that we can connect to the database using the command line from inside the container. This also allows us to use <code>drush</code> commands properly from inside the container.</p></li><li><p>Finally, since we're using the Composer template, Drupal will be located under the <code>web</code> sub-directory so we'll need to make a minor adjustment to the default <code>DocumentRoot</code> that comes with most Debian-based OS.</p></li></ul><h2>Docker Compose</h2><p>Now that we have our basic <code>Dockerfile</code> to define our Drupal container, we then need to define other services that we'll need for our application, such as a database service, by <em>sourcing</em> other containers.</p><p>If you haven't come across, <a href="https://docs.docker.com/compose/" rel="noopener noreferrer nofollow">Docker Compose</a> basically allows you to piece together multiple containers that work together to form your application. It does so via a YAML file where we define and configure the necessary services that our application needs.</p><p>For our setup, our Compose file, named <code>docker-compose.yml</code>, looks like this:</p><pre><code>version: '3.3'

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
      MYSQL_HOST: db</code></pre><p>From the code, you might have guessed that we have 2 services - one that serves our application(<code>drupal</code>) and a database service (<code>db</code>).</p><h3>The database service</h3><p>In the spirit of reusing existing solutions where possible, we use the latest <a href="https://hub.docker.com/_/mariadb" rel="noopener noreferrer nofollow">official MariaDB Docker image</a> instead of creating a custom image of the database ourselves. Furthermore, the image already allows us to set certain settings necessary for our application, such as the database credentials, without the need to further extend and customize the image.</p><p>Another option worth noting is the <code>volumes</code> option. This lets us persist the database by mounting a path located in the host machine into the container. If the container is destroyed or rebuilt, our database is safe since it resides in the host machine. For this setup, our database files will be located in the <code>data</code> sub-directory of the application root.</p><h3>The Drupal service</h3><p>Instead of using an existing image, like what we did with the database service, we want to use our own <code>Dockerfile</code> to build the Drupal container, which we've defined earlier.</p><p>You may have also noticed that we defined the <code>environment</code> option in a similar way as what we did in the database service. Being able to set our containers' <a href="https://docs.docker.com/compose/environment-variables/" rel="noopener noreferrer nofollow">environment variables</a> is one of the best benefits of using Docker. This allows us to pass the same database credentials we used in our database service into our Drupal service. We can then change the database variable initialization in the <code>settings.php</code> to:</p><pre><code>&lt;?php

$databases['default']['default'] = [
  'database' =&gt; getenv('MYSQL_DATABASE'),
  'username' =&gt; getenv('MYSQL_USER'),
  'password' =&gt; getenv('MYSQL_PASSWORD'),
  'host' =&gt; getenv('MYSQL_HOST'),
  'prefix' =&gt; '',
  'port' =&gt; '3306',
  'namespace' =&gt; 'Drupal\\Core\\Database\\Driver\\mysql',
  'driver' =&gt; 'mysql',
];</code></pre><p>In most Drupal projects, you wouldn't want to commit the <code>settings.php</code> file since this file will contain sensitive information such as your database credentials along with other non-sensitive settings. By using environment variables and stripping away only the sensitive parts and replacing them with environment variables, you can now safely commit this file and be assured that the rest of your team will have the same settings.</p><p>We've used other options in our Composer file such as <code>ports</code>, <code>volumes</code>, and <code>restart</code>. Some may look self-explanatory but you can find out more about them from the <a href="https://docs.docker.com/compose/compose-file" rel="noopener noreferrer nofollow">Compose file reference</a>.</p><h3>Networking</h3><p>You might be curious how can containers communicate with each other? In our case, how can the Drupal service access the database service? According to the docs:</p><blockquote><p>By default Compose sets up a single network for your app. Each container for a service joins the default network and is both reachable by other containers on that network, and discoverable by them at a hostname identical to the container name.</p></blockquote><p>It means that we can use the service names we defined in the Compose file as a hostname. In our <code>drupal</code> service, we pass the hostname <code>db</code> using the environment variable <code>MYSQL_HOST</code>, which is then used in the <code>settings.php</code>.</p><h2>Setting up a local environment</h2><p>With our Docker configurations out of the way, any developer can spin up a local development environment with just 2 steps!</p><h3>1. Run the app</h3><pre><code>$ docker-compose up</code></pre><p>This command will build and launch your application and database containers.</p><h3>2. Install Drupal</h3><p>This step only needs to be done once. Once you've installed Drupal, the database will persist even when the containers are destroyed or rebuilt since the database files are located in the host machine in the <code>data</code> directory.</p><h4>SSH into the Drupal container</h4><p>First you'll need to know the name of the Drupal container. You can do so by running <code>docker ps</code> command. The name of the container will be indicated under the <code>NAMES</code> column. You'll see an output similar to the one below:</p><pre><code>$ docker ps
CONTAINER ID        IMAGE                  COMMAND                  CREATED             STATUS              PORTS                  NAMES
b94ec144233a        drupal-docker_drupal   "docker-php-entrypoi…"   2 hours ago         Up 2 hours          0.0.0.0:8080-&gt;80/tcp   drupal-docker_drupal_1
d384c1b920ab        mariadb:latest         "docker-entrypoint.s…"   6 hours ago         Up 2 hours          3306/tcp               drupal-docker_db_1</code></pre><p>In the example output above, the name of the Drupal container is <code>drupal-docker_drupal_1</code>. So to SSH into the Drupal container, you'll need to run <code>docker exec -it drupal-docker_drupal_1 /bin/bash</code>.</p><p>Once inside the container, you should immediately see yourself in the <code>/var/www/html</code> directory. From here you can either do a fresh install or import a database dump.</p><h4>Install dependencies via Composer</h4><p>First of all, install all dependencies via <code>composer</code> by running:</p><pre><code>composer install</code></pre><h4>Fresh install via Drush</h4><p>To do a fresh install, we can use <code>drush</code> by running:</p><pre><code>./vendor/bin/drush si standard --site-name="My Drupal 8 site"</code></pre><p>Once done, you'll see an output similar to:</p><pre><code>root@b94ec144233a:/var/www/html# ./vendor/bin/drush si standard --site-name="My Drupal 8 site"

 You are about to DROP all tables in your 'drupal' database. Do you want to continue? (yes/no) [yes]:
 &gt; 

 [notice] Starting Drupal installation. This takes a while.
 [success] Installation complete.  User name: admin  User password: kaEzmrBwAV</code></pre><h4>Importing a database dump</h4><p>To import a database dump, you can run:</p><pre><code>./vendor/bin/drush sql-cli &lt; mysql-dump.sql</code></pre><p>Finally, to access your local site, go to http://localhost:8080.</p><h2>Closing</h2><p>Gone are the days when a developer has to go through multiple lines of documentation and steps in order to set up a local development environment. In some cases, this can even take a full day. With the options we have today, such as Docker, setting up a local development environment has never been easier.</p><p>My original aim was a bare minimum setup that can be easily changed to suite the needs of a project and, so far, this is what I can come up with. With this setup, I hope that one can easily setup a local Drupal development environment anytime.</p><p>While I liked working with Docker, another viable option that I've used before and liked is <a href="https://www.drupalvm.com/" rel="noopener noreferrer nofollow">DrupalVM</a>, which uses <a href="https://www.vagrantup.com/" rel="noopener noreferrer nofollow">Vagrant</a>. I plan to write about this approach too so stay tuned!</p>