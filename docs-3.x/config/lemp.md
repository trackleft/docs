---
description: Use a generic LEMP stack on Lando for local development; powered by Docker and Docker Compose. Learn how to config php and nginx version, use postgres or mysql or mariadb, composer, xdebug and custom config files, oh and also import and export databases.
---

# LEMP

The LEMP stack is a common infrastructure designed to run PHP applications, it is similar to the more well known [LAMP](https://en.wikipedia.org/wiki/LAMP_%28software_bundle%29) stack but replaces [Apache](http://apache.org/) with [nginx](http://nginx.org) as the web server.

Lando offers a configurable [recipe](./../config/recipes.md) for developing [LEMP](https://drupal.org/) apps.

Note that this recipe is for a generic LEMP stack. Definitely check out Lando's [other recipes](./../config/recipes.md) before you use this as there may be one designed specifically for use with your framework.

[[toc]]

## Getting Started

Before you get started with this recipe we assume that you have:

1. [Installed Lando](./../basics/installation.md) and gotten familiar with [its basics](./../basics/).
2. [Initialized](./../basics/init.md) a [Landofile](./../config/lando.md) for your codebase for use with this recipe.
3. Read about the various [services](./../config/services.md), [tooling](./../config/tooling.md), [events](./../config/events.md) and [routing](./../config/proxy.md) Lando offers.

However, because you are a developer and developers never ever [RTFM](https://en.wikipedia.org/wiki/RTFM), an example of using the LEMP recipe to run the older [CakePHP 2.0](https://cakephp.org/) project is shown below:

Note that CakePHP could be any other [php framework](https://www.dotcominfoway.com/blog/top-5-must-have-php-frameworks-2018) or your own custom php thing.

```bash
# Initialize a LEMP recipe using the latest CakePHP 2.0 version
lando init \
  --source remote \
  --remote-url git://github.com/cakephp/cakephp.git \
  --remote-options="--branch 2.x --depth 1" \
  --recipe lemp \
  --webroot . \
  --name my-first-lemp-app

# Start it up
lando start

# List information about this app.
lando info
```

## Configuration

While Lando [recipes](./../config/recipes.md) set sane defaults so they work out of the box, they are also [configurable](./../config/recipes.md#config).

Here are the configuration options, set to the default values, for this recipe's [Landofile](./../config/lando.md). If you are unsure about where this goes or what this means we *highly recommend* scanning the [recipes documentation](./../config/recipes.md) to get a good handle on how the magicks work.

```yaml
recipe: lemp
config:
  php: '7.3'
  composer_version: '2.0.7'
  webroot: .
  database: mysql:5.7
  xdebug: false
  config:
    server: SEE BELOW
    php: SEE BELOW
    database: SEE BELOW
    vhosts: SEE BELOW
```

Note that if the above config options are not enough, all Lando recipes can be further [extended and overriden](./../config/recipes.md#extending-and-overriding-recipes).

### Choosing a php version

You can set `php` to any version that is available in our [php service](./php.md). However, you should make sure that whatever framework or custom code you write is designed to work with your choice.

The [recipe config](./../config/recipes.md#config) to set the LEMP recipe to use `php` version `5.4` is shown below:

```yaml
recipe: lemp
config:
  php: '5.4'
```

### Choosing a composer version

You can set `composer_version` to any version that is available in our [php service](./php.md#installing-composer).

```yaml
recipe: lemp
config:
  composer_version: '1.10.1'
```

### Choosing a database backend

By default, this recipe will use the default version of our [mysql](./mysql.md) service as the database backend but you can also switch this to use [`mariadb`](./mariadb.md) or ['postgres'](./postgres.md) instead. Note that you can also specify a version *as long as it is a version available for use with lando* for either `mysql`, `mariadb` or `postgres`.

If you are unsure about how to configure the `database`, we *highly recommend* you check out the [mysql](./mysql.md), [mariadb](./mariadb.md)and ['postgres'](./postgres.md) services before you change the default.

#### Using MySQL (default)

```yaml
recipe: lemp
config:
  database: mysql
```

#### Using MariaDB

```yaml
recipe: lemp
config:
  database: mariadb
```

#### Using Postgres

```yaml
recipe: lemp
config:
  database: postgres
```

#### Using a custom version

```yaml
recipe: lemp
config:
  database: postgres:9.6
```

### Using xdebug

This is just a passthrough option to the [xdebug setting](./php.md#toggling-xdebug) that exists on all our [php services](./php.md). The `tl;dr` is `xdebug: true` enables and configures the php xdebug extension and `xdebug: false` disables it.

```yaml
recipe: lemp
config:
  xdebug: true|false
```

However, for more information we recommend you consult the [php service documentation](./php.md).


### Using custom config files

You may need to override our [default LEMP config](https://github.com/lando/lando/tree/master/plugins/lando-recipes/recipes/LEMP) with your own.

If you do this, you must use files that exist inside your application and express them relative to your project root as shown below:

Note that the default files may change based on how you set both `ssl` and `via`. Also note that the `vhosts` and `server` config will be explicitly for `nginx`. We *highly recommend* you check out the [nginx](./nginx.md#configuration) if you plan to use a custom `vhosts` or `server` config.

**A hypothetical project**

Note that you can put your configuration files anywhere inside your application directory. We use a `config` directory but you can call it whatever you want such as `.lando` in the example below:

```bash
./
|-- config
   |-- default.conf
   |-- my-custom.cnf
   |-- nginx.conf
   |-- php.ini
|-- index.php
|-- .lando.yml
```

**Landofile using custom LEMP config**

```yaml
recipe: lemp
config:
  config:
    server: config/nginx.conf
    vhosts: config/default.conf
    php: config/php.ini
    database: config/my-custom.cnf
```

## Connecting to your database

Lando will automatically set up a database with a user and password and also set an environment variable called [`LANDO INFO`](./../guides/lando-info.md) that contains useful information about how your application can access other Lando services.

The default database connection information for a LEMP site is shown below:

Note that the `host` is not `localhost` but `database`.

```yaml
database: lemp
username: lemp
password: lemp
host: database
# for mysql
port: 3306
# for postgres
# port: 5432
```

You can get also get the above information, and more, by using the [`lando info`](./../cli/info.md) command.

## Importing Your Database

Once you've started up your LEMP site, you will need to pull in your database and files before you can really start to dev all the dev. Pulling your files is as easy as downloading an archive and extracting it to the correct location. Importing a database can be done using our helpful `lando db-import` command.

```bash
# Grab your database dump
curl -fsSL -o database.sql.gz "https://url.to.my.db/database.sql.gz"

# Import the database
# NOTE: db-import can handle uncompressed, gzipped or zipped files
# Due to restrictions in how Docker handles file sharing your database
# dump MUST exist somewhere inside of your app directory.
lando db-import database.sql.gz
```

You can learn more about the `db-import` command [over here](./../guides/db-import.md)

## Tooling

By default, each Lando LEMP recipe will also ship with helpful dev utilities.

This means you can use things like `drush`, `composer` and `php` via Lando and avoid mucking up your actual computer trying to manage `php` versions and tooling.

```bash
lando composer          Runs composer commands
lando db-export [file]  Exports database from a service into a file
lando db-import <file>  Imports a dump file into database service
lando mysql|psql        Drops into a MySQL (or psql) shell on a database service
lando php               Runs php commands
```

**Usage examples**

```bash
# Download a dependency with drush
lando composer require phpunit/phpunit --dev

# Run composer tests
lando db-import dump.sql.gz

# Drop into a mysql shell
lando mysql

# Check hte app's installed php extensions
lando php -m
```

You can also run `lando` from inside your app directory for a complete list of commands. This is always advisable as your list of commands may not be 100% the same as above. For example, if you set `database: postgres` you will get `lando psql` instead of `lando mysql`.

<RelatedGuides tag="LEMP"/>
