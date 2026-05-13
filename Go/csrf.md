# CSRF Protection Remediation (Go)

In Go, CSRF protection is typically handled by middleware. The `gorilla/csrf` package is the most widely used solution.

## Using 'gorilla/csrf' (Recommended)
```go
import (
    "github.com/gorilla/csrf"
    "github.com/gorilla/mux"
)

func main() {
    r := mux.NewRouter()
    
    // Protect all routes
    CSRF := csrf.Protect([]byte("32-byte-long-auth-key"))
    
    r.HandleFunc("/signup", SignupHandler).Methods("GET", "POST")
    
    http.ListenAndServe(":8000", CSRF(r))
}

// In your handler (template data):
// "csrfField": csrf.TemplateField(r)
```

## Manual Implementation (Double Submit Cookie)
If not using a library, use the Double Submit Cookie pattern.
```go
func handlePost(w http.ResponseWriter, r *http.Request) {
    cookie, _ := r.Cookie("csrf_token")
    header := r.Header.Get("X-CSRF-Token")
    
    if cookie == nil || cookie.Value != header {
        http.Error(w, "Invalid CSRF Token", http.StatusForbidden)
        return
    }
}
```

## Best Practices
1. **SameSite Cookie Attribute**: Set your session cookies to `SameSite=Lax` or `Strict`.
2. **Secret Rotation**: Rotate your CSRF signing key periodically.
3. **Exclusion**: Only exclude idempotent methods (GET, HEAD, OPTIONS) from CSRF checks.
