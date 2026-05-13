# Command Injection Remediation (PHP)

Command injection occurs when an application passes unsafe user-supplied data to a system shell. In PHP, this often happens with `exec()`, `system()`, `shell_exec()`, or `passthru()`.

## Vulnerable Example
```php
// BAD: Direct concatenation into a shell command
$target = $_GET['ip'];
$output = shell_exec("ping -c 4 " . $target);
```

## Secure Implementation

### Using `escapeshellarg()`
Wrap every user-supplied argument in `escapeshellarg()`.
```php
// GOOD: Sanitize arguments
$target = $_GET['ip'];
$safe_target = escapeshellarg($target);
$output = shell_exec("ping -c 4 " . $safe_target);
```

### Avoiding the Shell (Preferred)
Use built-in PHP functions instead of calling external system commands.
```php
// GOOD: Use native PHP functions
// If you need to check if a host is up, use sockets or a library instead of ping
$host = 'example.com';
$port = 80;
$check = @fsockopen($host, $port, $errno, $errstr, 5);
```

## Best Practices
1. **Disable Dangerous Functions**: In `php.ini`, add `exec, system, shell_exec, passthru` to `disable_functions`.
2. **Use an Allow-list**: If you must execute commands, only allow a specific set of commands and arguments.
3. **Environment**: Ensure the web server user has limited shell permissions.
