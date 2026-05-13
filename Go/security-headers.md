# Security Headers Remediation (Go)

Implementing security headers in Go can be done manually using the `http.Header` map or more easily using a middleware package like `secure`.

## Native Implementation (Vanilla)
```go
func securityHeadersMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        w.Header().Set("X-Frame-Options", "SAMEORIGIN")
        w.Header().Set("X-Content-Type-Options", "nosniff")
        w.Header().Set("Strict-Transport-Security", "max-age=31536000; includeSubDomains")
        w.Header().Set("Content-Security-Policy", "default-src 'self'")
        w.Header().Set("Referrer-Policy", "strict-origin-when-cross-origin")
        
        next.ServeHTTP(w, r)
    })
}
```

## Using 'secure' Middleware (Recommended)
The `unrolled/secure` package is the industry standard for Go.
```go
import "github.com/unrolled/secure"

secureMiddleware := secure.New(secure.Options{
    FrameDeny:             true,
    ContentTypeNosniff:    true,
    BrowserXssFilter:      true,
    ContentSecurityPolicy: "default-src 'self'",
})

app := http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    w.Write([]byte("Hello, World!"))
})

http.ListenAndServe(":8080", secureMiddleware.Handler(app))
```

## Best Practices
1. **Use Middleware**: Don't reinvent the wheel; middleware ensures consistent header application.
2. **Start Strict**: Use a strict CSP and relax it only as needed.
3. **Verify**: Use tools like `securityheaders.com` to verify your implementation.
