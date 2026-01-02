# Go Clean Architecture プロジェクト詳細

このドキュメントは `tasks/plan.md` を進める際の参考資料です。

---

## 1. プロジェクト概要

**目的**: 記事（Article）の CRUD 操作を提供する REST API

**技術スタック**:
| カテゴリ | 技術 |
|---------|------|
| 言語 | Go 1.20 |
| Web フレームワーク | Echo v4 |
| データベース | MySQL |
| テスト | testify, sqlmock, mockery |
| ログ | logrus |
| 設定 | godotenv |
| バリデーション | validator.v9 |

---

## 2. API エンドポイント

| メソッド | エンドポイント | 機能 |
|---------|---------------|------|
| `GET` | `/articles` | 記事一覧取得（ページネーション対応） |
| `GET` | `/articles/:id` | 記事詳細取得 |
| `POST` | `/articles` | 記事作成 |
| `DELETE` | `/articles/:id` | 記事削除 |

---

## 3. データモデル

### Article（記事）
```go
type Article struct {
    ID        int64     // 記事ID
    Title     string    // タイトル（必須、バリデーション対象）
    Content   string    // 本文（必須、バリデーション対象）
    Author    Author    // 著者情報（関連エンティティ）
    UpdatedAt time.Time // 更新日時
    CreatedAt time.Time // 作成日時
}
```

### Author（著者）
```go
type Author struct {
    ID        int64  // 著者ID
    Name      string // 著者名
    CreatedAt string // 作成日時
    UpdatedAt string // 更新日時
}
```

---

## 4. アーキテクチャ詳細

### 4.1 レイヤー構成と依存の方向

```
┌─────────────────────────────────────────────────────────────────────┐
│                         app/main.go                                 │
│                    （Composition Root）                              │
│           全ての依存関係をここで組み立て、注入する                      │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│   internal/rest/                 │   internal/repository/mysql/     │
│   ─────────────────              │   ───────────────────────────    │
│   ArticleHandler                 │   ArticleRepository              │
│   ├─ FetchArticle()              │   ├─ Fetch()                     │
│   ├─ GetByID()                   │   ├─ GetByID()                   │
│   ├─ Store()                     │   ├─ GetByTitle()                │
│   └─ Delete()                    │   ├─ Store()                     │
│                                  │   ├─ Delete()                    │
│   ArticleService (interface)     │   └─ Update()                    │
│   └─ Handler が使う Service の    │                                  │
│      インターフェースを定義        │   AuthorRepository               │
│                                  │   └─ GetByID()                   │
└──────────────────────────────────┴───────────────────────────────────┘
                    │                              │
                    │  依存（インターフェース経由）   │
                    ▼                              ▼
┌─────────────────────────────────────────────────────────────────────┐
│                        article/service.go                           │
│                        ──────────────────                           │
│   Service struct                                                    │
│   ├─ Fetch()           記事一覧取得 + 著者情報補完                    │
│   ├─ GetByID()         ID指定で記事取得                              │
│   ├─ GetByTitle()      タイトル指定で記事取得                         │
│   ├─ Store()           新規記事保存（重複チェック付き）                │
│   ├─ Update()          記事更新                                     │
│   └─ Delete()          記事削除                                     │
│                                                                     │
│   ArticleRepository (interface)  ← Repository が実装すべき契約       │
│   AuthorRepository (interface)   ← Repository が実装すべき契約       │
└─────────────────────────────────────────────────────────────────────┘
                                    │
                                    │  依存（import）
                                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                           domain/                                   │
│                           ───────                                   │
│   Article struct         コアエンティティ（ビジネスオブジェクト）       │
│   Author struct          コアエンティティ                            │
│                                                                     │
│   ErrNotFound            「見つからない」エラー                       │
│   ErrConflict            「競合（重複）」エラー                        │
│   ErrInternalServerError 「サーバーエラー」                           │
│   ErrBadParamInput       「不正な入力」エラー                         │
└─────────────────────────────────────────────────────────────────────┘
```

### 4.2 依存性逆転の原則（DIP）

**重要な設計パターン**: インターフェースは「利用する側」で定義する

```
┌─────────────────┐         ┌─────────────────┐
│ article/        │         │ internal/       │
│ service.go      │         │ repository/     │
│                 │         │ mysql/          │
│ ArticleRepository ◀─────── ArticleRepository │
│ (interface)     │  実装   │ (struct)        │
└─────────────────┘         └─────────────────┘
   利用側で定義                 実装側
```

これにより：
- Service 層は Repository 層の実装詳細を知らない
- Repository の実装を差し替え可能（テスト時にモック化）
- 依存の方向が常に「内側」を向く

---

## 5. ファイル一覧と役割

### Domain 層（`domain/`）

| ファイル | 役割 | 主要シンボル |
|---------|------|-------------|
| `article.go` | 記事エンティティ | `Article` struct |
| `author.go` | 著者エンティティ | `Author` struct |
| `errors.go` | ドメインエラー定義 | `ErrNotFound`, `ErrConflict`, `ErrInternalServerError`, `ErrBadParamInput` |

### Service 層（`article/`）

| ファイル | 役割 | 主要シンボル |
|---------|------|-------------|
| `service.go` | ビジネスロジック | `Service` struct, `ArticleRepository` interface, `AuthorRepository` interface, `NewService()`, `Fetch()`, `GetByID()`, `Store()`, `Delete()`, `Update()` |

### Repository 層（`internal/repository/`）

| ファイル | 役割 | 主要シンボル |
|---------|------|-------------|
| `helper.go` | カーソルエンコード/デコード | `EncodeCursor()`, `DecodeCursor()` |
| `mysql/article.go` | 記事のDB操作 | `ArticleRepository` struct, `NewArticleRepository()`, `Fetch()`, `GetByID()`, `GetByTitle()`, `Store()`, `Delete()`, `Update()` |
| `mysql/author.go` | 著者のDB操作 | `AuthorRepository` struct, `NewAuthorRepository()`, `GetByID()` |

### Delivery 層（`internal/rest/`）

| ファイル | 役割 | 主要シンボル |
|---------|------|-------------|
| `article.go` | HTTPハンドラ | `ArticleHandler` struct, `ArticleService` interface, `NewArticleHandler()`, `FetchArticle()`, `GetByID()`, `Store()`, `Delete()` |
| `middleware/cors.go` | CORS設定 | `CORS()` |
| `middleware/timeout.go` | タイムアウト設定 | `SetRequestContextWithTimeout()` |

### エントリーポイント（`app/`）

| ファイル | 役割 | 主要シンボル |
|---------|------|-------------|
| `main.go` | アプリケーション起動 | `main()`, `init()` |

---

## 6. 主要な処理フロー

### 6.1 記事一覧取得（`GET /articles`）

```
1. ArticleHandler.FetchArticle()
   ├─ クエリパラメータ取得（num, cursor）
   └─ Service.Fetch() 呼び出し
       ├─ ArticleRepository.Fetch() でDB取得
       ├─ fillAuthorDetails() で著者情報を並行取得
       │   └─ errgroup + goroutine で並列処理
       └─ 結果を返却
```

### 6.2 記事作成（`POST /articles`）

```
1. ArticleHandler.Store()
   ├─ リクエストボディをバインド
   ├─ バリデーション（isRequestValid）
   └─ Service.Store() 呼び出し
       ├─ ArticleRepository.GetByTitle() で重複チェック
       │   └─ 存在すれば ErrConflict を返す
       └─ ArticleRepository.Store() でDB保存
```

### 6.3 記事削除（`DELETE /articles/:id`）

```
1. ArticleHandler.Delete()
   ├─ パスパラメータからID取得
   └─ Service.Delete() 呼び出し
       ├─ ArticleRepository.GetByID() で存在確認
       │   └─ 存在しなければ ErrNotFound を返す
       └─ ArticleRepository.Delete() でDB削除
```

---

## 7. 重要な Go パターン

### 7.1 コンストラクタパターン

```go
func NewService(ar ArticleRepository, author AuthorRepository) *Service {
    return &Service{
        articleRepo: ar,
        authorRepo:  author,
    }
}
```

### 7.2 context.Context の使用

```go
func (s *Service) Fetch(ctx context.Context, cursor string, num int64) ([]domain.Article, string, error) {
    // ctx は全てのメソッドの第一引数
}
```

### 7.3 errgroup による並行処理

```go
g, ctx := errgroup.WithContext(ctx)
for _, article := range articles {
    g.Go(func() error {
        // 並行処理
    })
}
if err := g.Wait(); err != nil {
    return err
}
```

### 7.4 defer によるリソース解放

```go
rows, err := r.Conn.QueryContext(ctx, query, args...)
if err != nil {
    return nil, err
}
defer rows.Close()  // 関数終了時に必ず実行
```

---

## 8. エラーハンドリング

### ドメインエラーと HTTP ステータスコードのマッピング

```go
func getStatusCode(err error) int {
    switch {
    case errors.Is(err, domain.ErrNotFound):
        return http.StatusNotFound           // 404
    case errors.Is(err, domain.ErrConflict):
        return http.StatusConflict           // 409
    case errors.Is(err, domain.ErrBadParamInput):
        return http.StatusBadRequest         // 400
    default:
        return http.StatusInternalServerError // 500
    }
}
```

---

## 9. テスト構成

| テストファイル | テスト対象 | 手法 |
|---------------|----------|------|
| `article/service_test.go` | Service 層 | mockery でモック生成、テーブル駆動テスト |
| `internal/repository/mysql/article_test.go` | Repository 層 | sqlmock でDB操作をモック |
| `internal/rest/article_test.go` | Handler 層 | Echo のテストユーティリティ |

---

## 10. 学習時の注意点

1. **import 文の確認**: 各ファイルの import を見て、何に依存しているか確認する
2. **インターフェースの場所**: どのパッケージでインターフェースが定義されているか注目
3. **エラーの伝播**: エラーがどのように上位レイヤーに伝わるか追跡
4. **context の流れ**: context がどのように渡されているか確認

---

## 11. データベーススキーマ

`article.sql` に定義されています：

```sql
-- articles テーブル
CREATE TABLE articles (
    id         INT PRIMARY KEY AUTO_INCREMENT,
    title      VARCHAR(255) NOT NULL,
    content    TEXT NOT NULL,
    author_id  INT NOT NULL,
    updated_at DATETIME,
    created_at DATETIME
);

-- authors テーブル（推定）
CREATE TABLE authors (
    id         INT PRIMARY KEY AUTO_INCREMENT,
    name       VARCHAR(255) NOT NULL,
    created_at DATETIME,
    updated_at DATETIME
);
```

---

## 12. クイックリファレンス

### よく使うコマンド

```bash
# 開発環境起動
cd complete && make up

# ビルド確認
cd complete && go build ./...

# テスト実行
cd complete && make tests

# リンター実行
cd complete && make lint

# モック再生成
cd complete && make go-generate
```

### 環境変数（.env）

```env
DATABASE_HOST=localhost
DATABASE_PORT=3306
DATABASE_USER=root
DATABASE_PASS=password
DATABASE_NAME=article
SERVER_ADDRESS=:9090
CONTEXT_TIMEOUT=30
```
