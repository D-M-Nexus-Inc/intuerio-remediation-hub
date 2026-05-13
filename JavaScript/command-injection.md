# Command Injection Remediation (JavaScript / Node.js)

Command Injection occurs when an application passes untrusted data to a system shell. This can allow an attacker to execute arbitrary commands on the host operating system.

## Problem: Vulnerable Code

Using `child_process.exec()` with concatenated user input.

```javascript
const { exec } = require('child_process');

// VULNERABLE: Input is passed directly to the shell
const domain = req.query.domain;
exec(`nslookup ${domain}`, (error, stdout, stderr) => {
    res.send(stdout);
});
```

## Solution: Secure Code

Avoid using a shell. Use `execFile()` or `spawn()` and pass arguments as an array.

### Option 1: Using `execFile()` (Recommended)

```javascript
const { execFile } = require('child_process');

// SECURE: Arguments are passed as an array, bypassing the shell
const domain = req.query.domain;
execFile('nslookup', [domain], (error, stdout, stderr) => {
    res.send(stdout);
});
```

### Option 2: Using `spawn()`

```javascript
const { spawn } = require('child_process');

// SECURE: Similar to execFile, spawn does not use a shell by default
const domain = req.query.domain;
const ls = spawn('nslookup', [domain]);

ls.stdout.on('data', (data) => {
    res.send(data);
});
```

## Why this works

Functions like `execFile` and `spawn` (without the `shell: true` option) do not invoke a shell (`/bin/sh` or `cmd.exe`). Instead, they execute the binary directly and pass the arguments as a list. This prevents characters like `;`, `&`, or `|` from being interpreted as command separators.

## Best Practices

1. **Avoid System Commands**: If a Node.js library exists to do the task (e.g., the `dns` module for nslookup), use it instead of calling a shell command.
2. **Input Validation**: Use a strict regex to validate input (e.g., only allow alphanumeric characters and dots for a domain).
3. **Principle of Least Privilege**: Run the Node.js process with a user account that has minimal permissions.
