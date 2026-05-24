# CodeIgniter Migration CLI

[![Packagist Version](https://img.shields.io/packagist/v/fishingboy/codeigniter-migration.svg)](https://packagist.org/packages/fishingboy/codeigniter-migration)
[![Downloads](https://img.shields.io/packagist/dt/fishingboy/codeigniter-migration.svg?label=Downloads)](https://packagist.org/packages/fishingboy/codeigniter-migration)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

A small Composer package that adds practical command-line migration commands to CodeIgniter 3 projects.

Use it when you want to run, list, rollback, reset, and refresh CodeIgniter database migrations from `php index.php migration` without building your own migration controller.


## Language

[en-us](README.md) /
[zh-tw](README-zh-tw.md)

## Features

- Composer-installable migration helper for CodeIgniter 3.
- CLI commands for `migrate`, `rollback`, `ls`, `reset`, and `refresh`.
- Shows which migration files have already run and which are pending.
- Supports timestamp-style migration filenames such as `20001010101000_create_sample_tables.php`.
- Runs each pending migration in order and records execution state in the `migrations` table.
- Supports rolling back the latest migration or a specific migration version.
- Includes an upgrade command for converting older migration table schemas.
- Keeps migration classes compatible with CodeIgniter's `up()` and `down()` pattern.

## Why This Package?

CodeIgniter has a built-in migration library, but many legacy CodeIgniter 3 projects still need a simple CLI workflow for day-to-day schema changes.

This package provides that thin layer:

- No framework replacement.
- No new migration DSL.
- No application structure rewrite.
- Just a controller and library wrapper that make migrations easier to run from the command line.

It is best suited for existing CodeIgniter 3 applications that already use `application/migrations` and want a clearer operational workflow.

## Installation

Install the package with Composer:

```shell
composer require fishingboy/codeigniter-migration
```

## Quick Start

### 1. Create the migration controller

Create `application/controllers/Migration.php`:

```php
<?php

use fishingboy\ci_migration\CI_Migration_Controller;

class Migration extends CI_Migration_Controller {
}
```

### 2. Create the migration library wrapper

Create `application/libraries/Migration.php`:

```php
<?php

use fishingboy\ci_migration\CI_Migration_Library;

class CI_Migration extends CI_Migration_Library {
}
```

### 3. Enable CodeIgniter migrations

Modify `application/config/migration.php`:

```php
$config['migration_enabled'] = true;
$config['migration_type'] = 'timestamp';
$config['migration_table'] = 'migrations';
$config['migration_auto_latest'] = false;
$config['migration_version'] = 0;
$config['migration_path'] = APPPATH.'migrations/';
```

If the `migrations` table does not exist, this package creates it automatically when the migration library is loaded. If your project already has an older `migrations` table without the `file` and `run` columns, run `php index.php migration upgrade_migration` once after installation.

### 4. Create the migrations directory

Create this folder if it does not exist:

```text
application/migrations
```

### 5. Run the migration CLI

```shell
php index.php migration
```

Available commands:

```shell
php index.php migration                   # help
php index.php migration migrate           # run pending migrations
php index.php migration rollback          # rollback the latest migration
php index.php migration rollback VERSION  # rollback a specific migration version
php index.php migration ls                # list migration status
php index.php migration reset             # rollback all executed migrations
php index.php migration refresh           # reset, then run all migrations again
php index.php migration upgrade_migration # upgrade the migration table schema
```

## Real World Example

Create `application/migrations/20001010101000_create_sample_tables.php`:

```php
<?php defined('BASEPATH') OR exit('No direct script access allowed');

class Migration_Create_sample_tables extends CI_Migration
{
    public function up()
    {
        $sql = "CREATE TABLE `users` (
                  `id` INT NOT NULL AUTO_INCREMENT,
                  `name` VARCHAR(20) COMMENT 'name',
                  `created_at` DATETIME NOT NULL,
                  `updated_at` DATETIME NOT NULL,
                  PRIMARY KEY (`id`)
              ) COMMENT = 'user';";

        $this->db->query($sql);
    }

    public function down()
    {
        $sql = "DROP TABLE `users`";
        $this->db->query($sql);
    }
}
```

List migration status:

```shell
$ php index.php migration ls

     Version         Status  File
---  --------------  ------  ------------------------------------
  1. 20001010101000    --    application/migrations/20001010101000_create_sample_tables.php
---  --------------  ------  ------------------------------------
     1 Migration not execute.
```

Run pending migrations:

```shell
$ php index.php migration migrate
Migration Run : Migration_Create_sample_tables::up() ............. OK !
```

Rollback the latest migration:

```shell
$ php index.php migration rollback
Migration Run : Migration_Create_sample_tables::down() ............. OK !
```

Rollback a specific migration version:

```shell
$ php index.php migration rollback 20001010101000
Migration Run : Migration_Create_sample_tables::down() ............. OK !
```

## Command Reference

| Command | Description |
| --- | --- |
| `php index.php migration` | Show help. |
| `php index.php migration migrate` | Run all pending migrations in version order. |
| `php index.php migration rollback` | Roll back the latest executed migration. |
| `php index.php migration rollback VERSION` | Roll back a specific migration version. |
| `php index.php migration ls` | List all migration files and execution status. |
| `php index.php migration reset` | Roll back all executed migrations. |
| `php index.php migration refresh` | Run `reset`, then run all migrations again. |
| `php index.php migration upgrade_migration` | Manually add the package-required `file` and `run` columns to an older `migrations` table. |

## Migration File Rules

- Place migration files in `application/migrations`.
- Use timestamp filenames when your CodeIgniter migration config uses timestamp mode.
- The class name must match the migration filename after the version prefix.
- Each migration class should implement both `up()` and `down()`.

Example:

```text
application/migrations/20001010101000_create_sample_tables.php
```

```php
class Migration_Create_sample_tables extends CI_Migration
{
    public function up()
    {
        // Apply schema changes.
    }

    public function down()
    {
        // Revert schema changes.
    }
}
```

## Comparison

| Option | Best for | Trade-off |
| --- | --- | --- |
| CodeIgniter built-in migration library | Basic migration support inside a CodeIgniter app | CLI workflow and status visibility are limited. |
| `fishingboy/codeigniter-migration` | Existing CodeIgniter 3 apps that need simple CLI migration commands | Intentionally focused on CodeIgniter's migration model, not a standalone migration framework. |
| Phinx or Doctrine Migrations | Framework-independent migration workflows | Requires adopting another migration system and conventions. |

## FAQ

### Is this for CodeIgniter 3?

Yes. The package extends CodeIgniter-style `CI_Controller` and migration library behavior, so it is intended for CodeIgniter 3 projects.

### Does this replace CodeIgniter migrations?

No. It wraps and extends the CodeIgniter migration workflow with practical CLI commands and migration status tracking.

### Does it create the `migrations` table?

Yes. The library creates the migration table when it does not exist.

### Does `upgrade_migration` run automatically?

No. `upgrade_migration` is a manual command. It is only needed when your project already has an older `migrations` table that does not include the `file` and `run` columns.

### Can I see which migrations already ran?

Yes. Use:

```shell
php index.php migration ls
```

### Can I roll back only one migration?

Yes. Roll back the latest migration:

```shell
php index.php migration rollback
```

Or roll back a specific version:

```shell
php index.php migration rollback 20001010101000
```

### Can I use raw SQL in migrations?

Yes. Migration classes can use `$this->db->query($sql)` or CodeIgniter database utilities.

## Roadmap

- Improve README examples for different database drivers.
- Add more migration command examples.
- Add automated tests around migration status tracking.
- Document common CodeIgniter configuration variants.

## License

This project is licensed under the [MIT License](LICENSE).
