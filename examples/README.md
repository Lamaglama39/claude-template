# Claude Code テンプレート例集

代表的なアプリケーション向けにカスタマイズしたCLAUDE.mdの実例集です。

## 例一覧

### Webアプリケーション

#### 1. [ecommerce-nextjs](./ecommerce-nextjs/)
**ECサイト（Next.js + TypeScript）**
- Next.js 14 (App Router) + TypeScript
- Stripe決済、在庫管理、SEO対応
- PostgreSQL + Prisma、Tailwind CSS
- 中小企業向けオンラインショップを想定

#### 2. [saas-dashboard-react](./saas-dashboard-react/)
**SaaS業務ツール（React + Node.js）**
- React 18 + TypeScript + Node.js
- マルチテナント、権限管理、データ分析
- PostgreSQL、Redis、WebSocket
- 企業向けダッシュボードツールを想定

### モバイルアプリケーション

#### 3. [social-app-react-native](./social-app-react-native/)
**ソーシャルメディアアプリ（React Native）**
- React Native + TypeScript
- リアルタイム通信、メディア配信、プッシュ通知
- Firebase、WebRTC、Redux Toolkit
- 若年層向けSNSアプリを想定

#### 4. [fintech-app-react-native](./fintech-app-react-native/)
**フィンテックアプリ（React Native）**
- React Native + TypeScript
- 高セキュリティ、生体認証、暗号化通信
- 金融API連携、リアルタイム取引
- 個人向け投資・決済アプリを想定

### バックエンド・API

#### 5. [api-backend-nodejs](./api-backend-nodejs/)
**REST API（Node.js + Express）**
- Node.js + TypeScript + Express
- RESTful API設計、JWT認証、レート制限
- PostgreSQL、Redis、Docker
- マイクロサービス対応のAPIサーバーを想定

### デスクトップアプリケーション

#### 6. [dev-tool-electron](./dev-tool-electron/)
**開発者ツール（Electron + React）**
- Electron + React + TypeScript
- ファイルシステム連携、プラグインアーキテクチャ
- 自動更新、クロスプラットフォーム対応
- コードエディタ・統合開発環境を想定

## 使用方法

1. 目的に近い例を選択
2. 該当フォルダの`CLAUDE.md`をプロジェクトルートにコピー
3. プロジェクト固有の要件に合わせて修正
4. 必要に応じて他の例も参考にしてカスタマイズ

## カスタマイズポイント

各例では以下の観点で深く検討されています：

### ドメイン特有の設計
- **データモデル**: そのアプリケーション領域に特化したエンティティ設計
- **ビジネスロジック**: ドメイン固有の処理とルール
- **ワークフロー**: 典型的なユーザーフローと業務プロセス

### 技術選定の根拠
- **アーキテクチャパターン**: 要件に最適化された設計パターン
- **技術スタック**: 実証済みの技術組み合わせ
- **スケーラビリティ**: 成長に対応できる拡張戦略

### 実装上の考慮事項
- **セキュリティ**: アプリケーション種別に応じたセキュリティ対策
- **パフォーマンス**: 想定規模に最適化された性能設計
- **運用**: 実際の本番運用を考慮した監視・保守設計

## 新しい例の作成

新しいアプリケーション種別の例を作成する場合の手順：

1. **ドメイン調査**: 対象領域の典型的な要件と課題を調査
2. **技術選定**: 要件に最適な技術スタックを選定
3. **アーキテクチャ設計**: スケーラブルで保守性の高い設計を検討
4. **テンプレート作成**: ベーステンプレートから該当分野に特化
5. **検証**: 実際のプロジェクトで使用して品質を確認

## 貢献

新しい例やexistingな例の改善について：
1. Issue作成で提案内容を説明
2. Pull Request作成で実装を提出
3. レビュー後にマージ

各例は実際の開発現場で使用されることを想定して作成されています。