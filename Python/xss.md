# Cross-Site Scripting (XSS) Remediation (Python)

Cross-Site Scripting (XSS) in Python web applications occurs when untrusted input is rendered in a template without proper escaping.

## Problem: Vulnerable Code

Using the `| safe` filter in Jinja2 or returning raw strings from a Flask route.

```python
# VULNERABLE: Disabling auto-escaping
from flask import Flask, request, render_template_string

@app.route('/hello')
def hello():
    name = request.args.get('name')
    # Vulnerable to XSS if 'name' contains <script>
    return render_template_string("<h1>Hello {{ name | safe }}</h1>", name=name)
```

## Solution: Secure Code

Keep auto-escaping enabled and use libraries like `MarkupSafe` for manual escaping.

### Option 1: Default Jinja2 Escaping (Recommended)

Jinja2 (used by Flask) escapes variables by default. Just avoid the `| safe` filter.

```python
# SECURE: Default escaping
@app.route('/hello')
def hello():
    name = request.args.get('name')
    # name is automatically escaped
    return render_template("hello.html", name=name)
```

### Option 2: Using `markupsafe.escape`

```python
from markupsafe import escape

# SECURE: Manual escaping for raw strings
@app.route('/hello')
def hello():
    name = request.args.get('name')
    return f"<h1>Hello {escape(name)}</h1>"
```

### Option 3: Using `bleach` for HTML Sanitization

If you must allow some HTML tags (e.g., in a blog comment), use `bleach`.

```python
import bleach

# SECURE: Only allow specific tags
html_content = request.form.get('comment')
clean_html = bleach.clean(html_content, tags=['b', 'i', 'em', 'strong'])
```

## Why this works

Escaping converts special characters like `<` and `>` into HTML entities. The browser then treats them as text to be displayed, rather than code to be executed. Sanitization (with `bleach`) goes a step further by stripping out dangerous tags and attributes while keeping a safe subset of HTML.

## Best Practices

1. **Don't use `| safe`**: Only use this filter when you are 100% sure the content is trusted and has been sanitized.
2. **HTTP-Only Cookies**: Set the `HttpOnly` flag on sensitive cookies to prevent them from being stolen via XSS.
3. **Use a CSP**: Implement a Content Security Policy to restrict where scripts can be loaded from.
