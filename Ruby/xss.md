# Cross-Site Scripting (XSS) Remediation (Ruby on Rails)

Modern Rails versions enable auto-escaping in templates by default. XSS typically occurs when developers explicitly bypass this protection using `.html_safe` or `raw`.

## Vulnerable Example
```erb
<%# BAD: Bypassing escaping %>
<%= @user_input.html_safe %>
<%= raw @user_input %>

<%# BAD: Unsafe interpolation in JS %>
<script>
  var name = '<%= @user_name %>'; // Can break out of the string
</script>
```

## Secure Implementation

### Default Escaping (Automatic)
```erb
<%# GOOD: Rails escapes this by default %>
<div><%= @user_input %></div>
```

### Safe Content Generation
Use `content_tag` or `link_to` which handle escaping correctly.
```ruby
# GOOD: Helper methods handle attributes securely
content_tag(:div, params[:comment], class: "user-comment")
```

### JSON in Templates
Use `j` or `escape_javascript` for JS contexts.
```erb
# GOOD: Escaping for JS
<script>
  var name = "<%= j @user_name %>";
</script>
```

## Best Practices
1. **Never use `.html_safe`** on strings containing user input.
2. **Use `ERB::Util.html_escape`** if you need to escape strings in controllers or models.
3. **Audit your gems** for known XSS vulnerabilities using `bundler-audit`.
