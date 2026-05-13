# Cross-Site Request Forgery (CSRF) Remediation (JavaScript / Node.js)

CSRF allows an attacker to perform actions on behalf of an authenticated user without their consent.

## Problem: Vulnerable Code

An Express app that accepts state-changing requests without validating a secret token.

```javascript
// VULNERABLE: No CSRF protection
app.post('/update-profile', (req, res) => {
    // Updates user profile without checking for a token
    db.updateUser(req.session.userId, req.body);
    res.send('Profile updated');
});
```

## Solution: Secure Code

Use a CSRF protection middleware like `csurf` or implement Double Submit Cookies.

### Option 1: Using `csurf` (Express)

```javascript
const csrf = require('csurf');
const cookieParser = require('cookie-parser');

app.use(cookieParser());
app.use(csrf({ cookie: true }));

// SECURE: Form must include the token
app.get('/form', (req, res) => {
  res.render('send', { csrfToken: req.csrfToken() });
});

app.post('/process', (req, res) => {
  res.send('Data is being processed');
});
```

### Option 2: Using `helmet` (Security Headers)

While not a full CSRF fix, `helmet` helps by setting the `SameSite` attribute on cookies.

```javascript
const helmet = require('helmet');
app.use(helmet());

// Ensure session cookies use SameSite=Lax or Strict
app.use(session({
  cookie: { sameSite: 'lax' }
}));
```

## Why this works

CSRF protection works by requiring a unique, cryptographically strong token for every non-GET request. The attacker cannot read this token from the user's browser (due to the Same-Origin Policy), so they cannot include it in a forged request.

## Best Practices

1. **Use `SameSite` Cookies**: Set `SameSite: Lax` or `Strict` on all sensitive cookies.
2. **Avoid GET for State Changes**: Never use GET requests for actions that modify data (e.g., `/logout`, `/delete`).
3. **Verify Origin/Referer**: Check these headers as a secondary defense.
