# Path Traversal Remediation (Node.js)

Path traversal vulnerabilities occur when user input is used to build file paths without proper sanitization, allowing attackers to access files outside the intended directory.

## Vulnerable Example
```javascript
// BAD: Direct concatenation of user input
const path = require('path');
const fs = require('fs');

app.get('/view-file', (req, res) => {
    const filename = req.query.file;
    const filepath = path.join(__dirname, 'uploads', filename);
    
    fs.readFile(filepath, (err, data) => {
        res.send(data);
    });
});
```

## Secure Implementation

### Using `path.basename`
Strip all directory information from the input to ensure only a filename is used.
```javascript
const path = require('path');

app.get('/view-file', (req, res) => {
    // GOOD: Strip directory info
    const filename = path.basename(req.query.file);
    const filepath = path.join(__dirname, 'uploads', filename);
    
    // ...
});
```

### Path Normalization and Validation
Verify that the resolved path still starts with the intended directory.
```javascript
const path = require('path');

const rootDir = path.join(__dirname, 'uploads');
const requestedPath = path.join(rootDir, req.query.file);

// GOOD: Verify normalized path is within root
if (!requestedPath.startsWith(rootDir)) {
    return res.status(403).send('Forbidden');
}
```

## Best Practices
1. **Never trust user input** for file paths.
2. **Use an Allow-list**: If possible, only allow specific file IDs and map them to paths in a database.
3. **Least Privilege**: Run the Node.js process with a user that only has access to the necessary directories.
