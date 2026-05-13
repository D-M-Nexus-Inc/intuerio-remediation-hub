# Cross-Site Scripting (XSS) Remediation (JavaScript / Node.js)

Cross-Site Scripting (XSS) occurs when an application includes untrusted data in a web page without proper validation or escaping.

## Problem: Vulnerable Code

Sending raw user input back to the browser in an HTML response.

```javascript
// VULNERABLE: Direct injection of user input into HTML
app.get('/welcome', (req, res) => {
    const name = req.query.name;
    res.send(`<h1>Welcome, ${name}</h1>`);
});
```

## Solution: Secure Code

Always **escape** user input before rendering it in HTML.

### Option 1: Using `escape-html` (Simple)

```javascript
const escapeHtml = require('escape-html');

// SECURE: Escaping the input before sending
app.get('/welcome', (req, res) => {
    const name = req.query.name;
    res.send(`<h1>Welcome, ${escapeHtml(name)}</h1>`);
});
```

### Option 2: Using Template Engines (EJS / Pug)

Most template engines have auto-escaping enabled by default.

```javascript
// SECURE: EJS escapes <%= %> by default
// welcome.ejs: <h1>Welcome, <%= name %></h1>
app.get('/welcome', (req, res) => {
    res.render('welcome', { name: req.query.name });
});
```

### Option 3: Using `DOMPurify` (For Client-Side / Rich Text)

If you must render HTML from a user, sanitize it first.

```javascript
const createDOMPurify = require('dompurify');
const { JSDOM } = require('jsdom');

const window = new JSDOM('').window;
const DOMPurify = createDOMPurify(window);

// SECURE: Sanitizing dirty HTML
const clean = DOMPurify.sanitize(dirtyHtml);
```

## Why this works

Escaping converts dangerous characters like `<` and `>` into safe HTML entities (`&lt;` and `&gt;`). This prevents the browser from interpreting the input as a `<script>` or other HTML tag.

## Best Practices

1. **Use `textContent` over `innerHTML`**: In client-side JS, always prefer `element.textContent = data` to avoid XSS.
2. **Set Security Headers**: Use the `helmet` middleware in Express to set headers like `X-XSS-Protection` and `Content-Security-Policy`.
3. **Validate Input**: Ensure data matches the expected format (e.g., an email address should look like an email).
