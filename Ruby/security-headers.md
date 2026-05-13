# Security Headers Remediation (Ruby on Rails)

Rails includes a built-in mechanism for security headers via the `config.action_dispatch.default_headers` setting or the `Content Security Policy` initializer.

## Default Rails Headers
In `config/application.rb` or `config/environments/production.rb`:
```ruby
# Rails handles these by default in modern versions
config.action_dispatch.default_headers = {
  'X-Frame-Options' => 'SAMEORIGIN',
  'X-XSS-Protection' => '0', # Modern browsers prefer CSP
  'X-Content-Type-Options' => 'nosniff',
  'X-Download-Options' => 'noopen',
  'X-Permitted-Cross-Domain-Policies' => 'none',
  'Referrer-Policy' => 'strict-origin-when-cross-origin'
}
```

## Content Security Policy (CSP)
In `config/initializers/content_security_policy.rb`:
```ruby
Rails.application.config.content_security_policy do |policy|
  policy.default_src :self, :https
  policy.font_src    :self, :https, :data
  policy.img_src     :self, :https, :data
  policy.object_src  :none
  policy.script_src  :self, :https
  policy.style_src   :self, :https

  # Specify URI for violation reports
  # policy.report_uri "/csp-violation-report-endpoint"
end
```

## Best Practices
1. **Use the Rails Initializer**: It's the most robust way to manage CSP in Rails.
2. **HSTS**: Ensure `config.force_ssl = true` is set in production to enable `Strict-Transport-Security`.
3. **Audit**: Use `Brakeman` to check for missing security headers in your configuration.
