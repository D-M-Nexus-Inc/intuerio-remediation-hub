# Cross-Site Scripting (XSS) Remediation (PHP)

Cross-Site Scripting (XSS) occurs when an application includes untrusted data in a web page without proper validation or escaping. This allows attackers to execute malicious scripts in the victim's browser.

## Problem: Vulnerable Code

Outputting user-supplied data directly to the browser is the most common cause of XSS.

```php
<?php
// VULNERABLE: Direct echo of user input
$name = $_GET['name'];
echo "<h1>Welcome, " . $name . "</h1>";
?>
```

## Solution: Secure Code

Always **escape** output data based on the context (HTML, JavaScript, CSS, etc.).

### Option 1: Using `htmlspecialchars()` (Vanilla PHP)

```php
<?php
// SECURE: Using htmlspecialchars to escape HTML entities
$name = $_GET['name'];
echo "<h1>Welcome, " . htmlspecialchars($name, ENT_QUOTES, 'UTF-8') . "</h1>";
?>
```

### Option 2: Using Blade Templates (Laravel)

Laravel's Blade engine escapes data by default using the `{{ }}` syntax.

```blade
{{-- SECURE: Blade automatically escapes this output --}}
<h1>Welcome, {{ $name }}</h1>

{{-- WARNING: Only use {!! !!} for data you TRUST and have already sanitized --}}
<div>{!! $trustedHtml !!}</div>
```

### Option 3: Using a Library (e.g., HTML Purifier)

For cases where you must allow some HTML (like a CMS editor), use a library to sanitize it.

```php
// SECURE: Sanitizing HTML with HTML Purifier
$purifier = new HTMLPurifier();
$cleanHtml = $purifier->purify($dirtyHtml);
echo $cleanHtml;
```

## Why this works

Escaping converts special characters (like `<` and `>`) into their HTML entity equivalents (like `&lt;` and `&gt;`). The browser then displays these as literal text instead of interpreting them as HTML tags or script blocks.

## Best Practices

1. **Escape on Output**: Don't escape data when saving to the database; escape it right before it's sent to the browser.
2. **Context Matters**: Escaping for an HTML attribute is different from escaping for a `<script>` block.
3. **Use Content Security Policy (CSP)**: Implement a strong CSP header to prevent unauthorized scripts from running.
