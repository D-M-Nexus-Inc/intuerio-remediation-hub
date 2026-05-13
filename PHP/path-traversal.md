# Path Traversal Remediation (PHP)

Path Traversal (or Directory Traversal) allows an attacker to read arbitrary files on the server by manipulating file paths used by the application.

## Problem: Vulnerable Code

Using user input directly in file-related functions like `file_get_contents()` or `include()` is extremely dangerous.

```php
<?php
// VULNERABLE: Direct use of user input in file path
$file = $_GET['page'];
// An attacker could pass "../../etc/passwd"
include("pages/" . $file . ".php");
?>
```

## Solution: Secure Code

Sanitize the input and validate it against an allowlist or a safe directory.

### Option 1: Using `basename()` and `realpath()`

```php
<?php
// SECURE: Sanitize and validate the path
$file = $_GET['page'];

// 1. Get only the filename part (removes ../)
$safeFile = basename($file);

$basePath = realpath(__DIR__ . '/pages');
$requestedPath = realpath($basePath . '/' . $safeFile . '.php');

// 2. Ensure the resolved path starts with the intended base directory
if ($requestedPath && strpos($requestedPath, $basePath) === 0) {
    include($requestedPath);
} else {
    die("Invalid file requested.");
}
?>
```

### Option 2: Using an Allowlist (Best for fixed files)

```php
<?php
// SECURE: Use an allowlist
$allowedPages = ['home', 'about', 'contact'];
$page = $_GET['page'];

if (in_array($page, $allowedPages)) {
    include("pages/" . $page . ".php");
} else {
    include("pages/404.php");
}
?>
```

## Why this works

`basename()` strips out directory traversal sequences like `../`. `realpath()` resolves the final path, allowing you to verify that the file actually exists within the intended directory and hasn't escaped to a sensitive area of the filesystem.

## Best Practices

1. **Avoid user input in paths**: If possible, use an ID or a slug that maps to a filename in a database.
2. **Limit File Permissions**: Ensure the web server user only has access to the directories it absolutely needs.
3. **Turn off `allow_url_include`**: Ensure this is `Off` in your `php.ini` to prevent Remote File Inclusion (RFI).
