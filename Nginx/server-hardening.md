# Server Hardening Remediation (Nginx)

Hardening Nginx involves disabling unnecessary features and restricting access to sensitive files to reduce the server's attack surface.

## Problem: Vulnerable Configuration

Default Nginx settings often reveal the server version and allow access to sensitive directories or files.

## Solution: Secure Configuration

### 1. Disable Server Tokens (Information Leakage)
This prevents Nginx from displaying its version number in error pages and the `Server` header.

```nginx
# In /etc/nginx/nginx.conf
http {
    server_tokens off;
}
```

### 2. Disable Directory Listing
This prevents attackers from browsing your file system if an `index.html` file is missing.

```nginx
server {
    location / {
        autoindex off;
    }
}
```

### 3. Restrict Access to Sensitive Files
Block access to hidden files (like `.env`, `.git`, `.htaccess`).

```nginx
server {
    # Block all files starting with a dot
    location ~ /\.(?!well-known) {
        deny all;
        access_log off;
        log_not_found off;
    }

    # Specifically block common sensitive files
    location ~* \.(bak|conf|dist|fla|in[ci]|log|psd|sh|sql|sw[op])$ {
        deny all;
    }
}
```

### 4. Limit Request Body Size (DoS Protection)
Prevent attackers from overwhelming your server with large file uploads.

```nginx
http {
    client_max_body_size 10M;
}
```

## Why this works

*   **server_tokens off**: Hiding the version number makes it harder for attackers to find specific exploits for your version of Nginx.
*   **deny all**: Explicitly blocking access to sensitive files ensures that even if you accidentally upload a backup or config file, it won't be accessible to the public.

## Best Practices

1. **Least Privilege**: Run Nginx as a non-privileged user (usually `www-data`).
2. **Logging**: Keep access and error logs enabled but monitor them for unusual patterns (like 403 spikes).
3. **Keep Updated**: Regularly update Nginx using your package manager (`apt update && apt upgrade nginx`).
