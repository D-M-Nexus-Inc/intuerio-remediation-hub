# SQL Injection Remediation (Go)

SQL injection in Go occurs when user input is concatenated directly into a query string rather than using parameterized placeholders provided by the `database/sql` package.

## Vulnerable Example
```go
// BAD: Direct concatenation of user input
query := "SELECT * FROM users WHERE username = '" + username + "'"
rows, err := db.Query(query)
```

## Secure Implementation
Always use placeholders (`?` for MySQL/SQL Server, `$1`, `$2` for PostgreSQL) in your queries.

### Using database/sql (Vanilla)
```go
// GOOD: Using parameterized queries
query := "SELECT id, email FROM users WHERE username = ?"
rows, err := db.Query(query, username)
if err != nil {
    log.Fatal(err)
}
defer rows.Close()
```

### Using GORM (ORM)
If you are using an ORM like GORM, use the built-in parameterization.
```go
// GOOD: GORM automatically handles parameterization
var user User
db.Where("username = ?", username).First(&user)
```

## Best Practices
1. **Never use `fmt.Sprintf`** to build queries.
2. **Use an ORM** like GORM or Ent for higher-level abstraction.
3. **Scan for vulnerabilities** using tools like `gosec` in your CI/CD pipeline.
