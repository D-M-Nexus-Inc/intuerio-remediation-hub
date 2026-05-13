# Security Headers Remediation (JavaScript / Node.js)

Missing security headers can expose your application to Clickjacking, XSS, and MIME-sniffing attacks.

## Problem: Vulnerable Configuration

An Express app without any security middleware will not send protective headers by default.

```javascript
const express = require('express');
const app = express();

// VULNERABLE: No security headers are set
app.get('/', (req, res) => {
    res.send('Hello World');
});
```

## Solution: Secure Configuration

### Option 1: Using `helmet` (Recommended for Express)

`helmet` is a collection of 15 smaller middleware functions that set security-related HTTP headers.

```javascript
const express = require('express');
const helmet = require('helmet');
const app = express();

// SECURE: Automatically sets 11 default security headers
app.use(helmet());

// Customizing CSP (Example)
app.use(
  helmet.contentSecurityPolicy({
    directives: {
      "default-src": ["'self'"],
      "script-src": ["'self'", "trusted-cdn.com"],
      "object-src": ["'none'"],
      "upgrade-insecure-requests": [],
    },
  })
);

app.get('/', (req, res) => {
    res.send('Hello World (Securely)');
});
```

### Option 2: Vanilla Node.js (http module)

```javascript
const http = require('http');

const server = http.createServer((req, res) => {
    // SECURE: Setting headers manually
    res.setHeader('X-Content-Type-Options', 'nosniff');
    res.setHeader('X-Frame-Options', 'DENY');
    res.setHeader('Strict-Transport-Security', 'max-age=31536000; includeSubDomains');
    
    res.end('Hello World');
});
```

## Why this works

Headers like `X-Frame-Options: DENY` tell the browser that the page should never be rendered in a frame, protecting against Clickjacking. `helmet` simplifies this by applying sensible defaults that follow security best practices with a single line of code.

## Best Practices

1. **Keep Helmet Updated**: Vulnerabilities in headers (like CSP bypasses) are found often; keep your dependencies fresh.
2. **Review your CSP**: The default `helmet` CSP is strict. You may need to adjust it if you use external fonts or analytics scripts.
3. **Use HSTS in Production**: Only enable `Strict-Transport-Security` once you have verified your HTTPS setup is working perfectly, as it can "lock out" users if configured incorrectly.
