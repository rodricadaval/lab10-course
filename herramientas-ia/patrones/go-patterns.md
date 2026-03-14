# Go Architecture Patterns

## Project Layout: Clean Architecture

Follow the **hexagonal architecture** for clear separation of concerns:

```
myapp/
├── cmd/
│   └── server/
│       └── main.go          # Entry point
├── internal/
│   ├── domain/              # Business logic, entities
│   │   └── user.go
│   ├── ports/               # Interfaces (what we depend on)
│   │   ├── user_repository.go
│   │   └── email_service.go
│   ├── adapters/            # Implementations (handlers, DB, external services)
│   │   ├── http/
│   │   ├── postgres/
│   │   └── smtp/
│   └── service/             # Use cases / business logic
│       └── user_service.go
└── pkg/
    └── shared/              # Utilities, helpers (reusable across services)
```

**Rules:**
- `internal/` is private to this service
- `pkg/` can be imported by other services
- `cmd/` contains only entry points, minimal logic

---

## Interface-Driven Design

Define interfaces at the consumer (what you need), implement at the provider.

```go
// internal/ports/user_repository.go
package ports

import "context"

type User struct {
    ID    string
    Email string
    Name  string
}

// Define at the consumer
type UserRepository interface {
    GetByID(ctx context.Context, id string) (*User, error)
    Save(ctx context.Context, u *User) error
    Delete(ctx context.Context, id string) error
}
```

```go
// internal/service/user_service.go
package service

import "context"

type UserService struct {
    repo ports.UserRepository
}

func NewUserService(repo ports.UserRepository) *UserService {
    return &UserService{repo: repo}
}

func (s *UserService) GetUser(ctx context.Context, id string) (*ports.User, error) {
    return s.repo.GetByID(ctx, id)
}
```

```go
// internal/adapters/postgres/user_repository.go
package postgres

import "context"

type PostgresUserRepository struct {
    db *sql.DB
}

func (r *PostgresUserRepository) GetByID(ctx context.Context, id string) (*ports.User, error) {
    // Implementation
    return &ports.User{}, nil
}
```

**Benefits:**
- Testable: inject mocks
- Swappable: change DB without changing service
- Clear dependencies

---

## Error Handling

Use error wrapping and sentinel errors for context-aware debugging.

```go
// Define sentinel errors
var (
    ErrNotFound     = errors.New("user not found")
    ErrUnauthorized = errors.New("unauthorized")
    ErrInternal     = errors.New("internal error")
)

// Custom error type for more info
type ValidationError struct {
    Field   string
    Message string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("%s: %s", e.Field, e.Message)
}

// Wrap errors with context
func (s *UserService) CreateUser(ctx context.Context, email string) error {
    if email == "" {
        return &ValidationError{Field: "email", Message: "required"}
    }

    user := &ports.User{Email: email}
    if err := s.repo.Save(ctx, user); err != nil {
        return fmt.Errorf("failed to save user: %w", err)
    }
    return nil
}
```

**HTTP handler pattern:**
```go
func (h *UserHandler) GetUser(w http.ResponseWriter, r *http.Request) {
    id := mux.Vars(r)["id"]
    user, err := h.service.GetUser(r.Context(), id)

    if err != nil {
        if errors.Is(err, ports.ErrNotFound) {
            http.Error(w, "Not found", http.StatusNotFound)
            return
        }
        http.Error(w, "Internal error", http.StatusInternalServerError)
        return
    }

    json.NewEncoder(w).Encode(user)
}
```

---

## Dependency Injection (No Frameworks)

Pass dependencies in constructors:

```go
type App struct {
    userService *service.UserService
    handler     *adapters.UserHandler
}

func NewApp(db *sql.DB, logger *log.Logger) *App {
    userRepo := postgres.NewUserRepository(db)
    userService := service.NewUserService(userRepo)
    handler := adapters.NewUserHandler(userService, logger)

    return &App{
        userService: userService,
        handler:     handler,
    }
}
```

For complex dependency graphs, consider **wire** or **fx** as code generators/frameworks. For most services, manual DI is clearer.

---

## Repository Pattern

Encapsulate data access logic:

```go
// internal/adapters/postgres/user_repository.go
type PostgresUserRepository struct {
    db *sql.DB
}

func (r *PostgresUserRepository) GetByID(ctx context.Context, id string) (*ports.User, error) {
    row := r.db.QueryRowContext(ctx, "SELECT id, email, name FROM users WHERE id = $1", id)
    user := &ports.User{}
    err := row.Scan(&user.ID, &user.Email, &user.Name)
    if err == sql.ErrNoRows {
        return nil, ports.ErrNotFound
    }
    return user, err
}

func (r *PostgresUserRepository) Save(ctx context.Context, u *ports.User) error {
    _, err := r.db.ExecContext(ctx,
        "INSERT INTO users (id, email, name) VALUES ($1, $2, $3) ON CONFLICT (id) DO UPDATE SET email=$2, name=$3",
        u.ID, u.Email, u.Name,
    )
    return err
}
```

---

## Key Takeaways

- **Structure matters** — hexagonal layout enforces separation of concerns
- **Interfaces first** — define at consumer, implement at provider
- **Errors are data** — wrap with context, don't ignore them
- **No magic** — manual DI keeps things explicit and testable
- **Repositories** — single source of truth for data access