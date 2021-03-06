# Codeigniter-Migration

## Language

[en-us](README.md) / 
[zh-tw](README-zh-tw.md)

## Installation
```
composer require fishingboy/codeigniter-migration
```

## Usage
1. Create file: `application/controller/Migration.php` 
    ```php
    <?php
    use fishingboy\ci_migration\CI_Migration_Controller;
    class Migration extends CI_Migration_Controller {
    }
    ```
2. Create file: `application/libraries/Migration.php` 
    ```php
    <?php
    use fishingboy\ci_migration\CI_Migration_Library;
    class CI_Migration extends CI_Migration_Library {
    }   
    ```
3. Modify file: `application/config/Migration.php`   
    ```php
    $config['migration_enabled'] = true;
    ```
   
4. Create folder: `application/migrations` 

5. Create migration file :  application/migrations/20001010101000_create_sample_tables.php
    ```php
    <?php defined('BASEPATH') OR exit('No direct script access allowed');
    
    class Migration_Create_sample_tables extends CI_Migration
    {
        public function up()
        {
            $sql = "CREATE TABLE `users` ( 
                      `id` INT NOT NULL AUTO_INCREMENT , 
                      `name` VARCHAR(20) COMMENT 'name', 
                      `created_at` DATETIME NOT NULL , 
                      `updated_at` DATETIME NOT NULL , 
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
    
6. Migration help:  `php index.php migration`
    ```shell
    $ php index.php migration
    
      migration
      
      php index.php migration          -- help 
      php index.php migration migrate  -- execute migrations
      php index.php migration rollback -- rollback to prev migration
      php index.php migration ls       -- check migrations list 

    ```
    
7. List of migrations
   ```shell
   $ php index.php migration ls
   
         Version         Status  File
    ---  --------------  ------  ------------------------------------
      1. 20190815002100    --    application/migrations/20190815002100_create_logs_tables.php 
    ---  --------------  ------  ------------------------------------
         0 Migration not execute.

   ```
   
8. Execute migration
   ```shell
   $ php index.php migration migrate
   Migration Run : Migration_Create_sample_tables::up() ............. OK !
   ```
   
9. Execute migration rollback
   ```shell
   $ php index.php migration rollback
   Migration Run : Migration_Create_sample_tables::down() ............. OK !
   ```


