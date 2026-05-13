# SQL Injection Remediation (JavaScript / Node.js)

SQL Injection (SQLi) in Node.js occurs when untrusted data is concatenated into database queries.

## Problem: Vulnerable Code

Using template literals or string concatenation to build queries directly.

```javascript
// VULNERABLE: Direct concatenation in the query string
const userId = req.query.id;
const query = `SELECT * FROM users WHERE id = ${userId}`;

db.query(query, (err, result) => {
    // ...
});
```

## Solution: Secure Code

Use **Parameterized Queries** (also called Bind Variables). The database driver handles the escaping and quoting for you.

### Option 1: Using `pg` (PostgreSQL)

```javascript
// SECURE: Using parameterized queries with 'pg'
const userId = req.query.id;
const query = 'SELECT * FROM users WHERE id = $1';

pool.query(query, [userId], (err, res) => {
    if (err) throw err;
    console.log(res.rows[0]);
});
```

### Option 2: Using `mysql2`

```javascript
// SECURE: Using placeholders with 'mysql2'
const userId = req.query.id;

connection.execute(
  'SELECT * FROM users WHERE id = ?',
  [userId],
  function(err, results) {
    console.log(results);
  }
);
```

### Option 3: Using an ORM (Prisma / Sequelize)

ORMs handle parameterization automatically.

```javascript
// SECURE: Using Prisma ORM
const userId = parseInt(req.query.id);
const user = await prisma.user.findUnique({
  where: { id: userId },
});
```

## Why this works

When you use parameters (`$1` or `?`), the query structure is sent to the database separately from the data. The database engine then treats the data strictly as a value, making it impossible for input like `1 OR 1=1` to change the query logic.

## Best Practices

1. **Avoid `eval()`**: Never use `eval()` on strings containing user input.
2. **Use Environment Variables**: Store database credentials in `.env` files, never hardcoded.
3. **Validation**: Use libraries like `joi` or `zod` to validate the type and format of input before it even reaches the database logic.
