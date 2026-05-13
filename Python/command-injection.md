# Command Injection Remediation (Python)

Command injection occurs when an application executes shell commands with user-supplied data. In Python, this often involves the `os.system()` or `subprocess` modules.

## Vulnerable Example
```python
# BAD: Using shell=True with user input
import subprocess

filename = input("Enter filename: ")
subprocess.call("ls -l " + filename, shell=True)
```

## Secure Implementation

### Using a List of Arguments (Preferred)
Pass arguments as a list and keep `shell=False` (the default). This avoids the shell interpreter entirely.
```python
import subprocess

filename = "myfile.txt"
# GOOD: Arguments are passed as a list, preventing injection
subprocess.run(["ls", "-l", filename])
```

### Using Shlex for Sanitization
If you must build a command string, use `shlex.quote()`.
```python
import shlex
import subprocess

filename = "; rm -rf /"
# GOOD: Properly quoted for shell safety
safe_filename = shlex.quote(filename)
subprocess.run(f"ls -l {safe_filename}", shell=True)
```

## Best Practices
1. **Never use `shell=True`**: It is the most common cause of command injection in Python.
2. **Prefer Native Libraries**: Instead of `ls`, use `os.listdir()`. Instead of `curl`, use `requests`.
3. **Validate Input**: Use regex to ensure input only contains expected characters.
