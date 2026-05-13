# Security Headers Remediation (Apache)

Setting security headers in Apache protects your users and application from common web-based attacks.

## Problem: Vulnerable Configuration

By default, Apache does not send headers like `X-Content-Type-Options` or `Strict-Transport-Security`.

## Solution: Secure Configuration

Use the `mod_headers` module. You can add these to your main config file (`httpd.conf` or `apache2.conf`), your virtual host file, or a local `.htaccess` file.

### Option 1: .htaccess (Per-directory)

```apache
<IfModule mod_headers.c>
    # SECURE: Essential Security Headers
    Header set X-Frame-Options "DENY"
    Header set X-Content-Type-Options "nosniff"
    Header set X-XSS-Protection "1; mode=block"
    Header set Referrer-Policy "no-referrer-when-downgrade"
    Header set Content-Security-Policy "default-src 'self';"
    
    # HSTS (Strict-Transport-Security)
    Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
</IfModule>
```

### Option 2: Virtual Host (Recommended)

```apache
<VirtualHost *:443>
    ServerName example.com
    
    # SECURE: Set headers globally for this host
    Header always set X-Frame-Options "DENY"
    Header always set X-Content-Type-Options "nosniff"
    
    # ... rest of your config
</VirtualHost>
```

## Why this works

The `Header` directive tells Apache to inject these strings into the HTTP response. Using `always set` ensures that the headers are included even on error responses, which is a key security requirement.

## Best Practices

1. **Enable mod_headers**: Ensure the module is enabled by running `sudo a2enmod headers`.
2. **Check Syntax**: Always run `apachectl configtest` before restarting.
3. **Use 'always'**: For headers like HSTS, using `always` is critical to ensure they are sent on 401, 403, and 500 error pages.
