# CodeIgniter Migration CLI

[![Packagist Version](https://img.shields.io/packagist/v/fishingboy/codeigniter-migration.svg)](https://packagist.org/packages/fishingboy/codeigniter-migration)
[![Downloads](https://img.shields.io/packagist/dt/fishingboy/codeigniter-migration.svg?label=Downloads)](https://packagist.org/packages/fishingboy/codeigniter-migration)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

一個小型 Composer 套件，為 CodeIgniter 3 專案補上實用的命令列 migration 操作。

當你想用 `php index.php migration` 執行、查看、rollback、reset 或 refresh CodeIgniter database migrations，又不想自己寫 migration controller 時，可以使用這個套件。


## 語言

[en-us](README.md) /
[zh-tw](README-zh-tw.md)

## 功能特色

- 可透過 Composer 安裝的 CodeIgniter 3 migration helper。
- 提供 `migrate`、`rollback`、`ls`、`reset`、`refresh` 等 CLI 指令。
- 可查看哪些 migration 已執行、哪些尚未執行。
- 支援 timestamp migration 檔名，例如 `20001010101000_create_sample_tables.php`。
- 依照版本順序執行尚未完成的 migration，並記錄執行狀態到 `migrations` 資料表。
- 支援 rollback 最新一筆 migration，也支援 rollback 指定版本。
- 提供 `upgrade_migration` 指令，用於轉換舊版 migration table schema。
- 維持 CodeIgniter 原本的 `up()` / `down()` migration 寫法。

## 為什麼使用這個套件？

CodeIgniter 本身有 migration library，但許多既有 CodeIgniter 3 專案仍需要一個簡單、清楚的 CLI workflow 來處理日常 schema 變更。

這個套件提供的是很薄的一層封裝：

- 不取代框架。
- 不引入新的 migration DSL。
- 不要求重整既有專案架構。
- 只補上 controller 與 library wrapper，讓 migration 可以更容易從命令列操作。

它適合已經使用 `application/migrations`，並希望 migration 執行流程更清楚的 CodeIgniter 3 專案。

## 安裝

使用 Composer 安裝：

```shell
composer require fishingboy/codeigniter-migration
```

## 快速開始

### 1. 建立 migration controller

建立 `application/controllers/Migration.php`：

```php
<?php

use fishingboy\ci_migration\CI_Migration_Controller;

class Migration extends CI_Migration_Controller {
}
```

### 2. 建立 migration library wrapper

建立 `application/libraries/Migration.php`：

```php
<?php

use fishingboy\ci_migration\CI_Migration_Library;

class CI_Migration extends CI_Migration_Library {
}
```

### 3. 啟用 CodeIgniter migrations

修改 `application/config/migration.php`：

```php
$config['migration_enabled'] = true;
$config['migration_type'] = 'timestamp';
$config['migration_table'] = 'migrations';
$config['migration_auto_latest'] = false;
$config['migration_version'] = 0;
$config['migration_path'] = APPPATH.'migrations/';
```

如果 `migrations` 資料表不存在，套件會在 migration library 載入時自動建立新版 schema。如果你的專案已經有舊版 `migrations` 資料表，而且缺少 `file` 與 `run` 欄位，安裝後需要手動執行一次 `php index.php migration upgrade_migration`。

### 4. 建立 migrations 目錄

如果目錄不存在，請建立：

```text
application/migrations
```

### 5. 執行 migration CLI

```shell
php index.php migration
```

可用指令：

```shell
php index.php migration                   # 顯示說明
php index.php migration migrate           # 執行尚未完成的 migrations
php index.php migration rollback          # rollback 最新一筆 migration
php index.php migration rollback VERSION  # rollback 指定版本
php index.php migration ls                # 查看 migration 狀態
php index.php migration reset             # rollback 所有已執行 migrations
php index.php migration refresh           # reset 後重新執行所有 migrations
php index.php migration upgrade_migration # 升級 migration table schema
```

## 實務範例

建立 `application/migrations/20001010101000_create_sample_tables.php`：

```php
<?php defined('BASEPATH') OR exit('No direct script access allowed');

class Migration_Create_sample_tables extends CI_Migration
{
    public function up()
    {
        $sql = "CREATE TABLE `users` (
                  `id` INT NOT NULL AUTO_INCREMENT,
                  `name` VARCHAR(20) COMMENT '姓名',
                  `created_at` DATETIME NOT NULL,
                  `updated_at` DATETIME NOT NULL,
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

查看 migration 狀態：

```shell
$ php index.php migration ls

     Version         Status  File
---  --------------  ------  ------------------------------------
  1. 20001010101000    --    application/migrations/20001010101000_create_sample_tables.php
---  --------------  ------  ------------------------------------
     1 Migration not execute.
```

執行尚未完成的 migrations：

```shell
$ php index.php migration migrate
Migration Run : Migration_Create_sample_tables::up() ............. OK !
```

Rollback 最新一筆 migration：

```shell
$ php index.php migration rollback
Migration Run : Migration_Create_sample_tables::down() ............. OK !
```

Rollback 指定 migration 版本：

```shell
$ php index.php migration rollback 20001010101000
Migration Run : Migration_Create_sample_tables::down() ............. OK !
```

## 指令參考

| 指令 | 說明 |
| --- | --- |
| `php index.php migration` | 顯示說明。 |
| `php index.php migration migrate` | 依版本順序執行所有尚未完成的 migrations。 |
| `php index.php migration rollback` | Rollback 最新一筆已執行的 migration。 |
| `php index.php migration rollback VERSION` | Rollback 指定 migration 版本。 |
| `php index.php migration ls` | 列出 migration 檔案與執行狀態。 |
| `php index.php migration reset` | Rollback 所有已執行 migrations。 |
| `php index.php migration refresh` | 執行 `reset`，再重新執行所有 migrations。 |
| `php index.php migration upgrade_migration` | 手動為舊版 `migrations` 資料表補上套件需要的 `file` 與 `run` 欄位。 |

## Migration 檔案規則

- Migration 檔案放在 `application/migrations`。
- 如果 CodeIgniter migration 設定使用 timestamp mode，請使用 timestamp 檔名。
- Class name 必須對應 migration 檔名移除版本前綴後的名稱。
- 每個 migration class 建議都實作 `up()` 與 `down()`。

範例：

```text
application/migrations/20001010101000_create_sample_tables.php
```

```php
class Migration_Create_sample_tables extends CI_Migration
{
    public function up()
    {
        // 套用 schema 變更。
    }

    public function down()
    {
        // 回復 schema 變更。
    }
}
```

## 比較

| 選項 | 適合情境 | 取捨 |
| --- | --- | --- |
| CodeIgniter 內建 migration library | 在 CodeIgniter app 內做基本 migration | CLI workflow 與狀態查看能力較有限。 |
| `fishingboy/codeigniter-migration` | 既有 CodeIgniter 3 專案需要簡單 CLI migration 指令 | 專注於 CodeIgniter migration 模型，不是獨立 migration framework。 |
| Phinx 或 Doctrine Migrations | 需要 framework-independent migration workflow | 需要導入另一套 migration 系統與慣例。 |

## FAQ

### 這個套件是給 CodeIgniter 3 使用的嗎？

是。此套件延伸 CodeIgniter 風格的 `CI_Controller` 與 migration library 行為，主要目標是 CodeIgniter 3 專案。

### 它會取代 CodeIgniter migrations 嗎？

不會。它是在 CodeIgniter migration workflow 上補上實用 CLI 指令與 migration 狀態追蹤。

### 它會建立 `migrations` 資料表嗎？

會。當 migration table 不存在時，library 會自動建立。

### `upgrade_migration` 會自動執行嗎？

不會。`upgrade_migration` 是手動指令，只有在專案已經存在舊版 `migrations` 資料表，而且缺少 `file` 與 `run` 欄位時才需要執行。

### 可以查看哪些 migrations 已經執行嗎？

可以，執行：

```shell
php index.php migration ls
```

### 可以只 rollback 一個 migration 嗎？

可以。Rollback 最新一筆 migration：

```shell
php index.php migration rollback
```

或 rollback 指定版本：

```shell
php index.php migration rollback 20001010101000
```

### Migration 裡可以寫 raw SQL 嗎？

可以。Migration class 可以使用 `$this->db->query($sql)` 或 CodeIgniter database utilities。

## Roadmap

- 補充不同 database driver 的 README 範例。
- 增加更多 migration 指令範例。
- 為 migration 狀態追蹤補上自動化測試。
- 補充常見 CodeIgniter migration 設定情境。

## 授權

本專案採用 [MIT License](LICENSE) 授權。
