# Cross-Site Scripting (XSS) Remediation (Go)

In Go, XSS is primarily prevented by the `html/template` package, which provides context-aware escaping. However, using `text/template` or raw output can lead to vulnerabilities.

## Vulnerable Example
```go
// BAD: Using text/template (no auto-escaping)
import "text/template"
tmpl, _ := template.New("test").Parse("<div>{{.}}</div>")
tmpl.Execute(w, userInput) // userInput: <script>alert(1)</script>
```

## Secure Implementation

### Using html/template (Automatic)
```go
// GOOD: html/template automatically escapes based on context
import "html/template"

func handler(w http.ResponseWriter, r *http.Request) {
    tmpl := template.Must(template.New("web").Parse("<h1>Hello, {{.}}</h1>"))
    tmpl.Execute(w, r.URL.Query().Get("name"))
}
```

### Manual Escaping
If you must output strings directly to HTML:
```go
import "html"

// GOOD: Manual escaping
escaped := html.EscapeString(userInput)
fmt.Fprintf(w, "<div>%s</div>", escaped)
```

## Best Practices
1. **Always use `html/template`** for web output.
2. **Set the correct Content-Type** header (e.g., `text/html; charset=utf-8`).
3. **Use a Content Security Policy (CSP)** as an additional defense layer.
