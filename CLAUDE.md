# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Go Clean Architecture implementation example. The main application code is in the `complete/` directory.

## Common Commands

All commands should be run from the `complete/` directory.

```bash
# Run tests with race detection and coverage
make tests

# Run linter (golangci-lint)
make lint

# Build the application
make build

# Build with race detector
make build-race

# Generate mocks for testing
make go-generate

# Start development environment (MySQL + hot reload)
make up

# Stop development environment
make down

# Teardown environment and clean up
make destroy
```

### Running a Single Test

```bash
go test -v -run TestFunctionName ./path/to/package
```

## Architecture

The codebase follows Uncle Bob's Clean Architecture with four layers:

### Layer Structure

1. **Domain Layer** (`domain/`)

   - Core business entities (`Article`, `Author`)
   - Domain errors (`ErrNotFound`, `ErrConflict`, etc.)
   - No external dependencies

2. **Service Layer** (`article/`)

   - Business logic and use cases
   - Defines repository interfaces it needs (dependency inversion)
   - Orchestrates data flow between repositories and delivery layer

3. **Repository Layer** (`internal/repository/`)

   - Data persistence implementations
   - MySQL implementations in `internal/repository/mysql/`

4. **Delivery Layer** (`internal/rest/`)
   - HTTP handlers using Echo framework
   - Defines service interfaces it consumes
   - Request validation and response formatting

### Key Design Patterns

- **Interface Declaration at Consumer Site**: Interfaces are declared where they are used, not where they are implemented. For example, `ArticleRepository` interface is defined in `article/service.go`, not in the repository package.

- **Internal Package**: The `internal/` directory prevents implementation details (repositories, REST handlers) from being imported by external projects.

- **Mocks via mockery**: Use `//go:generate mockery --name InterfaceName` directive and run `make go-generate` to regenerate mocks.

### Dependency Flow

```
main.go → creates concrete repositories → injects into Service → injects into Handler
```

The `app/main.go` is the composition root that wires all dependencies together.

## Environment Configuration

Copy `example.env` to `.env` before running. Required variables:

- `DATABASE_HOST`, `DATABASE_PORT`, `DATABASE_USER`, `DATABASE_PASS`, `DATABASE_NAME` - MySQL connection
- `SERVER_ADDRESS` - HTTP server address (default `:9090`)
- `CONTEXT_TIMEOUT` - Request timeout in seconds

## このプロジェクトの目的

Go 言語と Go 言語によるバックエンド開発の学習のために用意しました。
`complete`フォルダ内の内容を`practice`に写経することで、Go 言語によるクリーンアーキテクチャを理解指定です。
Go 言語を使用した、API 開発を控えており、それに向けたスキルアップを目的とします。

## 学習者の情報

- クリーンアーキテクチャの基礎は概ね理解している
- Go 言語の基本的な文法や機能を理解している（Udemy、Youtube で学習）
- このコードを見ても半分も理解できない程度
