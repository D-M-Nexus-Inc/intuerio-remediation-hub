# Security Headers Remediation (Python)

Implementing security headers in Python web frameworks like Flask and Django helps protect against common web attacks.

## Problem: Vulnerable Configuration

Standard Flask or Django setups might lack essential security headers if not explicitly configured.

## Solution: Secure Configuration

### Option 1: Using `Flask-Talisman` (For Flask)

`Flask-Talisman` is the Python equivalent of `helmet` for Node.js.

```python
from flask import Flask
from flask_talisman import Talisman

app = Flask(__name__)

# SECURE: Automatically sets HSTS, CSP, X-Frame-Options, etc.
# Note: 'force_https=True' should be used in production
Talisman(app, content_security_policy=None) 

@app.route('/')
def hello():
    return "Hello Secure World"
```

### Option 2: Using Django Security Middleware

Django has robust security headers built into its middleware system.

```python
# settings.py

MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware', # Must be at the top
    # ...
]

# SECURE: Configure security settings
SECURE_BROWSER_XSS_FILTER = True
SECURE_CONTENT_TYPE_NOSNIFF = True
X_FRAME_OPTIONS = 'DENY'
SECURE_HSTS_SECONDS = 31536000
SECURE_HSTS_INCLUDE_SUBDOMAINS = True
SECURE_HSTS_PRELOAD = True
```

### Option 3: Manual Headers in Flask

```python
@app.after_request
def add_security_headers(response):
    response.headers['X-Content-Type-Options'] = 'nosniff'
    response.headers['X-Frame-Options'] = 'DENY'
    response.headers['Referrer-Policy'] = 'no-referrer-when-downgrade'
    return response
```

## Why this works

Security middleware intercepts the response from your application and injects the necessary headers before they are sent to the user's browser. This ensures that every page on your site benefits from the same protection level.

## Best Practices

1. **Run `python manage.py check --deploy`**: If using Django, this command will verify that your security settings are correct for production.
2. **CSP for SPAs**: If your Python backend serves a React/Vue frontend, ensure your CSP allows the necessary origins for your API and static assets.
3. **Audit Headers**: Use browser developer tools (Network tab) to verify that headers are actually being sent as expected.
