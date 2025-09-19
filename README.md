# ddev-switch-db
Switch between active db's inside ddev

## First time setup
Run this command from inside this repo:

```
./ddev-switch-db-setup /path/to/your/ddev/project
```
This will move over all files needed to your ddev project so you don't have to do the manual steps below.

Run this command from inside your ddev project to switch the active db:
```
ddev switch-db [schema-name]
```

To import a db dump into the active schema, use:
```
ddev import-active --src=path/to/dump.sql.gz
```

## Manual setup
Copy `.ddev/host/switch-db` in your project and make it executable.
Copy `.ddev/host/import-active` in your project and make it executable.

Add this to your `.ddev/config.yaml`:
```yaml
web_environment:
    - ACTIVE_DB_SCHEMA=tst
```

Add this to your `settings.local.php`:
```php

$active = getenv('ACTIVE_DB_SCHEMA') ?: 'prd';

// Standard DDEV creds.
$databases['default']['default'] = [
  'database'  => $active,
  'username'  => 'db',
  'password'  => 'db',
  'host'      => 'db',
  'port'      => '3306',
  'driver'    => 'mysql',
  'prefix'    => '',
  'collation' => 'utf8mb4_general_ci',
];

try {
  $pdo = new PDO('mysql:host=db;port=3306', 'db', 'db');
  $stmt = $pdo->query("SHOW DATABASES LIKE " . $pdo->quote($active));
  if ($stmt->rowCount() === 0) {
    \Drupal::messenger()->addWarning(t('ACTIVE_DB_SCHEMA "@db" not found.', ['@db' => $active]));
  }
} catch (\Throwable $e) {
  // ignore in local bootstrap if MySQL not reachable yet
}
```