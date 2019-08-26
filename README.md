# Codeigniter-Migration

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
2. Create folder: `application/migrations` 

3. Create migration file :  application/migrations/20001010101000_create_sample_tables.php
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
    
4. Migration help:  `php index.php migration`
    ```shell
    $ php index.php migration
    
      migration
      
      php index.php migration          -- help 
      php index.php migration migrate  -- execute migrations
      php index.php migration rollback -- rollback to prev migration
      php index.php migration ls       -- check migrations list 

    ```
    
5. List of migrations
   ```shell
   $ php index.php migration ls
   
         Version         Status  File
    ---  --------------  ------  ------------------------------------
      1. 20190815002100    --    application/migrations/20190815002100_create_logs_tables.php 
    ---  --------------  ------  ------------------------------------
         0 Migration not execute.

   ```
   
6. Execute migration
   ```shell
   $ php index.php migration migrate
   Migration Run : Migration_Create_sample_tables::up() ............. OK !
   ```
   
7. Execute migration rollback
   ```shell
   $ php index.php migration rollback
   Migration Run : Migration_Create_sample_tables::down() ............. OK !
   ```
