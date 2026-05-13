# Command Injection Remediation (Ruby on Rails)

In Ruby, command injection occurs when user input is passed to methods that execute shell commands, such as `system`, `exec`, or using backticks (``).

## Vulnerable Example
```ruby
# BAD: Backticks interpolate and execute in a shell
filename = params[:file]
`ls -l #{filename}`

# BAD: system with a single string
system("ls -l #{filename}")
```

## Secure Implementation

### Multiple Arguments (Preferred)
When using `system`, `exec`, or `Open3.capture3`, pass each argument as a separate element. This avoids the shell interpreter.
```ruby
# GOOD: system with multiple arguments (no shell interpolation)
filename = "myfile.txt"
system("ls", "-l", filename)
```

### Using `Shellwords.escape`
If you must use a single string, sanitize every argument.
```ruby
require 'shellwords'

# GOOD: Escaped for shell safety
safe_filename = Shellwords.escape(params[:file])
system("ls -l #{safe_filename}")
```

## Best Practices
1. **Never use backticks (``)** or `%x()` with user input.
2. **Use `Open3`**: For more control, use `Open3.capture3` with an array of arguments.
3. **Prefer Native Ruby Methods**: Instead of `system("mkdir", ...)`, use `Dir.mkdir`. Instead of `rm`, use `FileUtils.rm`.
