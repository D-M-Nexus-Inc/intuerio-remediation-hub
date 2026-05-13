# CSRF Protection Remediation (Ruby on Rails)

Ruby on Rails provides strong, built-in CSRF protection by default. It generates a unique authenticity token for each session and validates it on every non-GET request.

## Standard Implementation
In `app/controllers/application_controller.rb`:
```ruby
class ApplicationController < ActionController::Base
  # GOOD: This is enabled by default in Rails
  protect_from_forgery with: :exception
end
```

## Form Implementation
In your views (ERB):
```erb
<%# GOOD: form_with automatically injects the CSRF token %>
<%= form_with url: posts_path do |form| %>
  ...
<% end %>
```

## AJAX Requests
Ensure the CSRF token is included in your request headers. Rails `javascript_importmap_tags` or `ujs` handle this automatically if the `csrf-param` and `csrf-token` meta tags are present.
```html
<!-- In layouts/application.html.erb -->
<%= csrf_meta_tags %>
```

## Best Practices
1. **Never use `skip_before_action :verify_authenticity_token`** unless you are implementing an API that uses a different authentication method (like JWT or API Keys).
2. **Use `form_with`**: It is the modern Rails standard and handles security features automatically.
3. **SameSite Cookies**: Ensure `config.action_dispatch.cookies_same_site_protection = :lax` is set in your environment config.
