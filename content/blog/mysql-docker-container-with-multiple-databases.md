---
title: "MySQL Docker Container with Multiple Databases"
subtitle: "How to creating a MySQL container with multiple databases."
lead: "Creating a MySQL container with multiple databases."
description: "Creating a MySQL container with multiple databases."
createdAt: 2020-02-12T03:12:57.993Z
updatedAt: 2020-02-21T05:45:16.174Z
cover: 
  image: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/images%2FDocker%20DBs(1).png?alt=media&token=f41e9d3d-facd-482b-859f-ab65ce6ea5df"
  alt: "MySQL Docker Container with Multiple Databases cover image"
  caption: ""
  thumb: "https://firebasestorage.googleapis.com/v0/b/donlalicon.appspot.com/o/images%2FDocker%20DBs(1)_thumb.png?alt=media&token=a30745a9-ab62-44aa-afed-2b614b52f30f"
tags: 
  - Docker
  - MySQL
---
I was recently _dockerizing_ a Drupal multisite. Each site in this Drupal 8 multisite required a separate database but the official MySQL docker _seem_ to only support one. After much needed reading, I finally found that you can use the directory `/docker-entrypoint-initdb.d` to store any shell or SQL scripts that you want to be executed. It is important to remember that the scripts inside that directory will only get executed once - **when initializing a fresh instance**.

Compose file
------------

The compose file below will execute any script that is located in the host directory mounted to `/docker-entrypoint-initdb.d`:

```yml
version: '3'
services:
  db:
    image: mysql:5
    volumes:
      - ./data:/var/lib/mysql
      - ./docker/db/init:/docker-entrypoint-initdb.d
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
```

In this case, the local directory `/docker/db/init` will be mounted to the `/docker-entrypoint-initdb.d` directory of the container.

Custom environment variables
----------------------------

I consider it best practice to use environment variables where sensible. For instance, the individual sub-site database credentials should be set in the environment variables. For this case, we will add some custom environment variables into the `environment` option.

In the compose file, we set the custom environment variables `DATABASE_USERNAME`, `DATABASE_PASSWORD`, and the database names of each sub-site in the database service. It is then easy to use the same environment variables on the Drupal service.

```yml
version: '3'
services:
  db:
    image: mysql:5
    volumes:
      - ./data:/var/lib/mysql
      - ./docker/db/init:/docker-entrypoint-initdb.d
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root
      DATABASE_USERNAME: drupal
      DATABASE_PASSWORD: password123
      DATABASE_NAME_SITE_A: site_a
      DATABASE_NAME_SITE_B: site_b
  drupal:
    build:
      context: .
      dockerfile: Dockerfile.local
    depends_on:
      - db
    ports:
      - "8080:80"
    volumes:
      - .:/var/www/html
    environment:
      DATABASE_USERNAME: drupal
      DATABASE_PASSWORD: password123
      DATABASE_NAME_SITE_A: site_a
      DATABASE_NAME_SITE_B: site_b
```

Now that we're setting the environment variables in the compose file, we can use it in the scripts inside `docker-entrypoint-initdb.d` easily.

For instance, we can use the environment variables in a shell script that creates the database user as well as the necessary databases for our sub-sites:

```bash
#!/bin/bash

set -eo pipefail

_create_drupal_database() {
  docker_process_sql --database=mysql <<-EOSQL
    CREATE DATABASE \`$1\` CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
    GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER, CREATE TEMPORARY TABLES ON \`$1\`.* TO '$DATABASE_USERNAME'@'%';
EOSQL
}

mysql_note "Creating user ${DATABASE_USERNAME}"
docker_process_sql --database=mysql <<<"CREATE USER '$DATABASE_USERNAME'@'%' IDENTIFIED BY '$DATABASE_PASSWORD';"

mysql_note "Creating Drupal databases"
for DATABASE_NAME in $DATABASE_NAME_SITE_A $DATABASE_NAME_SITE_B; do
  mysql_note "Creating ${DATABASE_NAME}"
  _create_drupal_database $DATABASE_NAME
done
```

Note that the `mysql_note` and `docker_process_sql` are all existing functions. See https://github.com/docker-library/mysql/blob/6952c5d5a9889311157362c528d65dc2e37ff660/5.7/docker-entrypoint.sh.

Re-initializing databases
-------------------------

As mentioned earlier, the scripts will only run **when initializing a fresh instance**. If you need to re-run the scripts, you will need to make sure that the database directory is empty. In the example compose file, the MySQL database directory is stored in `/data`. So you only need to make sure that this directory is empty before rebuilding the container.

While the examples I gave were based on my experience with developing a Drupal multisite, a similar approach can be applied for other applications as well.