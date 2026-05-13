# SQL Injection Remediation (Python)

SQL Injection (SQLi) in Python occurs when untrusted data is directly formatted into SQL strings.

## Problem: Vulnerable Code

Using f-strings or `.format()` to build queries is highly insecure.

```python
# VULNERABLE: Direct string formatting
user_id = request.args.get('id')
query = f"SELECT * FROM users WHERE id = {user_id}"

cursor.execute(query)
```

## Solution: Secure Code

Use **Parameterized Queries**. Pass the data as a separate argument to the `execute()` method.

### Option 1: Using `sqlite3` (or `psycopg2`)

```python
# SECURE: Using placeholders (?) for sqlite3
user_id = request.args.get('id')
query = "SELECT * FROM users WHERE id = ?"

# Data is passed as a tuple
cursor.execute(query, (user_id,))
```

*Note: For PostgreSQL (`psycopg2`), use `%s` as the placeholder instead of `?`.*

### Option 2: Using SQLAlchemy ORM

```python
# SECURE: SQLAlchemy handles parameterization automatically
user_id = request.args.get('id')
user = session.query(User).filter(User.id == user_id).first()
```

### Option 3: Using Django ORM

```python
# SECURE: Django ORM is safe by default
user_id = request.GET.get('id')
user = User.objects.get(id=user_id)
```

## Why this works

By using placeholders, the SQL driver ensures that the database receives the query logic and the data as separate entities. The database then treats the parameters as literal values, preventing any malicious SQL syntax from being executed.

## Best Practices

1. **Never use `format()` or `%` for SQL**: These are for general string formatting, not for building database queries.
2. **Use an ORM**: Django, SQLAlchemy, and Peewee provide high-level abstractions that make SQLi much harder to introduce.
3. **Input Type Validation**: Ensure that an `id` is actually an integer before using it.
