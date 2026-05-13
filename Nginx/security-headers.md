# Security Headers Remediation (Nginx)

Configuring security headers at the web server level ensures that they are applied consistently across all applications hosted on that server.

## Problem: Vulnerable Configuration

By default, Nginx does not include headers like `X-Frame-Options` or `Content-Security-Policy`.

## Solution: Secure Configuration

Add the `add_header` directive to your `server` or `location` block in your Nginx configuration (usually in `/etc/nginx/sites-available/default` or `/etc/nginx/nginx.conf`).

```nginx
server {
    listen 443 ssl;
    server_name example.com;

    # SECURE: Essential Security Headers
    add_header X-Frame-Options "DENY" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self'; http: https: data: blob: 'unsafe-inline'" always;
    
    # HSTS (Strict-Transport-Security) - Forces HTTPS for 1 year
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains; preload" always;

    # ... rest of your config
}
```

## Why this works

Setting headers at the Nginx level is often more performant than setting them in the application code. The `always` parameter ensures that the headers are sent even for error pages (like 404 or 500), which is a common security requirement.

## Best Practices

1. **Verify Config**: Always run `nginx -t` after making changes to check for syntax errors.
2. **Reload Gracefully**: Use `systemctl reload nginx` instead of `restart` to apply changes without dropping active connections.
3. **Use 'always'**: Always include the `always` parameter to ensure headers are sent on all response codes.
