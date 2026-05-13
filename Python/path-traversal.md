# Path Traversal Remediation (Python)

Path Traversal allows an attacker to access files outside of the intended directory by using `../` sequences in file paths.

## Problem: Vulnerable Code

Using user input directly in file system operations.

```python
# VULNERABLE: Direct use of input in open()
filename = request.args.get('file')
with open(f"uploads/{filename}", "rb") as f:
    data = f.read()
```

## Solution: Secure Code

Sanitize the filename and validate the final path.

### Option 1: Using `werkzeug.utils.secure_filename` (Flask)

```python
from werkzeug.utils import secure_filename
import os

# SECURE: Strips directory traversal characters
filename = request.args.get('file')
safe_name = secure_filename(filename)

file_path = os.path.join("uploads", safe_name)
with open(file_path, "rb") as f:
    data = f.read()
```

### Option 2: Using `os.path.abspath` and `os.path.commonpath`

```python
import os

base_dir = os.path.abspath("uploads")
filename = request.args.get('file')
requested_path = os.path.abspath(os.path.join(base_dir, filename))

# SECURE: Check if the resolved path is still inside the base directory
if os.path.commonpath([base_dir]) == os.path.commonpath([base_dir, requested_path]):
    with open(requested_path, "rb") as f:
        data = f.read()
else:
    return "Access Denied", 403
```

## Why this works

`secure_filename` replaces characters like `..` with underscores. The `commonpath` approach ensures that even if a clever encoding bypasses simple filters, the final absolute path on the disk is still physically located within the allowed "uploads" directory.

## Best Practices

1. **Avoid user-controlled paths**: Use an ID in the URL and look up the real filename in a database.
2. **Read-Only Access**: Ensure the application process has only read-only access to the files it needs to serve.
3. **Use a Storage Service**: Consider using S3 or similar cloud storage where traversal is handled by the service API.
