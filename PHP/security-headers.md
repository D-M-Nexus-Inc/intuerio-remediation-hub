# Security Headers Remediation (PHP)

HTTP security headers provide an extra layer of security by helping to mitigate attacks like XSS, Clickjacking, and Protocol Downgrades.

## Problem: Vulnerable Configuration

By default, many servers and applications do not send essential security headers, leaving them vulnerable to various client-side attacks.

## Solution: Secure Configuration

### Option 1: Using Laravel Middleware (Recommended)

Laravel makes it easy to set headers globally via middleware.

```php
// app/Http/Middleware/SecureHeaders.php
namespace App\Http\Middleware;

use Closure;

class SecureHeaders
{
    public function handle($request, Closure $next)
    {
        $response = $next($request);

        $response->headers->set('X-Content-Type-Options', 'nosniff');
        $response->headers->set('X-Frame-Options', 'DENY');
        $response->headers->set('X-XSS-Protection', '1; mode=block');
        $response->headers->set('Content-Security-Policy', "default-src 'self'; script-src 'self' https://trusted.cdn.com;");
        $response->headers->set('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
        $response->headers->set('Referrer-Policy', 'no-referrer-when-downgrade');

        return $response;
    }
}
```

### Option 2: Vanilla PHP

```php
<?php
// SECURE: Setting headers manually in PHP
header("X-Content-Type-Options: nosniff");
header("X-Frame-Options: DENY");
header("X-XSS-Protection: 1; mode=block");
header("Content-Security-Policy: default-src 'self';");
header("Strict-Transport-Security: max-age=31536000; includeSubDomains");
header("Referrer-Policy: no-referrer-when-downgrade");
?>
```

### Option 3: .htaccess (Apache)

If you are using Apache, you can set these globally without touching your code.

```apache
<IfModule mod_headers.c>
    Header set X-Content-Type-Options "nosniff"
    Header set X-Frame-Options "DENY"
    Header set Strict-Transport-Security "max-age=31536000; includeSubDomains"
    Header set Referrer-Policy "no-referrer-when-downgrade"
</IfModule>
```

## Why this works

*   **X-Content-Type-Options**: Prevents the browser from "sniffing" the MIME type, which can prevent XSS via uploaded files.
*   **X-Frame-Options**: Prevents your site from being embedded in an `<iframe>`, mitigating Clickjacking.
*   **Strict-Transport-Security (HSTS)**: Forces the browser to only communicate via HTTPS.
*   **Content-Security-Policy (CSP)**: Restricts where scripts, styles, and other resources can be loaded from, significantly reducing XSS impact.

## Best Practices

1. **Test your CSP**: Use tools like [CSP Evaluator](https://csp-evaluator.withgoogle.com/) to ensure your policy isn't too lax.
2. **Use `HttpOnly` and `Secure` cookies**: Always set these flags on your session cookies.
3. **Verify with Security Headers tool**: Use [securityheaders.com](https://securityheaders.com/) to scan your site and get a grade.
