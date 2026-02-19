# 📚 Article Search App

記事を登録・検索・RSS配信するWebアプリケーション。
AI（Cohere, Dify）を活用したベクトル検索とRAG質問機能を搭載。

## ✨ 機能

- **📝 記事登録**: URLを入力するとAIが自動で要約・タグ付け
- **📋 記事一覧**: 登録した記事をリスト表示
- **🔍 ベクトル検索**: 類似記事をセマンティック検索
- **💬 RAG質問**: 記事をもとにAIが質問に回答
- **📡 RSS配信**: 登録した記事をRSSフィードとして公開

## 🛠 技術スタック

- **フレームワーク**: [Hono](https://hono.dev/)
- **ホスティング**: [Cloudflare Pages](https://pages.cloudflare.com/)
- **ベクトルDB**: [Qdrant](https://qdrant.tech/)
- **Embedding**: [Cohere](https://cohere.ai/) (embed-multilingual-v3.0)
- **RAG**: [Dify](https://dify.ai/)
- **ワークフロー**: [n8n](https://n8n.io/)

## 📋 アーキテクチャ

```
[ブラウザ] → [Cloudflare Pages (Hono)]
                    ↓
    ┌───────────────┼───────────────┐
    ↓               ↓               ↓
[n8n Webhook]   [Qdrant]       [Dify RAG]
(記事登録)      (検索/一覧)    (質問応答)
    ↓
[Firecrawl] → [Gemini要約] → [Cohereタグ生成] → [Qdrant保存]
```

### 各サービスの役割と連携方法

| サービス | 役割 | 連携方法 | 主なメリット |
|---------|------|----------|------------|
| **n8n** | ワークフローエンジン、記事登録のトリガー | Webhook URLをn8nに設定 | 外部ツールの統合が容易、視覚的なワークフロー構築 |
| **Qdrant** | ベクトル検索・保存 | API経由でembeddingを保存・検索 | 高速なセマンティック検索が可能、スケーラブルなコスト構造 |
| **Cohere** | Embedding生成 | API経由でテキストをembeddingに変換 | 多言語対応（embed-multilingual-v3.0）、高品質なベクトル |
| **Dify** | RAG（検索拡張質問） | Difyのナレッジベースを活用して回答生成 | 既存ナレッジの統合、柔軟なRAG構築 |

### n8nワークフローの設定

`article-search/n8n-workflow-article-register-v5.json` をn8nにインポートすると以下の5つの処理が自動実行されます：

1. **Firecrawl**: URLをスクレイピングしてHTMLを取得
2. **Gemini**: スクレイピング結果から要約を生成
3. **Gemini（2回目）**: 要約に基づいて自動タグを生成
4. **Cohere**: 本文を多言語embeddingに変換
5. **Qdrant**: embeddingを保存して検索可能に

#### 設定手順

1. [n8nにログイン](https://app.n8n.io/)
2. 左メニューから「Import from File」を選択
3. `article-search/n8n-workflow-article-register-v5.json` を選択
4. インポート後、各ノードのAPIキーやエンドポイントを設定

### Difyの設定

`article-search/dify-chatbot-article-rag.yaml` をn8nから参照して設定：

1. 外部ナレッジベースを有効化する
2. QdrantのRetrieval APIをエンドポイントとして設定
3. Difyのナレッジベースで登録したQdrantをソースに指定

### Dify vs n8nの比較

顧客から「Difyよりn8nがいい」というフィードバックを受けることがあります。個人的な視点から比較すると：

| 項目 | Dify（直接） | n8n（ワークフロー経由） |
|-------|--------------|---------------------|
| コスト | サブスクリプション型 | 固定額（または従量課金） |
| 拡張性 | RAG機能内に限定 | 他のAIサービスも統合可能 |
| ユーザー界面 | Difyのダッシュボード | n8nのビジュアルワークフロー |
| 導入ハードル | Difyアカウント作成 | 既にn8nを使っているなら追加のみ |
| ナレッジ管理 | Dify内で一元管理 | 外部サービスとの連携が必要 |
| 管理のしやすさ | 1つのダッシュボードで完結 | ワークフロー単位の管理 |

**結論**: 純粋なRAG機能だけが必要であればDify、複雑なワークフロー構築や他サービス統合を重視するならn8nが適しています。

## 🚀 セットアップ

### 1. リポジトリのクローン

```bash
git clone https://github.com/yourusername/article-search-app.git
cd article-search-app
```

### 2. 依存関係のインストール

```bash
npm install
# または
pnpm install
```

### 3. 環境変数の設定

```bash
cp .dev.vars.example .dev.vars
# .dev.vars を編集して各種APIキーを設定
```

必要な環境変数:
- `BASIC_USERNAME`, `BASIC_PASSWORD`: Basic認証
- `N8N_WEBHOOK_URL`: n8n記事登録webhook
- `QDRANT_URL`: Qdrant API URL
- `COHERE_API_KEY`: Cohere API Key
- `DIFY_API_URL`, `DIFY_API_KEY`: Dify API設定
- `RSS_TITLE`, `RSS_LINK`: RSSフィード設定

### 4. ローカル開発

```bash
npm run dev
# http://localhost:8787 でアクセス
```

### 5. デプロイ

```bash
npm run deploy
```

## 🗄 デプロイ構成

### Cloudflare Pages + Cloudflare Workers

このアプリは以下の構成でデプロイされます：

```
                ┌─────────────────────────┐
                │  Cloudflare Pages  │  ← 静的アセット
                └────────┬────────────────┘
                         │
                         ↓
                ┌─────────────────────────┐
                │ Cloudflare Workers │  ← APIエンドポイント
                └────────┬────────────────┘
                         │
                         ↓
                ┌─────────────────────────┐
                │  Qdrant (Vector DB)  │
                └─────────────────────────┘
```

- **Pages**: Honoアプリをホスティング
- **Workers**: APIエンドポイント（`/api/*`）を提供
- **D1 Database**: ローカル開発時は使用（本番はD1またはD1）

#### デプロイコマンド

```bash
# Cloudflare Pagesにデプロイ
npm run deploy

# 環境変数はCloudflareダッシュボードで設定
```

#### リダイレクト設定

Cloudflare Accessで以下のリダイレクトを設定：

| パス | 転送先 | 説明 |
|------|--------|------|
| `/` → `/` | SPA対応 |
| `/feed.xml` → `/feed.xml` | RSSフィード |

## 🔗 外部サービスの設定

### n8n ワークフロー

`article-search/n8n-workflow-article-register-v5.json` をインポート:
1. Firecrawlでスクレイピング
2. Geminiで要約生成
3. Geminiでタグ自動生成
4. Cohereでembedding生成
5. Qdrantに保存

### Dify チャットボット

`article-search/dify-chatbot-article-rag.yaml` を参考に設定:
1. 外部ナレッジベースを有効化
2. retrieval APIエンドポイントを設定

## 🔒 認証設定

### Cloudflare Access（推奨）

1. Cloudflare Zero Trust > Access > Applications
2. 新しいアプリケーションを追加
3. パス `/feed.xml` をBypass（公開）
4. その他はメール認証などを設定

## 📱 API仕様

### POST /api/articles
記事を登録（n8n経由）

```json
{
  "url": "https://example.com/article",
  "memo": "メモ",
  "publish_rss": true
}
```

### GET /api/articles
記事一覧を取得

### POST /api/search
ベクトル検索

```json
{
  "query": "検索キーワード",
  "limit": 10
}
```

### POST /api/ask
RAG質問（Dify経由）

```json
{
  "question": "A2Aプロトコルについて教えて"
}
```

### GET /feed.xml
RSSフィード（認証不要）

## 📝 ライセンス

MIT
