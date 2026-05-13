# Cross-Site Request Forgery (CSRF) Remediation (PHP)

CSRF is an attack that forces an authenticated user to execute unwanted actions on a web application in which they are currently authenticated.

## Problem: Vulnerable Code

A form that does not include a unique, unpredictable token is vulnerable to CSRF.

```html
<!-- VULNERABLE: No CSRF protection -->
<form action="/update-email" method="POST">
    <input type="email" name="email">
    <button type="submit">Update Email</button>
</form>
```

## Solution: Secure Code

Include a **CSRF Token** in every state-changing request (POST, PUT, DELETE).

### Option 1: Manual Token Implementation (Vanilla PHP)

```php
<?php
// On form generation:
session_start();
if (empty($_SESSION['csrf_token'])) {
    $_SESSION['csrf_token'] = bin2hex(random_bytes(32));
}
?>

<form action="/update-email" method="POST">
    <input type="hidden" name="csrf_token" value="<?php echo $_SESSION['csrf_token']; ?>">
    <input type="email" name="email">
    <button type="submit">Update Email</button>
</form>

<?php
// On form submission:
if ($_POST['csrf_token'] !== $_SESSION['csrf_token']) {
    die("CSRF token validation failed.");
}
?>
```

### Option 2: Using Laravel (Automatic)

Laravel provides built-in CSRF protection for all POST, PUT, PATCH, or DELETE routes.

```blade
{{-- SECURE: Laravel adds the hidden token field for you --}}
<form action="/update-email" method="POST">
    @csrf
    <input type="email" name="email">
    <button type="submit">Update Email</button>
</form>
```

## Why this works

A CSRF token is a unique value generated for each user session. Since an attacker cannot predict or read this token (due to Same-Origin Policy), they cannot forge a request that includes the correct token.

## Best Practices

1. **Use Framework Middleware**: Most modern frameworks (Laravel, Symfony) have CSRF protection enabled by default. Don't disable it!
2. **Check the Origin**: In addition to tokens, check the `Origin` or `Referer` headers for added security.
3. **SameSite Cookie Attribute**: Set your session cookies to `SameSite=Lax` or `SameSite=Strict`.
