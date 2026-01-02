# Task Completion Checklist

When completing a coding task in this project, follow these steps:

## 1. Code Quality Checks

```bash
# From the complete/ directory

# Run linter to check code style
make lint

# Run tests with race detection
make tests
```

## 2. Pre-Commit Checklist

- [ ] Code follows the project's naming conventions (see code_style_conventions.md)
- [ ] New interfaces are declared at the consumer site
- [ ] Error handling uses domain errors where applicable
- [ ] Context is passed as the first parameter to functions
- [ ] JSON tags are present on structs that will be serialized
- [ ] Validation tags are present where input validation is needed

## 3. If Adding New Interfaces

```bash
# Regenerate mocks after adding/modifying interfaces
make go-generate
```

## 4. If Modifying Database Layer

- Ensure SQL queries are parameterized (prevent SQL injection)
- Update tests with sqlmock expectations

## 5. If Adding REST Endpoints

- Add appropriate middleware (timeout, CORS if needed)
- Add input validation
- Use proper HTTP status codes
- Handle errors with `getStatusCode()` helper

## 6. Testing Verification

```bash
# Run specific tests for modified packages
go test -v -run TestName ./path/to/package

# Run all tests
make tests

# For detailed test output
make tests-complete
```

## 7. Build Verification

```bash
# Ensure build succeeds
make build

# Optional: Build with race detector
make build-race
```

## 8. Clean Up

```bash
# Clean test artifacts
make clean-artifacts
```

## Common Issues to Avoid

1. **Shadowed variables**: govet checks for shadowing
2. **Unused parameters**: unparam linter checks this
3. **Long lines**: Max 160 characters (lll linter)
4. **Long functions**: Max 150 lines, 80 statements (funlen)
5. **Missing error checks**: errcheck linter
6. **Security issues**: gosec linter checks for common vulnerabilities
