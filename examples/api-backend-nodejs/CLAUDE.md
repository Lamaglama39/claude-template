# CLAUDE.md - REST API バックエンド（Node.js + Express）

## 設計

### モデル
<!-- TypeScriptで厳密に型定義したコアデータ構造と関係性 -->
```typescript
// API基盤・認証
interface ApiKey {
  id: string;
  name: string;
  organizationId: string;
  keyPrefix: string; // "pk_live_", "pk_test_"
  hashedKey: string;
  permissions: ApiPermission[];
  rateLimit: RateLimit;
  isActive: boolean;
  lastUsedAt?: Date;
  expiresAt?: Date;
  createdBy: string;
  createdAt: Date;
  updatedAt: Date;
}

interface ApiPermission {
  resource: string; // "users", "orders", "products"
  actions: ('create' | 'read' | 'update' | 'delete')[];
  conditions?: {
    fields?: string[];
    filters?: { [key: string]: any };
  };
}

interface RateLimit {
  requestsPerSecond: number;
  requestsPerMinute: number;
  requestsPerHour: number;
  requestsPerDay: number;
  burstLimit: number;
}

interface Organization {
  id: string;
  name: string;
  slug: string;
  plan: 'free' | 'starter' | 'professional' | 'enterprise';
  settings: OrganizationSettings;
  subscription: Subscription;
  usage: UsageMetrics;
  status: 'active' | 'suspended' | 'deleted';
  createdAt: Date;
  updatedAt: Date;
}

interface OrganizationSettings {
  timezone: string;
  webhookEndpoints: WebhookEndpoint[];
  ipWhitelist: string[];
  dataRetention: {
    logs: number; // days
    analytics: number; // days
    backups: number; // days
  };
  security: {
    requireHttps: boolean;
    allowedOrigins: string[];
    sessionTimeout: number; // minutes
  };
}

// ユーザー・認証
interface User {
  id: string;
  email: string;
  firstName?: string;
  lastName?: string;
  avatar?: string;
  organizationId: string;
  role: UserRole;
  permissions: Permission[];
  status: 'active' | 'inactive' | 'suspended';
  emailVerified: boolean;
  twoFactorEnabled: boolean;
  lastLoginAt?: Date;
  metadata: { [key: string]: any };
  createdAt: Date;
  updatedAt: Date;
}

interface UserRole {
  id: string;
  name: string;
  description: string;
  isSystem: boolean;
  permissions: Permission[];
}

interface Permission {
  resource: string;
  actions: string[];
  conditions?: { [key: string]: any };
}

interface Session {
  id: string;
  userId: string;
  token: string;
  refreshToken?: string;
  deviceInfo: DeviceInfo;
  ipAddress: string;
  userAgent: string;
  isActive: boolean;
  expiresAt: Date;
  lastAccessedAt: Date;
  createdAt: Date;
}

interface DeviceInfo {
  type: 'web' | 'mobile' | 'desktop' | 'api';
  os?: string;
  browser?: string;
  version?: string;
  fingerprint: string;
}

// データ・リソース管理
interface Resource {
  id: string;
  type: string;
  organizationId: string;
  data: { [key: string]: any };
  metadata: ResourceMetadata;
  relationships: ResourceRelationship[];
  status: 'active' | 'archived' | 'deleted';
  version: number;
  createdBy: string;
  updatedBy: string;
  createdAt: Date;
  updatedAt: Date;
  deletedAt?: Date;
}

interface ResourceMetadata {
  tags: string[];
  labels: { [key: string]: string };
  annotations: { [key: string]: string };
  checksum: string;
  size?: number;
  encoding?: string;
}

interface ResourceRelationship {
  type: 'parent' | 'child' | 'reference' | 'dependency';
  resourceId: string;
  metadata?: { [key: string]: any };
}

// API Request/Response
interface ApiRequest {
  id: string;
  method: 'GET' | 'POST' | 'PUT' | 'PATCH' | 'DELETE';
  path: string;
  query: { [key: string]: string | string[] };
  headers: { [key: string]: string };
  body?: any;
  organizationId: string;
  userId?: string;
  apiKeyId?: string;
  ipAddress: string;
  userAgent: string;
  timestamp: Date;
}

interface ApiResponse {
  requestId: string;
  statusCode: number;
  data?: any;
  error?: ApiError;
  metadata: ResponseMetadata;
  headers: { [key: string]: string };
  timestamp: Date;
  processingTime: number; // milliseconds
}

interface ApiError {
  code: string;
  message: string;
  details?: any;
  stack?: string;
  requestId: string;
  timestamp: Date;
}

interface ResponseMetadata {
  version: string;
  requestId: string;
  rateLimit?: RateLimitInfo;
  pagination?: PaginationInfo;
  warnings?: string[];
}

interface RateLimitInfo {
  limit: number;
  remaining: number;
  resetAt: Date;
  retryAfter?: number;
}

interface PaginationInfo {
  page: number;
  limit: number;
  total: number;
  totalPages: number;
  hasNext: boolean;
  hasPrev: boolean;
  nextCursor?: string;
  prevCursor?: string;
}

// Webhook・イベント
interface WebhookEndpoint {
  id: string;
  organizationId: string;
  url: string;
  events: string[];
  secret: string;
  isActive: boolean;
  failureCount: number;
  lastSuccessAt?: Date;
  lastFailureAt?: Date;
  metadata: { [key: string]: any };
  createdAt: Date;
  updatedAt: Date;
}

interface WebhookEvent {
  id: string;
  type: string;
  organizationId: string;
  data: any;
  metadata: EventMetadata;
  deliveries: WebhookDelivery[];
  createdAt: Date;
}

interface EventMetadata {
  source: string;
  version: string;
  correlationId?: string;
  causedBy?: {
    userId?: string;
    apiKeyId?: string;
    system?: string;
  };
}

interface WebhookDelivery {
  id: string;
  webhookEndpointId: string;
  eventId: string;
  attemptNumber: number;
  status: 'pending' | 'delivered' | 'failed' | 'abandoned';
  responseCode?: number;
  responseBody?: string;
  errorMessage?: string;
  deliveredAt?: Date;
  nextRetryAt?: Date;
  createdAt: Date;
}

// 監視・分析
interface ApiMetric {
  id: string;
  organizationId: string;
  timestamp: Date;
  metrics: {
    requests: {
      total: number;
      successful: number;
      failed: number;
      byMethod: { [method: string]: number };
      byStatus: { [status: string]: number };
      byEndpoint: { [endpoint: string]: number };
    };
    performance: {
      averageResponseTime: number;
      p50ResponseTime: number;
      p95ResponseTime: number;
      p99ResponseTime: number;
    };
    errors: {
      total: number;
      byType: { [type: string]: number };
      byEndpoint: { [endpoint: string]: number };
    };
    rateLimit: {
      total: number;
      byApiKey: { [keyId: string]: number };
    };
  };
}

interface LogEntry {
  id: string;
  timestamp: Date;
  level: 'debug' | 'info' | 'warn' | 'error' | 'fatal';
  message: string;
  organizationId?: string;
  userId?: string;
  requestId?: string;
  component: string;
  metadata: { [key: string]: any };
  stackTrace?: string;
}

interface HealthCheck {
  service: string;
  status: 'healthy' | 'degraded' | 'unhealthy';
  timestamp: Date;
  responseTime: number;
  details: {
    database: ServiceStatus;
    redis: ServiceStatus;
    externalApis: { [name: string]: ServiceStatus };
    storage: ServiceStatus;
    messageQueue: ServiceStatus;
  };
  version: string;
  uptime: number;
}

interface ServiceStatus {
  status: 'healthy' | 'degraded' | 'unhealthy';
  responseTime?: number;
  errorMessage?: string;
  lastChecked: Date;
}

// ジョブ・タスク管理
interface Job {
  id: string;
  type: string;
  organizationId?: string;
  priority: 'low' | 'normal' | 'high' | 'critical';
  payload: any;
  status: 'pending' | 'running' | 'completed' | 'failed' | 'canceled';
  attempts: number;
  maxAttempts: number;
  runAt: Date;
  startedAt?: Date;
  completedAt?: Date;
  failedAt?: Date;
  result?: any;
  error?: string;
  metadata: { [key: string]: any };
  createdAt: Date;
  updatedAt: Date;
}

interface ScheduledTask {
  id: string;
  name: string;
  organizationId?: string;
  jobType: string;
  schedule: string; // cron expression
  payload: any;
  isActive: boolean;
  timezone: string;
  lastRunAt?: Date;
  nextRunAt: Date;
  successCount: number;
  failureCount: number;
  createdAt: Date;
  updatedAt: Date;
}

// 設定・環境
interface Configuration {
  id: string;
  key: string;
  value: any;
  type: 'string' | 'number' | 'boolean' | 'json' | 'secret';
  environment: 'development' | 'staging' | 'production';
  organizationId?: string;
  description?: string;
  isSecret: boolean;
  version: number;
  createdBy: string;
  updatedBy: string;
  createdAt: Date;
  updatedAt: Date;
}

interface FeatureFlag {
  id: string;
  key: string;
  name: string;
  description?: string;
  isEnabled: boolean;
  rolloutPercentage: number;
  conditions: FlagCondition[];
  organizationIds?: string[];
  userIds?: string[];
  createdBy: string;
  createdAt: Date;
  updatedAt: Date;
}

interface FlagCondition {
  field: string;
  operator: 'equals' | 'not_equals' | 'in' | 'not_in' | 'contains' | 'starts_with';
  value: any;
}

// サブスクリプション・請求
interface Subscription {
  id: string;
  organizationId: string;
  plan: string;
  status: 'active' | 'past_due' | 'canceled' | 'trialing';
  currentPeriodStart: Date;
  currentPeriodEnd: Date;
  trialEnd?: Date;
  canceledAt?: Date;
  billingCycle: 'monthly' | 'yearly';
  paymentMethod?: PaymentMethod;
  discounts: Discount[];
  metadata: { [key: string]: any };
}

interface PaymentMethod {
  id: string;
  type: 'card' | 'bank_account' | 'digital_wallet';
  details: any;
  isDefault: boolean;
  expiresAt?: Date;
}

interface UsageMetrics {
  organizationId: string;
  period: string; // "2024-01"
  metrics: {
    apiCalls: number;
    storage: number; // bytes
    bandwidth: number; // bytes
    users: number;
    customMetrics: { [key: string]: number };
  };
  costs: {
    total: number;
    breakdown: { [item: string]: number };
    currency: string;
  };
  limits: {
    apiCalls: number;
    storage: number;
    bandwidth: number;
    users: number;
  };
}

// 統合・外部API
interface Integration {
  id: string;
  organizationId: string;
  type: string;
  name: string;
  config: IntegrationConfig;
  status: 'active' | 'inactive' | 'error';
  lastSyncAt?: Date;
  lastErrorAt?: Date;
  errorMessage?: string;
  metadata: { [key: string]: any };
  createdAt: Date;
  updatedAt: Date;
}

interface IntegrationConfig {
  baseUrl?: string;
  apiKey?: string;
  clientId?: string;
  clientSecret?: string;
  scope?: string[];
  webhookUrl?: string;
  settings: { [key: string]: any };
}

interface ExternalApiCall {
  id: string;
  integrationId: string;
  method: string;
  url: string;
  requestHeaders: { [key: string]: string };
  requestBody?: any;
  responseStatus: number;
  responseHeaders: { [key: string]: string };
  responseBody?: any;
  duration: number;
  success: boolean;
  errorMessage?: string;
  timestamp: Date;
}

// 汎用型
interface PaginatedResult<T> {
  data: T[];
  pagination: PaginationInfo;
  metadata?: { [key: string]: any };
}

interface BulkOperation<T> {
  id: string;
  type: 'create' | 'update' | 'delete';
  organizationId: string;
  items: T[];
  status: 'pending' | 'processing' | 'completed' | 'failed';
  results: BulkOperationResult[];
  totalCount: number;
  successCount: number;
  failureCount: number;
  startedAt?: Date;
  completedAt?: Date;
  createdBy: string;
  createdAt: Date;
}

interface BulkOperationResult {
  index: number;
  success: boolean;
  resourceId?: string;
  error?: string;
}

interface Audit {
  id: string;
  organizationId: string;
  userId?: string;
  action: string;
  resource: string;
  resourceId: string;
  oldValues?: any;
  newValues?: any;
  metadata: {
    ipAddress: string;
    userAgent: string;
    requestId: string;
    source: 'api' | 'web' | 'system';
  };
  timestamp: Date;
}
```

#### データアーキテクチャ
- データモデル設計: PostgreSQL（主要）、Redis（キャッシュ・セッション）、ClickHouse（分析）
- データフロー: リクエスト受信 → 認証・認可 → ビジネスロジック → データ永続化 → レスポンス
- 状態管理: Stateless API、Redis（セッション）、データベーストランザクション
- キャッシュ戦略: Redis（頻繁アクセス）、メモリキャッシュ（設定）、CDN（静的コンテンツ）

### アプリケーション
<!-- 自然言語での機能仕様とビジネス要件 -->

#### ビジネス要件
- ターゲットユーザー: SaaS開発者、エンタープライズ開発チーム、API統合パートナー
- ビジネス価値: 開発生産性向上、システム統合促進、データ活用支援
- 成功指標: API使用量月1000万コール、稼働率99.9%、開発者満足度4.5/5
- 競合分析: AWS API Gateway、Stripe API、Twilio APIとの差別化（使いやすさ、価格）

#### 主要機能

##### 認証・認可
- **API Key認証**: プレフィックス付きキー、権限スコープ設定
- **JWT認証**: ステートレス認証、リフレッシュトークン対応
- **OAuth 2.0**: サードパーティ統合、スコープベース認可
- **ロールベース認可**: 細粒度権限制御、動的権限評価
- **マルチテナント**: 組織分離、クロステナント防止

##### API管理
- **RESTful API**: 統一されたリソース設計、HTTP標準準拠
- **バージョニング**: URLパス、ヘッダー、クエリパラメータ対応
- **レート制限**: 階層的制限、バースト対応、フェアユース
- **入力検証**: JSONスキーマ、型安全性、サニタイゼーション
- **エラーハンドリング**: 構造化エラー、国際化対応、詳細情報

##### データ管理
- **CRUD操作**: Create、Read、Update、Delete、Upsert
- **クエリ機能**: フィルタリング、ソート、ページネーション、検索
- **バルク操作**: 一括作成・更新・削除、バッチ処理
- **トランザクション**: ACID特性、分散トランザクション
- **データ変換**: 入出力変換、フォーマット標準化

##### リアルタイム機能
- **WebSocket**: リアルタイム通信、接続管理
- **Server-Sent Events**: 一方向イベント配信
- **Webhook**: イベント通知、配信保証、リトライ機構
- **ストリーミング**: 大容量データ配信、チャンク転送
- **プッシュ通知**: モバイル・Web通知、優先度制御

##### 分析・監視
- **メトリクス収集**: リクエスト数、レスポンス時間、エラー率
- **ログ管理**: 構造化ログ、ログレベル、保持期間
- **トレーシング**: 分散トレーシング、パフォーマンス分析
- **アラート**: 閾値監視、異常検知、通知
- **ダッシュボード**: リアルタイム可視化、カスタムメトリクス

#### ユーザー体験
- 開発者体験:
  - API探索 → ドキュメント確認 → 認証設定 → 実装・テスト → 本番利用
  - SDK提供 → サンプルコード → インタラクティブドキュメント
- 管理者体験:
  - システム監視 → パフォーマンス分析 → アラート対応 → 設定調整
- パートナー体験:
  - API統合 → データ連携 → ビジネス拡張 → 収益化

#### 機能要件

##### パフォーマンス要件
- レスポンス時間: 
  - 単純取得: 100ms以内
  - 複雑クエリ: 500ms以内
  - バルク操作: 5秒以内
- スループット:
  - 通常時: 10,000 RPS
  - ピーク時: 50,000 RPS
  - 同時接続: 100,000コネクション

##### 信頼性要件
- 可用性: 99.9%（SLA）
- データ整合性: ACID特性保証
- 障害復旧: 自動フェイルオーバー、データ復旧
- バックアップ: 日次バックアップ、ポイントインタイムリカバリ

#### 非機能要件
- スケーラビリティ:
  - 水平スケーリング対応
  - オートスケーリング
  - 負荷分散
  - データベースシャーディング
- セキュリティ:
  - HTTPS必須
  - 入力検証・サニタイゼーション
  - SQLインジェクション対策
  - DDoS攻撃対策
- 保守性:
  - モジュラー設計
  - 自動テスト
  - 継続的インテグレーション
  - 設定外部化

### システム

#### アーキテクチャ設計
- システムアーキテクチャ: マイクロサービス、レイヤードアーキテクチャ
- アプリケーションアーキテクチャ: Clean Architecture、Dependency Injection
- API設計: RESTful、OpenAPI 3.0、GraphQL（オプション）
- データベースアーキテクチャ: Primary-Replica、接続プール、クエリ最適化

#### 技術スタック
- プログラミング言語: Node.js 20 + TypeScript
- フレームワーク: Express.js、Fastify（高パフォーマンス版）
- データベース: PostgreSQL 15、Redis 7
- ORM: Prisma、TypeORM
- インフラ: Docker、Kubernetes、AWS/GCP
- 監視: Prometheus、Grafana、Jaeger
- メッセージング: Redis Pub/Sub、RabbitMQ、Kafka

#### 開発環境
```bash
# 環境構築
npm install
docker-compose up -d         # 開発用インフラ起動
npx prisma generate          # Prismaクライアント生成
npx prisma db push           # データベーススキーマ同期
npm run seed                 # 初期データ投入

# 開発サーバー起動
npm run dev                  # 開発サーバー（ホットリロード）
npm run dev:debug           # デバッグモード
npm run dev:watch           # ファイル監視モード

# テスト実行
npm run test                # Jest ユニットテスト
npm run test:integration    # 統合テスト
npm run test:e2e           # E2Eテスト
npm run test:load          # 負荷テスト（K6）
npm run test:security      # セキュリティテスト

# コード品質チェック
npm run lint               # ESLint
npm run type-check         # TypeScript型チェック
npm run format             # Prettier
npm run audit              # 脆弱性チェック
npm run coverage           # カバレッジレポート

# ビルド・デプロイ
npm run build              # プロダクションビルド
npm run start              # プロダクション実行
npm run docker:build       # Dockerイメージビルド
npm run deploy:staging     # ステージングデプロイ
npm run deploy:production  # 本番デプロイ

# ドキュメント生成
npm run docs:generate      # OpenAPI仕様書生成
npm run docs:serve         # ドキュメントサーバー起動
```

#### 開発プロセス
- 開発手法: アジャイル、2週間スプリント、継続的デリバリー
- バージョン管理: Git Flow、semantic versioning
- CI/CD: GitHub Actions、自動テスト、段階的デプロイ
- テスト戦略:
  - ユニットテスト: 90%以上カバレッジ
  - 統合テスト: API エンドポイント
  - E2Eテスト: クリティカルフロー
  - 負荷テスト: 性能要件検証
- コードレビュー: Pull Request、自動チェック、ペアプログラミング

#### データ管理
```sql
-- PostgreSQL スキーマ設計
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pg_stat_statements";

-- 組織・マルチテナント
CREATE TABLE organizations (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  name VARCHAR(255) NOT NULL,
  slug VARCHAR(100) UNIQUE NOT NULL,
  plan VARCHAR(50) DEFAULT 'free',
  settings JSONB DEFAULT '{}',
  status VARCHAR(20) DEFAULT 'active',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- ユーザー
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  email VARCHAR(255) UNIQUE NOT NULL,
  first_name VARCHAR(255),
  last_name VARCHAR(255),
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
  role VARCHAR(50) DEFAULT 'member',
  status VARCHAR(20) DEFAULT 'active',
  email_verified BOOLEAN DEFAULT FALSE,
  metadata JSONB DEFAULT '{}',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- API キー
CREATE TABLE api_keys (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
  name VARCHAR(255) NOT NULL,
  key_prefix VARCHAR(20) NOT NULL,
  hashed_key VARCHAR(255) NOT NULL,
  permissions JSONB DEFAULT '{}',
  rate_limit JSONB DEFAULT '{}',
  is_active BOOLEAN DEFAULT TRUE,
  last_used_at TIMESTAMP,
  expires_at TIMESTAMP,
  created_by UUID REFERENCES users(id),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- リソース（汎用データストレージ）
CREATE TABLE resources (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
  type VARCHAR(100) NOT NULL,
  data JSONB NOT NULL,
  metadata JSONB DEFAULT '{}',
  status VARCHAR(20) DEFAULT 'active',
  version INTEGER DEFAULT 1,
  created_by UUID REFERENCES users(id),
  updated_by UUID REFERENCES users(id),
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  deleted_at TIMESTAMP
);

-- API リクエストログ
CREATE TABLE api_requests (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  organization_id UUID REFERENCES organizations(id),
  user_id UUID REFERENCES users(id),
  api_key_id UUID REFERENCES api_keys(id),
  method VARCHAR(10) NOT NULL,
  path VARCHAR(500) NOT NULL,
  query_params JSONB,
  request_headers JSONB,
  request_body JSONB,
  response_status INTEGER,
  response_time INTEGER, -- milliseconds
  ip_address INET,
  user_agent TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

-- パーティショニング（ログテーブルのパフォーマンス向上）
CREATE TABLE api_requests_2024_01 PARTITION OF api_requests
FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

-- インデックス設計
CREATE INDEX idx_organizations_slug ON organizations(slug);
CREATE INDEX idx_users_organization ON users(organization_id);
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_api_keys_organization ON api_keys(organization_id);
CREATE INDEX idx_api_keys_prefix ON api_keys(key_prefix);
CREATE INDEX idx_resources_organization_type ON resources(organization_id, type);
CREATE INDEX idx_resources_created_at ON resources(created_at);
CREATE INDEX idx_api_requests_organization_created ON api_requests(organization_id, created_at);
CREATE INDEX idx_api_requests_path_created ON api_requests(path, created_at);

-- 全文検索インデックス（リソース検索用）
CREATE INDEX idx_resources_data_gin ON resources USING GIN(data);
```

- データ保護: 行レベルセキュリティ（RLS）、暗号化、データマスキング
- バックアップ: 自動バックアップ、地理的冗長性、ポイントインタイムリカバリ
- データ保持: アクティブデータ無期限、ログデータ90日、監査データ7年

#### セキュリティ
- 認証・認可:
  - API キー認証（HMAC署名）
  - JWT トークン（RS256署名）
  - OAuth 2.0 + PKCE
  - ロールベースアクセス制御（RBAC）
- データセキュリティ:
  - 保存時暗号化（AES-256）
  - 通信時暗号化（TLS 1.3）
  - 機密データハッシュ化
  - データベース暗号化
- API セキュリティ:
  - レート制限（Token Bucket）
  - 入力検証・サニタイゼーション
  - CORS設定
  - CSP（Content Security Policy）
  - OWASP Top 10対策
- インフラセキュリティ:
  - WAF（Web Application Firewall）
  - DDoS攻撃対策
  - IP ホワイトリスト
  - セキュリティグループ

#### 運用・監視
- ログ管理:
  - 構造化ログ（JSON）
  - ログレベル設定
  - ログローテーション
  - 集約・検索（ELK Stack）
- 監視・アラート:
  - アプリケーション監視（APM）
  - インフラ監視（Prometheus）
  - リアルタイムダッシュボード
  - 異常検知・アラート
- パフォーマンス監視:
  - レスポンス時間監視
  - スループット監視
  - エラー率監視
  - リソース使用量監視
- ヘルスチェック:
  - アプリケーションヘルス
  - データベースヘルス
  - 外部依存サービスヘルス
  - 自動復旧機能

#### 環境設定
```bash
# 必須環境変数
NODE_ENV=production
PORT=3000
DATABASE_URL=postgresql://user:pass@localhost:5432/api_db
REDIS_URL=redis://localhost:6379

# セキュリティ
JWT_SECRET=your_jwt_secret_key
API_SECRET=your_api_secret_key
ENCRYPTION_KEY=your_encryption_key
CORS_ORIGINS=https://app.example.com,https://admin.example.com

# データベース
DB_POOL_MIN=5
DB_POOL_MAX=20
DB_CONNECTION_TIMEOUT=10000
DB_STATEMENT_TIMEOUT=30000

# Redis設定
REDIS_POOL_MIN=5
REDIS_POOL_MAX=20
REDIS_KEY_PREFIX=api:
REDIS_SESSION_TTL=3600

# レート制限
RATE_LIMIT_WINDOW=3600000
RATE_LIMIT_MAX_REQUESTS=1000
RATE_LIMIT_SKIP_SUCCESSFUL_REQUESTS=false

# 外部サービス
SMTP_HOST=smtp.sendgrid.net
SMTP_PORT=587
SMTP_USER=apikey
SMTP_PASS=your_sendgrid_api_key

# 監視・ログ
LOG_LEVEL=info
LOG_FORMAT=json
SENTRY_DSN=your_sentry_dsn
PROMETHEUS_METRICS=true

# 機能フラグ
ENABLE_GRAPHQL=false
ENABLE_WEBSOCKETS=true
ENABLE_RATE_LIMITING=true
ENABLE_CACHING=true

# 開発・テスト用
SWAGGER_ENABLED=true
DEBUG_SQL=false
MOCK_EXTERNAL_APIS=false
```

#### デプロイ・運用
- 環境管理: Development、Testing、Staging、Production
- デプロイ戦略: Blue-Green デプロイ、Rolling Update、Canary Release
- スケーリング: Horizontal Pod Autoscaler、Cluster Autoscaler
- 災害復旧: Multi-Region、自動フェイルオーバー、データ復旧

#### 品質保証
- テスト自動化:
  - ユニットテスト: 90%以上カバレッジ
  - API テスト: 全エンドポイント
  - 負荷テスト: 想定トラフィックの200%
  - セキュリティテスト: 自動脆弱性スキャン
- 品質メトリクス:
  - 可用性: 99.9%以上
  - 平均レスポンス時間: 200ms以下
  - エラー率: 0.1%以下
  - セキュリティインシデント: 0件

#### ドキュメント・開発者支援
- API ドキュメント: OpenAPI 3.0、インタラクティブドキュメント
- SDK: JavaScript、Python、Go、PHP クライアント
- サンプルコード: 主要言語での実装例
- チュートリアル: Getting Started、Best Practices
- FAQ・トラブルシューティング: よくある問題と解決方法

#### 課金・使用量管理
- 使用量計測: API コール数、データ転送量、ストレージ使用量
- 課金モデル: 従量課金、階層課金、カスタム料金
- 請求: 月次請求、Stripe 連携、請求書発行
- 使用量制限: プラン別制限、オーバージ防止、アラート

#### 国際化・ローカライゼーション
- 多言語対応: エラーメッセージ、ドキュメント
- 地域対応: タイムゾーン、通貨、日付形式
- 法規制対応: GDPR、CCPA、個人情報保護法
- データ主権: 地域別データ保存、データ移転制限

#### プロジェクト管理
- チーム構成: バックエンド開発者、DevOps エンジニア、QA エンジニア、プロダクトマネージャー
- 開発期間: アーキテクチャ設計3ヶ月、MVP開発6ヶ月、拡張機能開発継続
- リスク管理: 技術的負債、パフォーマンス劣化、セキュリティ脆弱性
- ステークホルダー: エンジニア、プロダクトマネージャー、セキュリティチーム、法務チーム