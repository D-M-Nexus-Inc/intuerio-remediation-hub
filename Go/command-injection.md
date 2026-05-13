# Command Injection Remediation (Go)

Command injection in Go usually happens when using `os/exec` to run system commands with unsanitized user input.

## Vulnerable Example
```go
// BAD: Concatenating input into a single string for a shell
import "os/exec"

func handler(userInput string) {
    // This is dangerous if userInput contains "; rm -rf /"
    cmd := exec.Command("sh", "-c", "echo " + userInput)
    cmd.Run()
}
```

## Secure Implementation

### Direct Argument Passing (Preferred)
Pass each argument as a separate element in the `Command` function. Go does not use a shell by default, so it won't interpret characters like `;`, `&`, or `|`.
```go
import "os/exec"

func handler(userInput string) {
    // GOOD: Arguments are handled safely by the OS, not a shell
    cmd := exec.Command("echo", userInput)
    cmd.Run()
}
```

### Strict Validation
```go
import (
    "os/exec"
    "regexp"
)

func handler(userInput string) {
    // GOOD: Validate input using an allow-list regex
    isAlpha := regexp.MustCompile(`^[a-zA-Z0-9]+$`).MatchString
    if !isAlpha(userInput) {
        return
    }
    
    cmd := exec.Command("ls", "-l", userInput)
    cmd.Run()
}
```

## Best Practices
1. **Avoid `sh -c`**: Never wrap your commands in a shell string.
2. **Use Native Packages**: Instead of `exec.Command("mkdir", ...)`, use `os.Mkdir()`.
3. **Least Privilege**: Ensure the Go binary runs with the minimum necessary system permissions.
