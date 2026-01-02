# Project Overview: go-clean-arch

## Purpose
This is a Go Clean Architecture implementation example for learning purposes. The project demonstrates Uncle Bob's Clean Architecture principles in Go, serving as a reference implementation for API development.

## Learning Context
- Target: A learner who understands Clean Architecture basics and Go fundamentals
- Method: Copy code from `complete/` folder to `practice/` folder to learn by doing
- Goal: Skill development for upcoming Go-based API development work

## Tech Stack
- **Language**: Go 1.20
- **Web Framework**: Echo v4 (labstack/echo)
- **Database**: MySQL with go-sql-driver/mysql
- **Testing**: 
  - stretchr/testify (assertions)
  - go-sqlmock (database mocking)
  - mockery (interface mocking)
  - go-faker (test data generation)
- **Logging**: sirupsen/logrus
- **Configuration**: joho/godotenv
- **Validation**: go-playground/validator.v9
- **Concurrency**: golang.org/x/sync

## Architecture Layers

```
Delivery (REST) → Service → Repository → Domain
```

1. **Domain Layer** (`domain/`)
   - Core entities: `Article`, `Author`
   - Domain errors: `ErrNotFound`, `ErrConflict`, `ErrInternalServerError`, `ErrBadParamInput`
   - No external dependencies

2. **Service Layer** (`article/`)
   - Business logic and use cases
   - Defines repository interfaces (dependency inversion)
   - Orchestrates data flow

3. **Repository Layer** (`internal/repository/`)
   - MySQL implementations in `mysql/` subdirectory
   - Data persistence logic

4. **Delivery Layer** (`internal/rest/`)
   - HTTP handlers using Echo
   - Defines service interfaces
   - Request validation and response formatting
   - Middleware: CORS, Timeout

## Key Design Patterns
- **Interface at Consumer**: Interfaces declared where used, not where implemented
- **Internal Package**: `internal/` prevents external imports of implementation details
- **Dependency Injection**: `app/main.go` is the composition root

## Directory Structure
```
complete/
├── app/           # Application entry point (main.go)
├── article/       # Service layer with interfaces
│   └── mocks/     # Generated mocks for testing
├── domain/        # Core entities and errors
├── internal/
│   ├── repository/mysql/  # MySQL implementations
│   ├── rest/             # HTTP handlers
│   │   ├── middleware/   # CORS, timeout
│   │   └── mocks/       # Handler mocks
│   └── workers/         # Background workers (placeholder)
├── misc/make/     # Makefile includes
├── Makefile       # Build commands
├── compose.yaml   # Docker Compose config
└── .golangci.yaml # Linter configuration
```

## Environment Configuration
Required `.env` file variables (copy from `example.env`):
- `DATABASE_HOST`, `DATABASE_PORT`, `DATABASE_USER`, `DATABASE_PASS`, `DATABASE_NAME`
- `SERVER_ADDRESS` (default: `:9090`)
- `CONTEXT_TIMEOUT` (request timeout in seconds)
