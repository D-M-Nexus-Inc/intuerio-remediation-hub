# CSRF Protection Remediation (Python)

Cross-Site Request Forgery (CSRF) is a common vulnerability. Django has CSRF protection enabled by default, but Flask requires explicit implementation using the `Flask-WTF` or `Flask-SeaSurf` extensions.

## Flask Implementation (Flask-WTF)
```python
from flask import Flask, render_template
from flask_wtf.csrf import CSRFProtect

app = Flask(__name__)
app.config['SECRET_KEY'] = 'your-super-secret-key'
csrf = CSRFProtect(app)

# In your HTML template:
# <form method="post">
#     <input type="hidden" name="csrf_token" value="{{ csrf_token() }}">
#     ...
# </form>
```

## Django Implementation (Default)
Django handles this automatically via middleware. Ensure it is in your `MIDDLEWARE` setting.
```python
# settings.py
MIDDLEWARE = [
    ...
    'django.middleware.csrf.CsrfViewMiddleware',
    ...
]

# In your template:
# <form method="post">
#     {% csrf_token %}
#     ...
# </form>
```

## Best Practices
1. **Never Disable Globally**: Avoid disabling CSRF protection for convenience.
2. **Use SameSite Cookies**: Set `SESSION_COOKIE_SAMESITE = 'Lax'` in Django or Flask.
3. **Verify Tokens**: Ensure tokens are unique per session and verified for every `POST`, `PUT`, `DELETE` request.
