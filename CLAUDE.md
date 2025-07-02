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
  createdAt: Date;
  updatedAt: Date;
}

// 例: プロジェクトモデル
interface Project {
  id: string;
  name: string;
  description: string;
  ownerId: string;
  members: string[];
  status: 'active' | 'archived' | 'draft';
  createdAt: Date;
  updatedAt: Date;
}
```

### アプリケーション
<!-- 自然言語での機能仕様 -->

#### 主要機能
- [機能1の説明]
- [機能2の説明]
- [機能3の説明]

#### ユーザーフロー
- [フロー1]: [詳細な手順]
- [フロー2]: [詳細な手順]

#### バリデーションルール
- [フィールド名]: [検証ルール]
- [フィールド名]: [検証ルール]

#### 画面設計考慮事項
- [UI/UX要件]
- [レスポンシブ対応]
- [アクセシビリティ要件]

### システム

#### 開発環境
```bash
# セットアップ
[SETUP_COMMANDS]

# 開発サーバー起動
[DEV_COMMAND]

# テスト実行
[TEST_COMMAND]

# ビルド
[BUILD_COMMAND]
```

#### 技術スタック
- フレームワーク: [FRAMEWORK]
- データベース: [DATABASE]
- 認証: [AUTH_METHOD]
- スタイリング: [STYLING_APPROACH]

#### データベース設定
```sql
-- 主要テーブル構造の例
CREATE TABLE users (
  id UUID PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);
```

#### 認証・認可
- 認証方式: [AUTH_MECHANISM]
- セッション管理: [SESSION_MANAGEMENT]
- 権限管理: [PERMISSION_SYSTEM]

#### 開発ガイドライン
- コーディング規約: [既存コードのパターンに従う]
- ファイル管理: [新規作成は最小限、編集を優先]
- セキュリティ: [秘密情報の取り扱い、入力検証]
- エラーハンドリング: [統一的なエラー処理方針]
- ログ出力: [ログレベルと出力形式]

#### 環境変数
```bash
# 必要な環境変数
DATABASE_URL=
API_KEY=
JWT_SECRET=
```

#### デプロイ
- 環境: [DEPLOYMENT_ENVIRONMENT]
- CI/CD: [DEPLOYMENT_PIPELINE]
- モニタリング: [MONITORING_TOOLS]