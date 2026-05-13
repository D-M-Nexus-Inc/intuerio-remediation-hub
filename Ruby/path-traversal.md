# Path Traversal Remediation (Ruby on Rails)

Path traversal in Ruby can be mitigated by using `File.basename` or validating the expanded real path.

## Vulnerable Example
```ruby
# BAD: Direct interpolation of user input
def show
  filename = params[:file]
  send_file "app/assets/images/#{filename}"
end
```

## Secure Implementation

### Using `File.basename`
```ruby
# GOOD: Strip directory information
def show
  filename = File.basename(params[:file])
  send_file "app/assets/images/#{filename}"
end
```

### Expanding and Validating Paths
```ruby
def show
  root = File.expand_path("app/assets/images")
  requested_path = File.expand_path(params[:file], root)

  # GOOD: Check if the path is still within the root
  if requested_path.start_with?(root)
    send_file requested_path
  else
    render plain: "Forbidden", status: :forbidden
  end
end
```

## Best Practices
1. **Rails `send_file`**: Use `send_file` or `send_data` as they have some built-in protections, but don't rely on them alone.
2. **Store Files in Object Storage**: Use services like AWS S3 via `Active Storage` to avoid direct file system access entirely.
3. **Validation**: Use `validates_format_of` in models to ensure filenames only contain alphanumeric characters.
