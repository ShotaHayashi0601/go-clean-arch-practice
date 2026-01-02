# Suggested Commands for go-clean-arch

**Important**: All commands must be run from the `complete/` directory.

## Development Environment

```bash
# Start development environment (MySQL + hot reload with air)
make up

# Stop development environment
make down

# Teardown environment and clean up volumes
make destroy
```

## Testing

```bash
# Run all tests with race detection and coverage
make tests

# Run tests and show detailed results
make tests-complete

# Run a single test
go test -v -run TestFunctionName ./path/to/package

# Example: Run specific service test
go test -v -run TestFetch ./article/
```

## Code Quality

```bash
# Run linter (golangci-lint)
make lint

# Generate mocks for testing
make go-generate
```

## Building

```bash
# Build the application
make build

# Build with race detector enabled
make build-race
```

## Docker

```bash
# Build Docker image
make image-build

# Run with Docker Compose for testing
make dev-env-test
```

## Dependencies

```bash
# Install development dependencies
make install-deps

# Check if required tools are available
make deps
```

## Cleaning

```bash
# Clean all artifacts and docker
make clean

# Clean only test artifacts (*.out files)
make clean-artifacts

# Clean dangling docker images
make clean-docker
```

## Windows System Commands

Since this project runs on Windows, use these alternatives:
- Use `dir` instead of `ls` in PowerShell/cmd (or `ls` in PowerShell)
- Use `where` instead of `which`
- Use `findstr` instead of `grep` (or install grep via Git Bash/WSL)
- Git Bash provides Unix-like commands
- WSL (Windows Subsystem for Linux) is recommended for make commands

## Go Commands (Direct)

```bash
# Run the application directly
go run ./app/

# Format code
go fmt ./...

# Vet code
go vet ./...

# Get dependencies
go mod download

# Tidy dependencies
go mod tidy
```
