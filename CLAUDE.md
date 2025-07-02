# CLAUDE.md

## 設計

### モデル
<!-- TypeScriptで厳密に型定義したコアデータ構造と関係性 -->
```typescript
// 例: ユーザーモデル
interface User {
  id: string;
  name: string;
  email: string;
  role: 'admin' | 'user' | 'guest';
  preferences: UserPreferences;
  createdAt: Date;
  updatedAt: Date;
}

// 例: ビジネスドメインモデル
interface Project {
  id: string;
  name: string;
  description: string;
  ownerId: string;
  members: string[];
  status: 'active' | 'archived' | 'draft';
  priority: 'high' | 'medium' | 'low';
  createdAt: Date;
  updatedAt: Date;
}

// 例: APIレスポンス型
interface ApiResponse<T> {
  data: T;
  status: 'success' | 'error';
  message?: string;
  meta?: {
    total: number;
    page: number;
    limit: number;
  };
}
```

#### データアーキテクチャ
- データモデル設計: [ER図、正規化レベル]
- データフロー: [入力→処理→出力の流れ]
- 状態管理: [クライアント側状態管理戦略]
- キャッシュ戦略: [Redis、CDN、ブラウザキャッシュ]

### アプリケーション
<!-- 自然言語での機能仕様とビジネス要件 -->

#### ビジネス要件
- ターゲットユーザー: [ペルソナ、ユーザー層]
- ビジネス価値: [解決する課題、提供価値]
- 成功指標: [KPI、メトリクス]
- 競合分析: [差別化要因]

#### 主要機能
- [機能1]: [詳細な要件、受け入れ条件]
- [機能2]: [詳細な要件、受け入れ条件]
- [機能3]: [詳細な要件、受け入れ条件]

#### ユーザー体験
- ユーザーフロー: [主要な操作経路]
- UI/UXデザイン原則: [デザインシステム、ブランドガイドライン]
- アクセシビリティ: [WCAG準拠レベル、支援技術対応]
- レスポンシブデザイン: [ブレークポイント、デバイス対応]
- 国際化: [多言語対応、ロケール設定]

#### 機能要件
- バリデーションルール: [入力検証、ビジネスルール]
- 権限・ロール: [ユーザー権限、アクセス制御]
- 通知・アラート: [通知方法、配信タイミング]
- 検索・フィルタ: [検索機能、ソート・絞り込み]

#### 非機能要件
- パフォーマンス: [レスポンス時間、スループット]
- スケーラビリティ: [同時接続数、トラフィック増加対応]
- 可用性: [稼働率、ダウンタイム許容]
- 信頼性: [データ整合性、障害回復]

### システム

#### アーキテクチャ設計
- システムアーキテクチャ: [モノリス/マイクロサービス/サーバーレス]
- アプリケーションアーキテクチャ: [MVC、Clean Architecture、DDD]
- API設計: [REST/GraphQL/gRPC、バージョニング戦略]
- データベースアーキテクチャ: [RDBMS、NoSQL、レプリケーション]

#### 技術スタック
- プログラミング言語: [言語、バージョン]
- フレームワーク: [Webフレームワーク、ライブラリ]
- データベース: [RDBMS、NoSQL、検索エンジン]
- インフラ: [クラウドプロバイダー、コンテナ化]
- 認証: [認証プロバイダー、セッション管理]

#### 開発環境
```bash
# 環境構築
[SETUP_COMMANDS]

# 開発サーバー起動
[DEV_COMMAND]

# テスト実行
[TEST_COMMAND]

# コード品質チェック
[LINT_COMMAND]
[TYPECHECK_COMMAND]

# ビルド・デプロイ
[BUILD_COMMAND]
[DEPLOY_COMMAND]
```

#### 開発プロセス
- 開発手法: [アジャイル、スクラム、カンバン]
- バージョン管理: [Git Flow、GitHub Flow、ブランチ戦略]
- CI/CD: [パイプライン設計、自動化範囲]
- テスト戦略: [単体、統合、E2E、パフォーマンステスト]
- コードレビュー: [レビュープロセス、品質基準]

#### データ管理
```sql
-- データベース設計例
CREATE TABLE users (
  id UUID PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  role VARCHAR(50) NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  deleted_at TIMESTAMP NULL
);

-- インデックス設計
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_role ON users(role);
```

- データ保護: [暗号化、プライバシー保護]
- バックアップ: [バックアップ戦略、復旧手順]
- データ保持: [ライフサイクル管理、削除ポリシー]

#### セキュリティ
- 認証・認可: [認証方式、権限管理]
- データセキュリティ: [暗号化、秘密情報管理]
- 通信セキュリティ: [HTTPS、API保護]
- セキュリティテスト: [脆弱性診断、ペネトレーションテスト]
- コンプライアンス: [GDPR、SOC2、業界標準]

#### 運用・監視
- ログ管理: [構造化ログ、ログレベル、保存期間]
- 監視・アラート: [メトリクス、閾値、通知方法]
- パフォーマンス監視: [APM、ボトルネック検出]
- エラーハンドリング: [例外処理、エラー報告]
- 障害対応: [インシデント対応、復旧手順]

#### 環境設定
```bash
# 必須環境変数
DATABASE_URL=postgresql://user:pass@localhost:5432/dbname
API_KEY=your_api_key_here
JWT_SECRET=your_jwt_secret_here

# 機能フラグ
ENABLE_FEATURE_X=true
ENABLE_ANALYTICS=false

# 外部サービス連携
SMTP_SERVER=smtp.example.com
REDIS_URL=redis://localhost:6379
```

#### デプロイ・運用
- 環境管理: [Dev、Staging、Production]
- デプロイ戦略: [Blue-Green、Canary、Rolling]
- スケーリング: [水平・垂直スケーリング戦略]
- 災害復旧: [BCP、RTO、RPO]
- コスト最適化: [リソース管理、コスト監視]

#### 品質保証
- テスト自動化: [テストカバレッジ、回帰テスト]
- 品質メトリクス: [コード品質、バグ率]
- パフォーマンステスト: [負荷テスト、ストレステスト]
- セキュリティテスト: [自動化、定期実行]

#### プロジェクト管理
- チーム構成: [役割、責任、スキル要件]
- タイムライン: [マイルストーン、リリース計画]
- リスク管理: [リスク評価、対策]
- ステークホルダー: [関係者、コミュニケーション計画]