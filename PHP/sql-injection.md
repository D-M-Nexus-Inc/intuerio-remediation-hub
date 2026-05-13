# SQL Injection Remediation (PHP)

SQL Injection (SQLi) occurs when an application includes untrusted data in a database query in an unsafe way. An attacker can use this to view data they are not normally able to retrieve, or even modify/delete data.

## Problem: Vulnerable Code

Using raw string concatenation to build queries is the most common cause of SQLi.

```php
<?php
// VULNERABLE: Direct concatenation of user input
$userId = $_GET['id'];
$query = "SELECT username, email FROM users WHERE id = " . $userId;

$result = $conn->query($query);
?>
```

## Solution: Secure Code

Always use **Prepared Statements** (also known as Parameterized Queries). This ensures that the database treats the user input as data, not as executable code.

### Option 1: Using PDO (Recommended)

```php
<?php
// SECURE: Using PDO with prepared statements
$userId = $_GET['id'];

$stmt = $pdo->prepare('SELECT username, email FROM users WHERE id = :id');
$stmt->execute(['id' => $userId]);
$user = $stmt->fetch();
?>
```

### Option 2: Using Laravel Eloquent (Best for Laravel)

Laravel's Eloquent ORM uses prepared statements automatically.

```php
<?php
// SECURE: Using Eloquent ORM
$userId = $request->input('id');
$user = User::find($userId);
?>
```

## Why this works

Prepared statements separate the SQL query logic from the data. The database compiles the SQL query structure first, and then binds the parameters. This makes it impossible for an attacker to alter the query's logic via input.

## Best Practices

1. **Never trust user input**: Treat everything from `$_GET`, `$_POST`, `$_REQUEST`, and even database values as untrusted.
2. **Use an ORM**: Frameworks like Laravel (Eloquent) or Symfony (Doctrine) handle most of this for you.
3. **Least Privilege**: Ensure the database user used by the application has only the permissions it needs.
