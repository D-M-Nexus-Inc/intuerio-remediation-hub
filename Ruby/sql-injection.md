# SQL Injection Remediation (Ruby on Rails)

Active Record (the default Rails ORM) provides built-in protection against SQL injection, but developers can still introduce vulnerabilities by using raw SQL strings with string interpolation.

## Vulnerable Example
```ruby
# BAD: String interpolation in raw SQL
User.where("username = '#{params[:username]}'")

# BAD: Direct execution
ActiveRecord::Base.connection.execute("SELECT * FROM users WHERE id = #{params[:id]}")
```

## Secure Implementation

### Using Active Record Hash Syntax (Preferred)
```ruby
# GOOD: Rails automatically sanitizes hash arguments
User.where(username: params[:username])
```

### Using Array Placeholders
If you need complex logic, use the array syntax.
```ruby
# GOOD: Parameterized array syntax
User.where("username = ? AND status = ?", params[:username], "active")
```

### Sanitizing Raw SQL
If you must use raw SQL, use `sanitize_sql_array`.
```ruby
# GOOD: Manual sanitization
query = ActiveRecord::Base.sanitize_sql_array(["SELECT * FROM users WHERE id = ?", params[:id]])
ActiveRecord::Base.connection.execute(query)
```

## Best Practices
1. **Avoid `find_by_sql`** and `execute` unless absolutely necessary.
2. **Never use string interpolation (`#{}`)** inside any database query method.
3. **Use static analysis** tools like `Brakeman` to find SQLi in your Rails apps.
