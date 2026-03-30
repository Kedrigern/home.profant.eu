---
title: Go
icon: simple/go
---

## 1. Ecosystem & CLI

* **Init module:** `go mod init <module-name>` (creates `go.mod`).
* **Run dev:** `go run main.go` (compiles to temp, runs, deletes).
* **Build binary:** `go build` (creates a statically linked executable).
* **Cross-compile:** `GOOS=linux GOARCH=arm64 go build`
* **Add dependency:** `go get github.com/gin-gonic/gin` (updates `go.mod` & `go.sum`).
* **Format code:** `go fmt ./...`
* **Read docs:** `go doc <package/function>`

## 2. Syntax, Variables & Pointers
* **Visibility:** Capital first letter = **Public** (Exported), lowercase = **Private** (Unexported).
* **Semicolons:** Inserted automatically. Opening brace `{` *must* be on the same line.
* **Declarations:**
  * Explicit: `var name string = "Value"`
  * Short (type inference): `name := "Value"` (only inside functions).
* **Zero Values:** Uninitialized variables get `0`, `0.0`, `""`, `false`, or `nil` (for pointers/interfaces). No garbage in memory.
* **Pointers:** `&` gets address, `*` dereferences. No pointer arithmetic. Returning a pointer to a local variable is safe (Escape Analysis handles heap allocation).

## 3. Control Flow & Error Handling
**If with Initialization:**
```go
if val, err := doSomething(); err != nil {
    return err // 'val' and 'err' scoped only to if/else block
}
```

**For Loop (The only loop in Go):**
```go
for i := 0; i < 10; i++ { ... }       // Classic
for condition { ... }                 // Like "while"
for { ... }                           // Infinite
for index, value := range slice { ... } // Iterable
```

**Switch:**
Implicit `break` (no fallthrough by default). Use `fallthrough` if needed. Can be used without a condition as a clean `if-else` chain.

**Errors:**
No exceptions (`try/catch`). Errors are values returned as the last argument.
```go
import "errors"

func divide(a, b int) (int, error) {
    if b == 0 {
        return 0, errors.New("division by zero")
    }
    return a / b, nil
}
```

## 4. Structs, Methods & Interfaces (OOP)
No classes or inheritance. Use `struct` for data and receiver functions for methods.

**Structs & Methods:**
```go
type User struct {
    Name string // Public
    age  int    // Private to the package
}

// Pointer receiver allows modifying the struct
func (u *User) SetAge(newAge int) {
    u.age = newAge
}
```

**Composition (Embedding):**
Embed structs anonymously to inherit fields and methods.
```go
type Admin struct {
    User // Anonymous embedded field
    Role string
}
// Usage: admin.Name (direct access to embedded fields)
```

**Interfaces (Implicit "Duck Typing"):**
No `implements` keyword. If a struct has the required methods, it implements the interface.
```go
type Writer interface {
    Write(data string) error
}
```

## 5. Concurrency
**Goroutines:** Lightweight, user-space threads managed by Go runtime.
* Spawn a goroutine: `go functionName()`

**Channels:** Typed pipes to synchronize and send data between goroutines safely.
```go
ch := make(chan int) // Unbuffered channel

go func() {
    ch <- 42         // Send data (blocks until read)
}()

result := <-ch       // Receive data (blocks until sent)
```

## 6. Web APIs (Gin Framework)
Standard library `net/http` is great, but micro-frameworks like `Gin` simplify routing and JSON.
```go
package main

import (
    "net/http"
    "github.com/gin-gonic/gin"
)

func main() {
    r := gin.Default()
    
    r.GET("/api/user/:id", func(c *gin.Context) {
        id := c.Param("id")
        c.JSON(http.StatusOK, gin.H{"status": "ok", "id": id})
    })
    
    r.Run(":8080") // Blocking call
}
```

## 7. Databases & Migrations
The Go community prefers pure SQL or generators over heavy ORMs.

* **Driver:** `pgx` is the standard for PostgreSQL.
* **Code Generation:** `sqlc` is highly recommended (generates typesafe Go code from raw `.sql` files).
* **ORM:** `gorm` is available if Rapid Application Development is preferred.

**Migrations (`golang-migrate` & `//go:embed`):**
Embed raw `.sql` migration files directly into the compiled binary.
```go
import "embed"
import "github.com/golang-migrate/migrate/v4"

//go:embed migrations/*.sql
var migrationFiles embed.FS

// Initialize migrate instance using the embedded filesystem
```

## 8. Configuration & CLI
**12-Factor App:** Read configuration from environment variables.
```go
dbUrl := os.Getenv("DATABASE_URL")
if dbUrl == "" { dbUrl = "default_value" }
```
**CLI Apps:** Use `spf13/cobra` for command routing and `spf13/viper` for merging env vars, config files, and CLI flags. This allows building "swiss-army knife" single binaries (e.g., `app server` vs `app migrate`).

## 9. Testing
Built-in framework. No external tools needed. Use `go test ./...`.
* File naming: `name_test.go`
* Function naming: `func TestName(t *testing.T)`

**Table-Driven Tests (Standard Pattern):**
```go
import "testing"

func TestAdd(t *testing.T) {
    tests := []struct {
        name     string
        a, b     int
        expected int
    }{
        {"positive", 2, 3, 5},
        {"zeroes", 0, 0, 0},
    }

    for _, tc := range tests {
        t.Run(tc.name, func(t *testing.T) {
            result := Add(tc.a, tc.b)
            if result != tc.expected {
                t.Errorf("Expected %d, got %d", tc.expected, result)
            }
        })
    }
}
```

## 10. Project Structure & Documentation
* **Package by Feature:** Group code by domain (e.g., `/user`), not by layer (e.g., `/models`, `/controllers`). Avoid `utils` packages.
* **`cmd/` directory:** Contains the `main.go` entry points (e.g., `cmd/api/main.go`).
* **`internal/` directory:** Code inside cannot be imported by other external repositories.
* **Comments:** Docstrings are just standard `//` comments directly above the declaration without empty lines. Must start with the name of the documented entity.
