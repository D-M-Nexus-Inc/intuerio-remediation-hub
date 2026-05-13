# Path Traversal Remediation (Go)

In Go, the `path/filepath` package provides tools to sanitize and validate file paths to prevent traversal attacks.

## Vulnerable Example
```go
// BAD: Directly using user input in Open
func handler(w http.ResponseWriter, r *http.Request) {
    filename := r.URL.Query().Get("file")
    data, _ := os.ReadFile("uploads/" + filename)
    w.Write(data)
}
```

## Secure Implementation

### Using `filepath.Base`
`filepath.Base` returns the last element of the path, removing any directory traversal characters like `../`.
```go
import "path/filepath"

func handler(w http.ResponseWriter, r *http.Request) {
    // GOOD: Strip directory info
    filename := filepath.Base(r.URL.Query().Get("file"))
    data, _ := os.ReadFile(filepath.Join("uploads", filename))
    w.Write(data)
}
```

### Path Cleaning and Subdirectory Check
```go
import (
    "path/filepath"
    "strings"
)

func secureRead(userPath string) ([]byte, error) {
    rootDir := "/var/www/uploads"
    
    // GOOD: Clean and Join
    finalPath := filepath.Join(rootDir, userPath)
    
    // Verify finalPath is still under rootDir
    if !strings.HasPrefix(finalPath, filepath.Clean(rootDir)) {
        return nil, fmt.Errorf("Access Denied")
    }
    
    return os.ReadFile(finalPath)
}
```

## Best Practices
1. **Use `filepath.Clean`**: Always normalize paths before processing.
2. **Limit Permissions**: The application user should not have read access to `/etc/passwd` or other sensitive system files.
3. **Use File IDs**: Map numeric IDs to file paths in a database instead of exposing filenames.
