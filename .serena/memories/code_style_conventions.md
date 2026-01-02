# Code Style and Conventions

## Naming Conventions

### Packages
- Use short, lowercase names
- Examples: `domain`, `article`, `rest`, `mysql`

### Structs and Interfaces
- PascalCase for exported types: `Article`, `Service`, `ArticleRepository`
- Interfaces typically end with purpose: `ArticleRepository`, `ArticleService`

### Fields
- PascalCase for exported fields: `ID`, `Title`, `Content`
- camelCase for unexported fields: `articleRepo`, `authorRepo`

### Functions and Methods
- PascalCase for exported: `NewService()`, `Fetch()`, `GetByID()`
- camelCase for unexported: `fillAuthorDetails()`, `isRequestValid()`

### Variables
- camelCase for local variables
- Constants: `defaultNum`, `defaultTimeout`

## Struct Tags

Use JSON and validation tags on domain entities:
```go
type Article struct {
    ID        int64     `json:"id"`
    Title     string    `json:"title" validate:"required"`
    Content   string    `json:"content" validate:"required"`
    Author    Author    `json:"author"`
    UpdatedAt time.Time `json:"updated_at"`
    CreatedAt time.Time `json:"created_at"`
}
```

## Interface Declaration

**Key Pattern**: Interfaces are declared at the consumer site, not the implementation site.

```go
// In article/service.go (consumer)
type ArticleRepository interface {
    Fetch(ctx context.Context, cursor string, num int64) ([]domain.Article, string, error)
    GetByID(ctx context.Context, id int64) (domain.Article, error)
    // ...
}
```

## Constructor Pattern

Use `NewXxx` functions for creating instances:
```go
func NewService(a ArticleRepository, ar AuthorRepository) *Service {
    return &Service{
        articleRepo: a,
        authorRepo:  ar,
    }
}
```

## Error Handling

- Use predefined domain errors in `domain/errors.go`
- Wrap with context when appropriate
- Return errors, don't panic

```go
var (
    ErrInternalServerError = errors.New("internal Server Error")
    ErrNotFound = errors.New("your requested Item is not found")
    ErrConflict = errors.New("your Item already exist")
    ErrBadParamInput = errors.New("given Param is not valid")
)
```

## Mocking

Use mockery for generating mocks:
```go
//go:generate mockery --name ArticleRepository
```

## Linter Rules (golangci-lint)

Enabled linters:
- `errcheck` - Check error handling
- `funlen` - Function length (max 150 lines, 80 statements)
- `goconst` - Find repeated strings that could be constants
- `gocyclo` - Cyclomatic complexity (max 50)
- `gosec` - Security issues
- `govet` - Vet checks including shadowing
- `lll` - Line length (max 160 chars)
- `misspell` - Spelling errors
- `revive` - Linting (confidence 0.8)
- `staticcheck`, `ineffassign`, `unconvert`, `unparam`, `unused`

## File Organization

- One main type per file (e.g., `article.go` for Article handler)
- Tests in `*_test.go` files alongside source
- Mocks in `mocks/` subdirectory

## Context Usage

Always pass `context.Context` as the first parameter:
```go
func (s *Service) Fetch(ctx context.Context, cursor string, num int64) ([]domain.Article, string, error)
```
