# Go Clean Architecture 学習計画

この計画は、クリーンアーキテクチャの原則に従い、**依存関係の内側から外側**へ向かって段階的に学習していきます。

---

## 全体構成図

```
┌─────────────────────────────────────────────────────────────┐
│                    app/main.go (統合)                        │
├─────────────────────────────────────────────────────────────┤
│  internal/rest/          │  internal/repository/mysql/      │
│  (Delivery Layer)        │  (Repository Layer)              │
├─────────────────────────────────────────────────────────────┤
│                    article/service.go                        │
│                    (Service Layer)                           │
├─────────────────────────────────────────────────────────────┤
│                    domain/                                   │
│                    (Domain Layer - 最も内側)                 │
└─────────────────────────────────────────────────────────────┘
```

---

## Phase 1: Domain層（基礎・依存なし）

**目的**: ビジネスエンティティとドメインエラーを理解する

### Step 1.1: Author エンティティ
**ファイル**: `domain/author.go`
**行数**: 9行
**難易度**: ★☆☆☆☆

**学習ポイント**:
- Go の struct 定義
- JSON タグ（`json:"field_name"`）
- パッケージの命名

```go
// 写経するコード
package domain

type Author struct {
    ID        int64  `json:"id"`
    Name      string `json:"name"`
    CreatedAt string `json:"created_at"`
    UpdatedAt string `json:"updated_at"`
}
```

**確認コマンド**:
```bash
cd practice
go build ./domain/
```

---

### Step 1.2: ドメインエラー
**ファイル**: `domain/errors.go`
**行数**: 14行
**難易度**: ★☆☆☆☆

**学習ポイント**:
- `errors.New()` によるエラー定義
- パッケージレベル変数（`var`）
- エラーの命名規則（`Err` プレフィックス）

**確認コマンド**:
```bash
go build ./domain/
```

---

### Step 1.3: Article エンティティ
**ファイル**: `domain/article.go`
**行数**: 15行
**難易度**: ★★☆☆☆

**学習ポイント**:
- `time.Time` 型の使用
- 構造体の埋め込み（`Author` フィールド）
- バリデーションタグ（`validate:"required"`）

**確認コマンド**:
```bash
go build ./domain/
```

---

## Phase 2: Repository ヘルパー

**目的**: ユーティリティ関数のパターンを学ぶ

### Step 2.1: カーソルエンコード/デコード
**ファイル**: `internal/repository/helper.go`
**行数**: 30行
**難易度**: ★★☆☆☆

**学習ポイント**:
- `encoding/base64` パッケージ
- `time.Time` のフォーマットとパース
- 定数の定義（`const`）
- エラーハンドリングの基本

**確認コマンド**:
```bash
go build ./internal/repository/
```

---

## Phase 3: Repository層（データアクセス）

**目的**: データベース操作のパターンを学ぶ

### Step 3.1: Author Repository
**ファイル**: `internal/repository/mysql/author.go`
**行数**: 41行
**難易度**: ★★★☆☆

**学習ポイント**:
- `database/sql` パッケージの基本
- `PrepareContext` と `QueryRowContext`
- `Scan` によるデータマッピング
- コンストラクタパターン（`NewAuthorRepository`）

**確認コマンド**:
```bash
go build ./internal/repository/mysql/
```

---

### Step 3.2: Article Repository
**ファイル**: `internal/repository/mysql/article.go`
**行数**: 184行
**難易度**: ★★★★☆

**注意**: このファイルは長いので、メソッド単位で写経することをお勧めします。

#### Step 3.2.1: 構造体とコンストラクタ（1-21行）
- 構造体定義
- `NewArticleRepository`

#### Step 3.2.2: fetch メソッド（23-61行）
- `QueryContext` の使用
- `defer` と `rows.Close()`
- ループ処理とスライス操作

#### Step 3.2.3: Fetch メソッド（63-82行）
- カーソルベースのページネーション
- ヘルパー関数の呼び出し

#### Step 3.2.4: GetByID / GetByTitle（83-116行）
- 単一レコード取得パターン
- `domain.ErrNotFound` の使用

#### Step 3.2.5: Store / Delete / Update（118-184行）
- INSERT / DELETE / UPDATE クエリ
- `ExecContext` と `LastInsertId` / `RowsAffected`

**確認コマンド**:
```bash
go build ./internal/repository/mysql/
```

---

## Phase 4: Service層（ビジネスロジック）

**目的**: ビジネスロジックとインターフェース定義を学ぶ

### Step 4.1: Article Service
**ファイル**: `article/service.go`
**行数**: 167行
**難易度**: ★★★★★

**注意**: このファイルはプロジェクトの核心です。セクション単位で写経してください。

#### Step 4.1.1: インターフェース定義（1-30行）
**学習ポイント**:
- インターフェースは利用側で定義する（Dependency Inversion）
- `//go:generate mockery` ディレクティブ
- `context.Context` を第一引数にする慣例

#### Step 4.1.2: Service 構造体とコンストラクタ（32-43行）
**学習ポイント**:
- 依存性注入パターン
- プライベートフィールド（小文字開始）

#### Step 4.1.3: fillAuthorDetails メソッド（45-99行）
**学習ポイント**:
- `errgroup` による並行処理
- チャネル（`chan`）の使用
- ゴルーチン（`go func()`）
- マップの使用

#### Step 4.1.4: CRUD メソッド群（101-167行）
**学習ポイント**:
- シンプルなメソッド実装
- ドメインエラーの使用
- 時刻の設定（`time.Now()`）

**確認コマンド**:
```bash
go build ./article/
```

---

## Phase 5: Middleware（HTTP ミドルウェア）

**目的**: Echo フレームワークのミドルウェアパターンを学ぶ

### Step 5.1: CORS Middleware
**ファイル**: `internal/rest/middleware/cors.go`
**行数**: 11行
**難易度**: ★★☆☆☆

**学習ポイント**:
- Echo のミドルウェア関数シグネチャ
- HTTP ヘッダーの設定

---

### Step 5.2: Timeout Middleware
**ファイル**: `internal/rest/middleware/timeout.go`
**行数**: 22行
**難易度**: ★★★☆☆

**学習ポイント**:
- `context.WithTimeout` の使用
- `defer cancel()` パターン
- リクエストコンテキストの更新

**確認コマンド**:
```bash
go build ./internal/rest/middleware/
```

---

## Phase 6: Delivery層（HTTP ハンドラ）

**目的**: REST API ハンドラの実装パターンを学ぶ

### Step 6.1: Article Handler
**ファイル**: `internal/rest/article.go`
**行数**: 154行
**難易度**: ★★★★☆

#### Step 6.1.1: インターフェースと構造体（1-37行）
**学習ポイント**:
- Service インターフェースの定義（利用側）
- ハンドラ構造体

#### Step 6.1.2: NewArticleHandler（39-48行）
**学習ポイント**:
- ルーティング登録
- RESTful エンドポイント設計

#### Step 6.1.3: FetchArticle / GetByID（50-87行）
**学習ポイント**:
- クエリパラメータ取得
- パスパラメータ取得
- JSON レスポンス

#### Step 6.1.4: Store / Delete（89-136行）
**学習ポイント**:
- リクエストボディのバインド
- バリデーション
- HTTP ステータスコード

#### Step 6.1.5: getStatusCode ヘルパー（138-154行）
**学習ポイント**:
- エラーからステータスコードへの変換
- switch 文

**確認コマンド**:
```bash
go build ./internal/rest/
```

---

## Phase 7: 統合（エントリーポイント）

**目的**: 全レイヤーを統合する方法を学ぶ

### Step 7.1: main.go
**ファイル**: `app/main.go`
**行数**: 89行
**難易度**: ★★★★☆

**学習ポイント**:
- 環境変数の読み込み（`godotenv`）
- データベース接続
- 依存性注入の実践（Composition Root）
- Echo サーバーの起動

**確認コマンド**:
```bash
go build ./app/
```

---

## Phase 8: テスト（オプション）

**目的**: テストの書き方を学ぶ

### Step 8.1: Service テスト
**ファイル**: `article/service_test.go`

**学習ポイント**:
- テーブル駆動テスト
- mockery によるモック生成
- testify/assert の使用

### Step 8.2: Repository テスト
**ファイル**: `internal/repository/mysql/article_test.go`

**学習ポイント**:
- sqlmock によるDB モック
- データベーステストパターン

### Step 8.3: Handler テスト
**ファイル**: `internal/rest/article_test.go`

**学習ポイント**:
- Echo のテストユーティリティ
- HTTP リクエスト/レスポンスのモック

---

## 推奨スケジュール

| 日 | Phase | 所要時間目安 |
|---|-------|------------|
| 1 | Phase 1 (Domain) | 1-2時間 |
| 2 | Phase 2-3 (Repository) | 2-3時間 |
| 3 | Phase 4 (Service) | 2-3時間 |
| 4 | Phase 5-6 (Middleware + Handler) | 2-3時間 |
| 5 | Phase 7 (Integration) | 1-2時間 |
| 6+ | Phase 8 (Tests) | 各テスト1-2時間 |

---

## 動作確認方法

各 Phase 完了後、以下のコマンドで確認:

```bash
cd practice

# ビルド確認
go build ./...

# リンター確認（Phase 7完了後）
# golangci-lint run ./...

# テスト実行（Phase 8完了後）
# go test ./...
```

---

## 重要な注意点

1. **go.mod の作成が必要**
   ```bash
   cd practice
   go mod init github.com/bxcodec/go-clean-arch
   go mod tidy
   ```

2. **ディレクトリ構造を先に作成**
   ```bash
   mkdir -p domain
   mkdir -p article
   mkdir -p internal/repository/mysql
   mkdir -p internal/rest/middleware
   mkdir -p app
   ```

3. **依存パッケージのインストール**
   ```bash
   go get github.com/labstack/echo/v4
   go get github.com/sirupsen/logrus
   go get golang.org/x/sync
   go get github.com/go-sql-driver/mysql
   go get github.com/joho/godotenv
   go get gopkg.in/go-playground/validator.v9
   ```

---

## 学習のコツ

1. **写経中にわからない箇所があったら**:
   - まず書き写す
   - `go build` でエラーがないか確認
   - エラーがあれば修正
   - わからない箇所をメモして後で調べる

2. **各ファイル完了後**:
   - なぜこの設計なのか考える
   - 依存関係の方向を確認する

3. **Phase 4 は特に重要**:
   - インターフェースが利用側で定義される理由を理解する
   - これがクリーンアーキテクチャの核心
