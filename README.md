# Phalcon - MS SQL Server (PDO) Adapter

Microsoft SQL Server adapter for [Phalcon Framework](https://phalconphp.com/) using PDO (`pdo_sqlsrv`).

Provides a complete SQL Server integration including a PDO adapter, SQL dialect, result set handler, debug listener, and database logger.

## Requirements

- PHP >= 7.4
- Phalcon >= 4.0 (`ext-phalcon`)
- PHP extension: `pdo_sqlsrv`
- Microsoft SQL Server 2012+

## Installation

```bash
composer require baka/phalcon-sqlsrv
```

## Quick Start

Register the database service in your Phalcon DI container:

```php
$di->set('db', function () use ($config) {
    return new \Phalcon\Db\Adapter\Pdo\Sqlsrv([
        'host'     => $config->database->host,
        'username' => $config->database->username,
        'password' => $config->database->password,
        'dbname'   => $config->database->name,
    ]);
});
```

## Components

### Adapter (`Phalcon\Db\Adapter\Pdo\Sqlsrv`)

The main database adapter. Extends `Phalcon\Db\Adapter\Pdo\AbstractPdo` and provides:

- SQL Server connection via `pdo_sqlsrv` DSN (with `LoginTimeout=3`, `TrustServerCertificate=true`)
- Column metadata retrieval (`describeColumns`)
- Query execution with scrollable cursor support (auto-switches to forward-only for `exec` statements)
- `PDO::ERRMODE_EXCEPTION` and `PDO::ATTR_STRINGIFY_FETCHES` enabled by default

```php
use Phalcon\Db\Adapter\Pdo\Sqlsrv;

$connection = new Sqlsrv([
    'host'     => 'localhost',
    'username' => 'sa',
    'password' => 'YourPassword',
    'dbname'   => 'my_database',
]);

// Query with parameters
$results = $connection->query('SELECT * FROM users WHERE active = ?', [1]);

// Fetch all rows
$rows = $connection->fetchAll('SELECT * FROM users');

// Execute (INSERT/UPDATE/DELETE)
$connection->execute('UPDATE users SET active = 0 WHERE id = ?', [5]);
```

### Dialect (`Phalcon\Db\Dialect\Sqlsrv`)

Generates SQL Server-specific SQL. Handles:

- **Pagination**: `OFFSET ... FETCH NEXT` (instead of MySQL's `LIMIT`)
- **Locking**: `WITH (UPDLOCK)` for exclusive locks, `WITH (NOLOCK)` for shared locks
- **DDL**: `CREATE TABLE`, `ALTER TABLE`, `DROP TABLE`, `CREATE VIEW`, `DROP VIEW`
- **Schema introspection**: `describeColumns`, `describeIndexes`, `describeReferences`, `tableExists`, `viewExists`
- **Index / Foreign Key management**: `addIndex`, `dropIndex`, `addForeignKey`, `dropForeignKey`

### Result (`Phalcon\Db\Result\PdoSqlsrv`)

Wraps PDO statement results. Overrides `numRows()` to use `PDOStatement::rowCount()` for SQL Server compatibility.

### DbListener (`Phalcon\Db\DbListener`)

Debug listener that logs SQL queries to stdout:

```php
use Phalcon\Events\Manager as EventsManager;
use Phalcon\Db\DbListener;

$eventsManager = new EventsManager();
$eventsManager->attach('db', new DbListener());

$connection->setEventsManager($eventsManager);
$connection->debug = true;
```

### Database Logger (`Phalcon\Logger\Adapter\Database`)

Stores log entries in a database table with context (process, user, datetime, IP, browser):

```php
use Phalcon\Logger\Adapter\Database as DbLogger;
use Phalcon\Logger;

$adapter = new DbLogger('main', [
    'db'       => $connection,
    'table'    => 'logs',
    'username' => 'admin',
]);

$logger = new Logger('main', ['main' => $adapter]);
$logger->info('User logged in');
```

## Project Structure

```
Phalcon/
├── Db/
│   ├── Adapter/Pdo/Sqlsrv.php   # PDO Adapter
│   ├── Dialect/Sqlsrv.php        # SQL Dialect
│   ├── Result/PdoSqlsrv.php     # Result Set
│   └── DbListener.php           # Debug Listener
└── Logger/
    └── Adapter/Database.php      # Database Logger
```

## Credits

Forked from [bakaphp/phalcon-sqlsrv](https://github.com/bakaphp/phalcon-sqlsrv).
