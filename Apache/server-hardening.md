# Server Hardening Remediation (Apache)

Hardening Apache involves minimizing the information leaked to attackers and restricting access to critical files.

## Problem: Vulnerable Configuration

Default Apache installations often display detailed version info and allow directory indexing.

## Solution: Secure Configuration

### 1. Disable Server Signature & Tokens
This hides the Apache version and OS information from the public.

```apache
# In apache2.conf or httpd.conf
ServerSignature Off
ServerTokens Prod
```

### 2. Disable Directory Listing
Prevent users from seeing a list of files in a directory if `index.html` is missing.

```apache
# In your VirtualHost or <Directory> block
<Directory /var/www/html>
    Options -Indexes
</Directory>
```

### 3. Restrict Access to Sensitive Files
Prevent access to `.env`, `.git`, or other sensitive file types.

```apache
# In apache2.conf or .htaccess
<FilesMatch "^\.">
    Require all denied
</FilesMatch>

<FilesMatch "\.(bak|conf|dist|fla|in[ci]|log|psd|sh|sql|sw[op])$">
    Require all denied
</FilesMatch>
```

### 4. Disable Trace HTTP Request
The `TRACE` method can be used in Cross-Site Tracing (XST) attacks.

```apache
# In apache2.conf or httpd.conf
TraceEnable Off
```

## Why this works

*   **ServerTokens Prod**: Reduces the `Server` header to just `Server: Apache` instead of `Server: Apache/2.4.41 (Ubuntu)`.
*   **Options -Indexes**: Replaces a file list with a `403 Forbidden` error, protecting your internal file structure.

## Best Practices

1. **Disable Unused Modules**: Use `a2dismod` to remove modules you don't need (e.g., `mod_status`, `mod_autoindex`).
2. **Use `.htaccess` Sparingly**: While convenient, `.htaccess` files can cause performance hits. Use Virtual Host configurations if you have access to them.
3. **Run as Non-Root**: Ensure Apache is running as `www-data` or a dedicated low-privilege user.
