# CLAUDE.md - SaaS業務ツール（プロジェクト管理・チームコラボレーション）

## 設計

### モデル
<!-- TypeScriptで厳密に型定義したコアデータ構造と関係性 -->
```typescript
// マルチテナント・組織管理
interface Organization {
  id: string;
  name: string;
  slug: string;
  description?: string;
  website?: string;
  industry?: string;
  size: 'startup' | 'small' | 'medium' | 'large' | 'enterprise';
  plan: 'free' | 'starter' | 'professional' | 'enterprise';
  billingInfo: BillingInfo;
  settings: OrganizationSettings;
  features: OrganizationFeatures;
  members: OrganizationMember[];
  createdAt: Date;
  updatedAt: Date;
}

interface OrganizationSettings {
  timezone: string;
  dateFormat: 'MM/DD/YYYY' | 'DD/MM/YYYY' | 'YYYY-MM-DD';
  workingDays: number[]; // [1,2,3,4,5] for Mon-Fri
  workingHours: {
    start: string; // "09:00"
    end: string;   // "17:00"
  };
  notifications: {
    email: boolean;
    slack: boolean;
    inApp: boolean;
  };
  security: {
    ssoEnabled: boolean;
    twoFactorRequired: boolean;
    passwordPolicy: PasswordPolicy;
    sessionTimeout: number; // minutes
  };
}

interface OrganizationFeatures {
  maxUsers: number;
  maxProjects: number;
  storageLimit: number; // GB
  apiCalls: number; // per month
  advancedReporting: boolean;
  customFields: boolean;
  automation: boolean;
  timeTracking: boolean;
  ganttCharts: boolean;
  portfolioView: boolean;
}

interface OrganizationMember {
  userId: string;
  role: 'owner' | 'admin' | 'member' | 'guest';
  permissions: Permission[];
  joinedAt: Date;
  invitedBy: string;
  status: 'active' | 'invited' | 'suspended';
}

// ユーザー・認証
interface User {
  id: string;
  email: string;
  firstName: string;
  lastName: string;
  avatar?: string;
  title?: string;
  department?: string;
  timezone: string;
  locale: string;
  preferences: UserPreferences;
  lastLoginAt?: Date;
  emailVerified: boolean;
  twoFactorEnabled: boolean;
  ssoProvider?: 'google' | 'microsoft' | 'okta' | 'auth0';
  status: 'active' | 'inactive' | 'suspended';
  createdAt: Date;
  updatedAt: Date;
}

interface UserPreferences {
  theme: 'light' | 'dark' | 'auto';
  notifications: {
    email: NotificationSettings;
    browser: NotificationSettings;
    mobile: NotificationSettings;
  };
  dashboard: {
    layout: string;
    widgets: DashboardWidget[];
  };
  defaultProjectView: 'list' | 'board' | 'timeline' | 'calendar';
}

interface NotificationSettings {
  taskAssigned: boolean;
  taskCompleted: boolean;
  projectUpdates: boolean;
  deadlines: boolean;
  mentions: boolean;
  dailyDigest: boolean;
}

// 権限管理
interface Permission {
  resource: 'project' | 'task' | 'team' | 'report' | 'billing' | 'settings';
  actions: ('create' | 'read' | 'update' | 'delete' | 'admin')[];
  conditions?: {
    projectId?: string;
    teamId?: string;
    ownOnly?: boolean;
  };
}

interface Role {
  id: string;
  name: string;
  description: string;
  organizationId: string;
  permissions: Permission[];
  isDefault: boolean;
  isCustom: boolean;
  userCount: number;
  createdAt: Date;
  updatedAt: Date;
}

// プロジェクト管理
interface Project {
  id: string;
  name: string;
  description?: string;
  organizationId: string;
  key: string; // Project abbreviation (e.g., "PROJ")
  status: 'planning' | 'active' | 'on_hold' | 'completed' | 'archived';
  priority: 'low' | 'medium' | 'high' | 'critical';
  visibility: 'public' | 'private' | 'team_only';
  startDate?: Date;
  endDate?: Date;
  budget?: number;
  progress: number; // 0-100
  health: 'green' | 'yellow' | 'red';
  owner: string; // userId
  team: ProjectMember[];
  tags: string[];
  customFields: CustomField[];
  settings: ProjectSettings;
  metrics: ProjectMetrics;
  createdAt: Date;
  updatedAt: Date;
}

interface ProjectMember {
  userId: string;
  role: 'owner' | 'admin' | 'member' | 'viewer';
  joinedAt: Date;
  allocation: number; // percentage 0-100
}

interface ProjectSettings {
  allowExternalGuests: boolean;
  requireApproval: boolean;
  autoAssignTasks: boolean;
  timeTrackingEnabled: boolean;
  notificationSettings: {
    updates: boolean;
    deadlines: boolean;
    mentions: boolean;
  };
}

interface ProjectMetrics {
  totalTasks: number;
  completedTasks: number;
  overdueTasks: number;
  hoursLogged: number;
  budgetSpent: number;
  teamVelocity: number;
  lastUpdated: Date;
}

// タスク管理
interface Task {
  id: string;
  title: string;
  description?: string;
  projectId: string;
  assigneeId?: string;
  reporterId: string;
  parentTaskId?: string; // For subtasks
  status: TaskStatus;
  priority: 'low' | 'medium' | 'high' | 'critical';
  labels: string[];
  startDate?: Date;
  dueDate?: Date;
  estimatedHours?: number;
  actualHours: number;
  progress: number; // 0-100
  dependencies: TaskDependency[];
  subtasks: string[]; // Task IDs
  attachments: Attachment[];
  comments: Comment[];
  customFields: CustomField[];
  watchers: string[]; // User IDs
  createdAt: Date;
  updatedAt: Date;
  completedAt?: Date;
}

interface TaskStatus {
  id: string;
  name: string;
  category: 'todo' | 'in_progress' | 'review' | 'done' | 'blocked';
  color: string;
  position: number;
}

interface TaskDependency {
  taskId: string;
  type: 'blocks' | 'blocked_by' | 'relates_to' | 'duplicates';
}

// カスタムフィールド
interface CustomField {
  id: string;
  name: string;
  type: 'text' | 'number' | 'date' | 'select' | 'multiselect' | 'checkbox' | 'url' | 'user';
  value: any;
  options?: string[]; // For select fields
  required: boolean;
}

// ファイル・添付
interface Attachment {
  id: string;
  name: string;
  type: string;
  size: number;
  url: string;
  thumbnailUrl?: string;
  uploadedBy: string;
  uploadedAt: Date;
}

// コメント・アクティビティ
interface Comment {
  id: string;
  content: string;
  authorId: string;
  parentCommentId?: string;
  mentions: string[];
  attachments: Attachment[];
  reactions: Reaction[];
  isEdited: boolean;
  createdAt: Date;
  updatedAt: Date;
}

interface Reaction {
  emoji: string;
  userId: string;
  createdAt: Date;
}

interface Activity {
  id: string;
  type: 'task_created' | 'task_updated' | 'task_completed' | 'comment_added' | 'file_uploaded' | 'member_added';
  entityType: 'task' | 'project' | 'comment';
  entityId: string;
  userId: string;
  organizationId: string;
  description: string;
  metadata: any;
  createdAt: Date;
}

// 時間追跡
interface TimeEntry {
  id: string;
  userId: string;
  taskId?: string;
  projectId: string;
  description?: string;
  startTime: Date;
  endTime?: Date;
  duration: number; // minutes
  billable: boolean;
  rate?: number;
  tags: string[];
  status: 'running' | 'stopped' | 'approved' | 'rejected';
  createdAt: Date;
  updatedAt: Date;
}

// レポート・分析
interface Report {
  id: string;
  name: string;
  type: 'time' | 'progress' | 'productivity' | 'budget' | 'custom';
  organizationId: string;
  createdBy: string;
  filters: ReportFilter[];
  dateRange: {
    start: Date;
    end: Date;
  };
  data: any;
  isScheduled: boolean;
  schedule?: ReportSchedule;
  recipients: string[];
  createdAt: Date;
  updatedAt: Date;
}

interface ReportFilter {
  field: string;
  operator: 'equals' | 'not_equals' | 'contains' | 'greater_than' | 'less_than' | 'in' | 'not_in';
  value: any;
}

interface ReportSchedule {
  frequency: 'daily' | 'weekly' | 'monthly';
  dayOfWeek?: number; // For weekly
  dayOfMonth?: number; // For monthly
  time: string; // "09:00"
}

// ダッシュボード
interface Dashboard {
  id: string;
  name: string;
  organizationId: string;
  userId: string;
  isDefault: boolean;
  widgets: DashboardWidget[];
  layout: DashboardLayout;
  createdAt: Date;
  updatedAt: Date;
}

interface DashboardWidget {
  id: string;
  type: 'task_summary' | 'project_progress' | 'time_tracking' | 'team_workload' | 'burndown_chart' | 'custom_chart';
  title: string;
  position: {
    x: number;
    y: number;
    width: number;
    height: number;
  };
  config: any;
  data?: any;
}

// 統合・自動化
interface Integration {
  id: string;
  name: string;
  type: 'slack' | 'github' | 'gitlab' | 'jira' | 'trello' | 'google_calendar' | 'zapier';
  organizationId: string;
  isActive: boolean;
  config: any;
  lastSyncAt?: Date;
  errorCount: number;
  createdBy: string;
  createdAt: Date;
  updatedAt: Date;
}

interface Automation {
  id: string;
  name: string;
  description?: string;
  organizationId: string;
  projectId?: string;
  trigger: AutomationTrigger;
  conditions: AutomationCondition[];
  actions: AutomationAction[];
  isActive: boolean;
  runCount: number;
  lastRunAt?: Date;
  createdBy: string;
  createdAt: Date;
  updatedAt: Date;
}

interface AutomationTrigger {
  type: 'task_created' | 'task_updated' | 'task_completed' | 'due_date_approaching' | 'schedule';
  config: any;
}

interface AutomationCondition {
  field: string;
  operator: string;
  value: any;
}

interface AutomationAction {
  type: 'assign_task' | 'update_status' | 'add_comment' | 'send_notification' | 'create_task';
  config: any;
}

// 請求・サブスクリプション
interface BillingInfo {
  customerId: string; // Stripe customer ID
  subscriptionId?: string;
  plan: string;
  status: 'active' | 'past_due' | 'canceled' | 'trialing';
  trialEndsAt?: Date;
  nextBillingDate?: Date;
  paymentMethod?: PaymentMethod;
  billingAddress: Address;
  invoices: Invoice[];
}

interface PaymentMethod {
  id: string;
  type: 'card' | 'bank_account';
  last4: string;
  brand?: string;
  expiryMonth?: number;
  expiryYear?: number;
}

interface Invoice {
  id: string;
  amount: number;
  currency: string;
  status: 'draft' | 'paid' | 'past_due' | 'canceled';
  paidAt?: Date;
  dueDate: Date;
  invoiceUrl: string;
}

// API応答
interface PaginatedResponse<T> {
  data: T[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
    hasNext: boolean;
    hasPrev: boolean;
  };
}

interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: {
    code: string;
    message: string;
    details?: any;
    timestamp: string;
  };
  meta?: {
    requestId: string;
    timestamp: string;
    version: string;
  };
}
```

#### データアーキテクチャ
- データモデル設計: マルチテナント対応PostgreSQL、行レベルセキュリティ（RLS）
- データフロー: フロントエンド → GraphQL API → マイクロサービス → データベース
- 状態管理: Apollo Client（GraphQL）、Zustand（UI状態）、React Query（REST API）
- キャッシュ戦略: Redis（セッション、実績データ）、CDN（静的アセット）、ブラウザキャッシュ

### アプリケーション
<!-- 自然言語での機能仕様とビジネス要件 -->

#### ビジネス要件
- ターゲットユーザー: 10-1000人規模の企業・チーム、プロジェクトマネージャー、開発チーム
- ビジネス価値: 生産性向上、プロジェクト可視化、チームコラボレーション促進
- 成功指標: ユーザー継続率90%、プロジェクト完了率向上20%、チーム満足度4.5/5
- 競合分析: Asana、Monday.com、Notionとの差別化（カスタマイズ性、API連携）

#### 主要機能

##### プロジェクト管理
- **プロジェクトダッシュボード**: 進捗状況、メトリクス、チーム状況の一元表示
- **タスク管理**: カンバンボード、リスト表示、ガントチャート、カレンダービュー
- **時間追跡**: タイムトラッキング、工数見積もり、実績分析
- **マイルストーン**: 重要な節目の管理、依存関係の可視化
- **リソース管理**: チームメンバーの稼働状況、スキル管理

##### チームコラボレーション
- **リアルタイム更新**: WebSocketによる即座の変更反映
- **コメント・メンション**: タスクやプロジェクトでのコミュニケーション
- **ファイル共有**: ドラッグ&ドロップアップロード、バージョン管理
- **通知システム**: メール、ブラウザ、Slack連携
- **チームスペース**: 部署・チーム別の専用エリア

##### 分析・レポート
- **ダッシュボード**: カスタマイズ可能なKPIダッシュボード
- **進捗レポート**: プロジェクト進捗、チーム生産性分析
- **時間レポート**: 工数分析、請求可能時間の管理
- **カスタムレポート**: SQLライクなクエリビルダー
- **データエクスポート**: CSV、PDF、Excel形式での出力

##### 管理機能
- **組織管理**: マルチテナント、部署・チーム階層
- **ユーザー管理**: ロール・権限管理、SSO連携
- **設定管理**: ワークフロー、通知、セキュリティ設定
- **監査ログ**: 全ての操作履歴、コンプライアンス対応
- **API管理**: API キー発行、レート制限、使用量監視

#### ユーザー体験
- ユーザーフロー:
  - 新規ユーザー: 招待受信 → サインアップ → オンボーディング → 初期設定
  - 日常利用: ログイン → ダッシュボード確認 → タスク作業 → 進捗更新 → コミュニケーション
- UI/UXデザイン原則: 情報密度の最適化、直感的ナビゲーション、データ可視化重視
- アクセシビリティ: WCAG 2.1 AA準拠、キーボードショートカット、多言語対応
- レスポンシブデザイン: デスクトップ優先、タブレット・モバイル対応

#### 機能要件

##### データ管理
- バリデーションルール:
  - プロジェクト名: 必須、3-100文字
  - タスクタイトル: 必須、1-255文字
  - 期限: 未来日付、プロジェクト期間内
  - 工数: 0以上の数値
- 権限・ロール:
  - 組織オーナー: 全権限
  - 管理者: ユーザー・プロジェクト管理
  - プロジェクトマネージャー: プロジェクト内全権限
  - メンバー: アサインされたタスクの編集
  - ゲスト: 限定的な閲覧権限

##### ワークフロー
- カスタムステータス: 組織・プロジェクト別のワークフロー設定
- 承認フロー: タスク完了時の承認プロセス
- 自動化ルール: トリガーベースの自動タスク更新
- 通知設定: 個人・チーム・プロジェクト別の通知制御

#### 非機能要件
- パフォーマンス:
  - ページ読み込み: 2秒以内
  - リアルタイム更新: 500ms以内
  - 大量データ処理: 10万タスク対応
- スケーラビリティ:
  - 組織数: 10万組織
  - 同時接続: 10万ユーザー
  - データ保存: テラバイト級
- 可用性: 99.9%稼働率（SLA）、計画メンテナンス月1回
- 信頼性: データ整合性保証、自動バックアップ、災害復旧

### システム

#### アーキテクチャ設計
- システムアーキテクチャ: マイクロサービス、イベント駆動、CQRS
- アプリケーションアーキテクチャ: Clean Architecture、DDD（ドメイン駆動設計）
- API設計: GraphQL（読み取り）、REST（操作）、WebSocket（リアルタイム）
- データベースアーキテクチャ: PostgreSQL（主）、Redis（キャッシュ）、ClickHouse（分析）

#### 技術スタック
- プログラミング言語: TypeScript（フロント・バック共通）
- フロントエンド: React 18、Next.js 14、Apollo Client
- バックエンド: Node.js、Express、GraphQL Yoga、Prisma
- データベース: PostgreSQL 15、Redis 7、ClickHouse
- インフラ: AWS（EKS、RDS、ElastiCache）、Terraform
- 認証: Auth0、SAML、OAuth 2.0
- 監視: DataDog、Sentry、LogRocket

#### 開発環境
```bash
# 環境構築
npm install
docker-compose up -d        # 開発用DB起動
npx prisma generate         # Prisma Client生成
npx prisma db push          # スキーマ同期

# 開発サーバー起動
npm run dev:frontend        # Next.js開発サーバー (3000)
npm run dev:backend         # Node.js APIサーバー (4000)
npm run dev:graphql         # GraphQL Playground (4000/graphql)

# テスト実行
npm run test               # Jest ユニットテスト
npm run test:integration   # 統合テスト
npm run test:e2e           # Playwright E2Eテスト
npm run test:load          # K6 負荷テスト

# コード品質チェック
npm run lint               # ESLint + Prettier
npm run type-check         # TypeScript型チェック
npm run audit              # セキュリティ監査
npm run analyze            # Bundle Analyzer

# ビルド・デプロイ
npm run build              # プロダクションビルド
npm run docker:build       # Dockerイメージビルド
npm run deploy:staging     # ステージング環境デプロイ
npm run deploy:production  # 本番環境デプロイ
```

#### 開発プロセス
- 開発手法: スクラム、2週間スプリント、継続的デリバリー
- バージョン管理: Git Flow、feature/develop/main ブランチ
- CI/CD: GitHub Actions、AWS CodePipeline、Blue-Green デプロイ
- テスト戦略:
  - ユニットテスト: Jest（90%以上カバレッジ）
  - 統合テスト: Supertest（API テスト）
  - E2Eテスト: Playwright（クリティカルフロー）
  - 負荷テスト: K6（パフォーマンス検証）
- コードレビュー: Pull Request、ペアプログラミング、技術レビュー

#### データ管理
```sql
-- マルチテナント対応スキーマ設計
CREATE TABLE organizations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  slug VARCHAR(100) UNIQUE NOT NULL,
  plan VARCHAR(50) DEFAULT 'free',
  settings JSONB,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  first_name VARCHAR(255) NOT NULL,
  last_name VARCHAR(255) NOT NULL,
  avatar VARCHAR(500),
  preferences JSONB,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE organization_members (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  role VARCHAR(50) NOT NULL,
  permissions JSONB,
  joined_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(organization_id, user_id)
);

CREATE TABLE projects (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
  name VARCHAR(255) NOT NULL,
  key VARCHAR(10) NOT NULL,
  description TEXT,
  status VARCHAR(50) DEFAULT 'active',
  owner_id UUID REFERENCES users(id),
  start_date DATE,
  end_date DATE,
  settings JSONB,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(organization_id, key)
);

CREATE TABLE tasks (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
  project_id UUID REFERENCES projects(id) ON DELETE CASCADE,
  title VARCHAR(255) NOT NULL,
  description TEXT,
  assignee_id UUID REFERENCES users(id),
  reporter_id UUID REFERENCES users(id) NOT NULL,
  status VARCHAR(50) DEFAULT 'todo',
  priority VARCHAR(20) DEFAULT 'medium',
  due_date TIMESTAMP,
  estimated_hours DECIMAL(5,2),
  actual_hours DECIMAL(5,2) DEFAULT 0,
  custom_fields JSONB,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- Row Level Security (RLS) 設定
ALTER TABLE organizations ENABLE ROW LEVEL SECURITY;
ALTER TABLE projects ENABLE ROW LEVEL SECURITY;
ALTER TABLE tasks ENABLE ROW LEVEL SECURITY;

-- 組織メンバーのみアクセス可能
CREATE POLICY organization_policy ON organizations
  USING (id IN (
    SELECT organization_id FROM organization_members 
    WHERE user_id = current_setting('app.current_user_id')::uuid
  ));

-- インデックス設計
CREATE INDEX idx_tasks_project_status ON tasks(project_id, status);
CREATE INDEX idx_tasks_assignee_status ON tasks(assignee_id, status);
CREATE INDEX idx_tasks_due_date ON tasks(due_date) WHERE due_date IS NOT NULL;
CREATE INDEX idx_organization_members_user ON organization_members(user_id);
```

- データ保護: PostgreSQL暗号化、GDPR準拠、データ匿名化
- バックアップ: 自動バックアップ（日次）、ポイントインタイムリカバリ
- データ保持: ユーザーデータ削除後30日保管、ログデータ2年保管

#### セキュリティ
- 認証・認可: 
  - JWT + Refresh Token
  - SSO（SAML、OIDC）
  - 多要素認証（MFA）
  - 行レベルセキュリティ（RLS）
- データセキュリティ:
  - 保存時暗号化: AES-256
  - 通信時暗号化: TLS 1.3
  - API暗号化: GraphQL over HTTPS
  - フィールドレベル暗号化: PII データ
- APIセキュリティ: レート制限、CORS、CSP、API認証
- 監査・コンプライアンス: 全操作ログ、SOC2 Type II、GDPR準拠

#### 運用・監視
- ログ管理:
  - アプリケーションログ: ELK Stack
  - アクセスログ: CloudWatch Logs
  - 監査ログ: 専用データベース
  - エラーログ: Sentry
- 監視・アラート:
  - インフラ監視: DataDog
  - APM: New Relic
  - ログ監視: LogRocket
  - ビジネス監視: Mixpanel
- パフォーマンス監視: リアルユーザー監視、Core Web Vitals
- SLA監視: 可用性、レスポンス時間、エラー率

#### 環境設定
```bash
# 必須環境変数
DATABASE_URL=postgresql://user:pass@localhost:5432/saas_app
REDIS_URL=redis://localhost:6379
NEXTAUTH_SECRET=your_nextauth_secret
NEXTAUTH_URL=http://localhost:3000

# 認証設定
AUTH0_DOMAIN=your-tenant.auth0.com
AUTH0_CLIENT_ID=your_auth0_client_id
AUTH0_CLIENT_SECRET=your_auth0_client_secret

# サードパーティ統合
SLACK_CLIENT_ID=your_slack_client_id
SLACK_CLIENT_SECRET=your_slack_client_secret
GITHUB_CLIENT_ID=your_github_client_id
GITHUB_CLIENT_SECRET=your_github_client_secret

# メール・通知
SENDGRID_API_KEY=your_sendgrid_api_key
FROM_EMAIL=noreply@yourapp.com

# ファイルストレージ
AWS_ACCESS_KEY_ID=your_aws_access_key
AWS_SECRET_ACCESS_KEY=your_aws_secret_key
S3_BUCKET_NAME=your-app-uploads

# 監視・分析
SENTRY_DSN=your_sentry_dsn
DATADOG_API_KEY=your_datadog_api_key
MIXPANEL_TOKEN=your_mixpanel_token

# 機能フラグ
ENABLE_ADVANCED_REPORTING=true
ENABLE_AUTOMATION=true
ENABLE_TIME_TRACKING=true
ENABLE_CUSTOM_FIELDS=true

# Stripe（請求）
STRIPE_SECRET_KEY=sk_live_...
STRIPE_PUBLISHABLE_KEY=pk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...
```

#### デプロイ・運用
- 環境管理: Development、Staging、Production、Sandbox
- デプロイ戦略: Blue-Green デプロイ、Canary リリース、ロールバック対応
- スケーリング: Kubernetes HPA、クラスタオートスケーラー
- 災害復旧: Multi-Region、RTO: 1時間、RPO: 15分

#### 品質保証
- テスト自動化:
  - ユニットテスト: 90%以上カバレッジ
  - API テスト: 全エンドポイント
  - E2E テスト: クリティカルユーザーフロー
  - 負荷テスト: 想定トラフィックの150%
- 品質メトリクス:
  - バグ密度: 0.1%以下
  - 平均修復時間: 4時間以内
  - 顧客満足度: 4.5/5以上

#### 統合・API
- REST API: OpenAPI 3.0仕様、バージョニング対応
- GraphQL API: スキーマファースト、型安全性
- WebSocket: リアルタイム更新、Socket.io
- Webhook: 外部システム連携、イベント通知
- SDK: JavaScript、Python、Go クライアント

#### 請求・サブスクリプション
- プラン管理: Free、Starter、Professional、Enterprise
- 使用量ベース課金: API呼び出し、ストレージ、ユーザー数
- Stripe連携: サブスクリプション管理、請求書発行
- 企業向け: カスタム契約、年次請求、請求書払い

#### コンプライアンス・ガバナンス
- SOC2 Type II: 年次監査、セキュリティ統制
- GDPR: データ保護、削除権、ポータビリティ
- ISO 27001: 情報セキュリティ管理
- 企業ポリシー: データ保持、アクセス制御、インシデント対応

#### プロジェクト管理
- チーム構成: フルスタック開発者、DevOpsエンジニア、プロダクトマネージャー、デザイナー、QAエンジニア
- 開発スケジュール: MVP 9ヶ月、ベータ版6ヶ月、GA 3ヶ月
- リスク管理: 技術負債、スケーラビリティ、セキュリティ、法規制
- ステークホルダー: 創設者、投資家、顧客、パートナー企業