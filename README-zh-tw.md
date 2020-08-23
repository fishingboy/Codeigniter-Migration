# Codeigniter-Migration

## 安裝

```
composer require fishingboy/codeigniter-migration
```

## 使用方法

1. 建立 application/controller/Migration.php
    ```php
    <?php
    use fishingboy\ci_migration\CI_Migration_Controller;
    class Migration extends CI_Migration_Controller {
    }
    ```
2. 修改檔案: `application/config/Migration.php`   
    ```php
    $config['migration_enabled'] = true;
    ```

3. 建立 application/migrations 資料夾

4. 建立 application/migrations/20001010101000_create_sample_tables.php
    ```php
    <?php defined('BASEPATH') OR exit('No direct script access allowed');
    
    class Migration_Create_sample_tables extends CI_Migration
    {
        public function up()
        {
            $sql = "CREATE TABLE `users` ( 
                      `id` INT NOT NULL AUTO_INCREMENT , 
                      `name` VARCHAR(20) COMMENT '姓名', 
                      `created_at` DATETIME NOT NULL , 
                      `updated_at` DATETIME NOT NULL , 
                      PRIMARY KEY (`id`)
                  ) COMMENT = '記錄';";
            $this->db->query($sql);
        }
    
        public function down()
        {
            $sql = "DROP TABLE `users`";
            $this->db->query($sql);
        }
    }    
    ```
    
5. 進入 command line 專案目錄底下，執行 `php index.php migration`
    ```shell
    # php index.php migration
    
    migration (資料庫遷移)
    
    php index.php migration          -- 看指令
    php index.php migration migrate  -- 執行
    php index.php migration rollback -- 回復到前一個 migration
    php index.php migration ls       -- 看目前 migration 的狀
    ```
    
6. 查看有哪些 migration
   ```shell
   $ php index.php migration ls
   
         Version         Status  File
         --------------  ------  ------------------------------------
      1. 20001010101000    --    application/migrations/20001010101000_create_sample_tables.php 
         --------------  ------  ------------------------------------
         有 1 個 Migration 待執行。
   ```
   
6. 執行 migration
   ```shell
   $ php index.php migration migrate
   Migration Run : Migration_Create_sample_tables::up() ............. OK !
   ```

7. 執行 migration rollback
   ```shell
   $ php index.php migration rollback
   Migration Run : Migration_Create_sample_tables::down() ............. OK !
   ```
