# CLAUDE.md - フィンテックアプリ（個人向け投資・資産管理）

## 設計

### モデル
<!-- TypeScriptで厳密に型定義したコアデータ構造と関係性 -->
```typescript
// ユーザー・KYC管理
interface User {
  id: string;
  email: string;
  phoneNumber: string;
  firstName: string;
  lastName: string;
  firstNameKana?: string;
  lastNameKana?: string;
  dateOfBirth: Date;
  nationality: string;
  address: Address;
  kycStatus: KYCStatus;
  riskProfile: RiskProfile;
  accountTier: 'basic' | 'standard' | 'premium' | 'private';
  settings: UserSettings;
  complianceFlags: ComplianceFlag[];
  deviceInfo: DeviceInfo[];
  loginHistory: LoginHistory[];
  lastLoginAt: Date;
  isActive: boolean;
  suspensionReason?: string;
  createdAt: Date;
  updatedAt: Date;
}

interface Address {
  country: string;
  postalCode: string;
  prefecture: string;
  city: string;
  addressLine1: string;
  addressLine2?: string;
  isVerified: boolean;
  verifiedAt?: Date;
}

interface KYCStatus {
  level: 0 | 1 | 2 | 3; // 0=未確認, 1=基本, 2=詳細, 3=最高レベル
  status: 'pending' | 'in_review' | 'approved' | 'rejected' | 'expired';
  documents: KYCDocument[];
  verificationMethods: string[]; // ['email', 'sms', 'video_call', 'bank_account']
  riskScore: number; // 0-100
  lastReviewAt?: Date;
  expiresAt?: Date;
  rejectionReason?: string;
}

interface KYCDocument {
  id: string;
  type: 'drivers_license' | 'passport' | 'residence_card' | 'utility_bill' | 'bank_statement';
  status: 'pending' | 'approved' | 'rejected';
  uploadedAt: Date;
  reviewedAt?: Date;
  expiresAt?: Date;
  rejectionReason?: string;
}

interface RiskProfile {
  investmentExperience: 'none' | 'limited' | 'moderate' | 'extensive';
  riskTolerance: 'conservative' | 'moderate' | 'aggressive';
  investmentHorizon: 'short' | 'medium' | 'long'; // <1年, 1-5年, 5年+
  annualIncome: number;
  netWorth: number;
  investmentGoals: string[];
  liquidityNeeds: 'high' | 'medium' | 'low';
  lastUpdated: Date;
}

interface UserSettings {
  language: 'ja' | 'en';
  currency: 'JPY' | 'USD' | 'EUR';
  timezone: string;
  notifications: NotificationSettings;
  security: SecuritySettings;
  trading: TradingSettings;
  privacy: PrivacySettings;
}

interface NotificationSettings {
  push: boolean;
  email: boolean;
  sms: boolean;
  priceAlerts: boolean;
  orderUpdates: boolean;
  newsAlerts: boolean;
  marketHours: boolean;
  maintenance: boolean;
}

interface SecuritySettings {
  biometricEnabled: boolean;
  twoFactorEnabled: boolean;
  deviceAuthRequired: boolean;
  sessionTimeout: number; // minutes
  allowedIPs: string[];
  loginNotifications: boolean;
}

// アカウント・残高管理
interface Account {
  id: string;
  userId: string;
  type: 'securities' | 'crypto' | 'cash' | 'savings';
  currency: string;
  balance: MoneyAmount;
  availableBalance: MoneyAmount;
  frozenAmount: MoneyAmount;
  accountNumber: string;
  routingNumber?: string;
  bankName?: string;
  isMainAccount: boolean;
  status: 'active' | 'frozen' | 'closed';
  openedAt: Date;
  lastTransactionAt?: Date;
}

interface MoneyAmount {
  amount: number;
  currency: string;
}

interface Portfolio {
  id: string;
  userId: string;
  accountId: string;
  name: string;
  totalValue: MoneyAmount;
  totalCost: MoneyAmount;
  totalGainLoss: MoneyAmount;
  totalGainLossPercent: number;
  dayGainLoss: MoneyAmount;
  dayGainLossPercent: number;
  positions: Position[];
  allocation: AssetAllocation;
  riskMetrics: RiskMetrics;
  lastUpdated: Date;
}

interface Position {
  id: string;
  portfolioId: string;
  instrumentId: string;
  instrumentType: 'stock' | 'etf' | 'mutual_fund' | 'bond' | 'crypto' | 'forex';
  symbol: string;
  quantity: number;
  averagePrice: number;
  currentPrice: number;
  marketValue: MoneyAmount;
  costBasis: MoneyAmount;
  gainLoss: MoneyAmount;
  gainLossPercent: number;
  dayGainLoss: MoneyAmount;
  dayGainLossPercent: number;
  lastUpdated: Date;
}

// 金融商品・マーケットデータ
interface Instrument {
  id: string;
  symbol: string;
  name: string;
  type: 'stock' | 'etf' | 'mutual_fund' | 'bond' | 'crypto' | 'forex';
  exchange: string;
  currency: string;
  sector?: string;
  industry?: string;
  description?: string;
  website?: string;
  marketCap?: number;
  peRatio?: number;
  dividendYield?: number;
  beta?: number;
  isActive: boolean;
  tradingHours: TradingHours;
  minOrderSize: number;
  priceIncrement: number;
  regulations: string[];
}

interface TradingHours {
  timezone: string;
  preMarket?: TimeRange;
  regular: TimeRange;
  afterMarket?: TimeRange;
  holidays: Date[];
}

interface TimeRange {
  start: string; // "09:00"
  end: string;   // "15:00"
}

interface MarketData {
  instrumentId: string;
  symbol: string;
  price: number;
  change: number;
  changePercent: number;
  volume: number;
  high: number;
  low: number;
  open: number;
  previousClose: number;
  bid: number;
  ask: number;
  bidSize: number;
  askSize: number;
  timestamp: Date;
}

interface PriceHistory {
  instrumentId: string;
  interval: '1m' | '5m' | '15m' | '1h' | '1d' | '1w' | '1M';
  data: PricePoint[];
  lastUpdated: Date;
}

interface PricePoint {
  timestamp: Date;
  open: number;
  high: number;
  low: number;
  close: number;
  volume: number;
}

// 取引・注文管理
interface Order {
  id: string;
  userId: string;
  accountId: string;
  instrumentId: string;
  symbol: string;
  side: 'buy' | 'sell';
  type: 'market' | 'limit' | 'stop' | 'stop_limit' | 'trail_stop';
  quantity: number;
  price?: number;
  stopPrice?: number;
  trailAmount?: number;
  timeInForce: 'day' | 'gtc' | 'ioc' | 'fok'; // GTC=Good Till Canceled
  status: 'pending' | 'open' | 'partial_filled' | 'filled' | 'canceled' | 'rejected';
  filledQuantity: number;
  remainingQuantity: number;
  averagePrice?: number;
  commissions: Commission[];
  fees: Fee[];
  estimatedTotal: MoneyAmount;
  actualTotal?: MoneyAmount;
  placedAt: Date;
  updatedAt: Date;
  executedAt?: Date;
  canceledAt?: Date;
  rejectionReason?: string;
}

interface Trade {
  id: string;
  orderId: string;
  userId: string;
  instrumentId: string;
  symbol: string;
  side: 'buy' | 'sell';
  quantity: number;
  price: number;
  value: MoneyAmount;
  commission: MoneyAmount;
  fees: Fee[];
  netAmount: MoneyAmount;
  settlementDate: Date;
  tradeDate: Date;
  executionVenue: string;
}

interface Commission {
  type: 'fixed' | 'percentage';
  amount: MoneyAmount;
  description: string;
}

interface Fee {
  type: 'regulatory' | 'exchange' | 'clearing' | 'sec' | 'tax';
  amount: MoneyAmount;
  description: string;
}

// 送金・決済
interface Transfer {
  id: string;
  userId: string;
  fromAccountId: string;
  toAccountId?: string;
  toExternalAccount?: ExternalAccount;
  type: 'internal' | 'ach' | 'wire' | 'instant' | 'crypto' | 'international';
  amount: MoneyAmount;
  fees: Fee[];
  netAmount: MoneyAmount;
  description?: string;
  reference?: string;
  status: 'pending' | 'processing' | 'completed' | 'failed' | 'canceled';
  scheduledAt?: Date;
  processedAt?: Date;
  completedAt?: Date;
  failureReason?: string;
  complianceChecks: ComplianceCheck[];
  createdAt: Date;
  updatedAt: Date;
}

interface ExternalAccount {
  id: string;
  userId: string;
  type: 'bank' | 'card' | 'crypto_wallet' | 'e_wallet';
  name: string;
  accountNumber: string;
  routingNumber?: string;
  bankName?: string;
  isVerified: boolean;
  verificationMethod?: 'micro_deposits' | 'instant' | 'manual';
  lastUsedAt?: Date;
  createdAt: Date;
}

interface Payment {
  id: string;
  userId: string;
  merchantId?: string;
  type: 'qr' | 'nfc' | 'online' | 'p2p';
  amount: MoneyAmount;
  description?: string;
  status: 'pending' | 'completed' | 'failed' | 'refunded';
  paymentMethod: PaymentMethod;
  metadata: any;
  processedAt?: Date;
  createdAt: Date;
}

interface PaymentMethod {
  type: 'account_balance' | 'linked_card' | 'linked_bank' | 'crypto';
  details: any;
}

// セキュリティ・認証
interface DeviceInfo {
  id: string;
  userId: string;
  deviceId: string;
  deviceName: string;
  deviceType: 'ios' | 'android' | 'web';
  osVersion: string;
  appVersion: string;
  ipAddress: string;
  location?: GeoLocation;
  isTrusted: boolean;
  lastUsedAt: Date;
  registeredAt: Date;
}

interface GeoLocation {
  latitude: number;
  longitude: number;
  country: string;
  region: string;
  city: string;
  accuracy: number;
}

interface LoginHistory {
  id: string;
  userId: string;
  deviceId: string;
  ipAddress: string;
  location?: GeoLocation;
  method: 'password' | 'biometric' | 'sso' | 'mfa';
  status: 'success' | 'failed' | 'blocked';
  riskScore: number;
  timestamp: Date;
  failureReason?: string;
}

interface SecurityEvent {
  id: string;
  userId: string;
  type: 'login_failure' | 'device_change' | 'password_change' | 'suspicious_activity' | 'kyc_update';
  severity: 'low' | 'medium' | 'high' | 'critical';
  description: string;
  metadata: any;
  resolved: boolean;
  resolvedAt?: Date;
  createdAt: Date;
}

// コンプライアンス・監査
interface ComplianceFlag {
  id: string;
  userId: string;
  type: 'aml' | 'sanction' | 'pep' | 'high_risk' | 'manual_review';
  severity: 'low' | 'medium' | 'high';
  description: string;
  source: string;
  isActive: boolean;
  reviewRequired: boolean;
  reviewedBy?: string;
  reviewedAt?: Date;
  resolution?: string;
  createdAt: Date;
}

interface ComplianceCheck {
  id: string;
  entityType: 'user' | 'transaction' | 'transfer';
  entityId: string;
  checkType: 'aml' | 'sanction' | 'kyc' | 'fraud' | 'risk_assessment';
  status: 'pending' | 'passed' | 'failed' | 'manual_review';
  score: number;
  riskLevel: 'low' | 'medium' | 'high';
  flags: string[];
  provider: string;
  rawResponse: any;
  reviewedBy?: string;
  reviewedAt?: Date;
  createdAt: Date;
}

interface AuditLog {
  id: string;
  userId?: string;
  action: string;
  resource: string;
  resourceId: string;
  details: any;
  ipAddress: string;
  userAgent: string;
  timestamp: Date;
  sessionId: string;
}

// リスク管理・分析
interface RiskMetrics {
  portfolioId: string;
  volatility: number;
  sharpeRatio: number;
  beta: number;
  var: number; // Value at Risk
  maxDrawdown: number;
  correlationMatrix: { [symbol: string]: number };
  sectorAllocation: { [sector: string]: number };
  geographicAllocation: { [country: string]: number };
  calculatedAt: Date;
}

interface AssetAllocation {
  stocks: number;
  bonds: number;
  crypto: number;
  cash: number;
  alternatives: number;
  domestic: number;
  international: number;
}

interface PriceAlert {
  id: string;
  userId: string;
  instrumentId: string;
  symbol: string;
  type: 'price_above' | 'price_below' | 'percent_change' | 'volume';
  threshold: number;
  isActive: boolean;
  triggered: boolean;
  triggeredAt?: Date;
  createdAt: Date;
  expiresAt?: Date;
}

// ニュース・マーケット情報
interface NewsArticle {
  id: string;
  title: string;
  summary: string;
  content?: string;
  author: string;
  source: string;
  url: string;
  imageUrl?: string;
  publishedAt: Date;
  symbols: string[];
  tags: string[];
  sentiment: 'positive' | 'neutral' | 'negative';
  relevanceScore: number;
}

interface MarketEvent {
  id: string;
  type: 'earnings' | 'dividend' | 'split' | 'ipo' | 'economic_data' | 'fed_meeting';
  title: string;
  description: string;
  instrumentId?: string;
  symbol?: string;
  impact: 'low' | 'medium' | 'high';
  scheduledAt: Date;
  actualValue?: number;
  forecastValue?: number;
  previousValue?: number;
}

// API レスポンス
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
    rateLimit?: {
      remaining: number;
      resetAt: Date;
    };
  };
}

interface PaginatedResponse<T> {
  data: T[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    hasNext: boolean;
    hasPrev: boolean;
    cursor?: string;
  };
}
```

#### データアーキテクチャ
- データモデル設計: PostgreSQL（主要）、ClickHouse（分析）、Redis（リアルタイム）
- データフロー: 外部API → データ正規化 → リアルタイム配信 → 暗号化保存
- 状態管理: Redux Toolkit（取引状態）、React Query（マーケットデータ）、Zustand（UI状態）
- キャッシュ戦略: Redis（価格データ）、メモリキャッシュ（認証トークン）、SQLite（オフライン）

### アプリケーション
<!-- 自然言語での機能仕様とビジネス要件 -->

#### ビジネス要件
- ターゲットユーザー: 20-50歳個人投資家、投資初心者から中級者
- ビジネス価値: 投資の民主化、金融リテラシー向上、資産形成支援
- 成功指標: 月間アクティブユーザー10万人、平均取引額50万円、継続率80%
- 競合分析: 楽天証券、SBI証券、PayPay証券との差別化（UX、手数料、AI活用）

#### 主要機能

##### 口座・認証機能
- **本人確認（KYC）**: 運転免許証認証、マイナンバー連携、ビデオ通話確認
- **リスク診断**: 投資経験・リスク許容度評価、適合性診断
- **生体認証**: Face ID、Touch ID、音声認証
- **多要素認証**: SMS認証、認証アプリ、ハードウェアトークン
- **デバイス認証**: 端末固有ID、位置情報、行動パターン分析

##### 投資・取引機能
- **株式取引**: 現物取引、信用取引、IPO投資、立会外取引
- **投資信託**: 積立投資、スイッチング、分配金再投資
- **ETF取引**: 国内・海外ETF、コモディティETF
- **暗号通貨**: ビットコイン、イーサリアム、アルトコイン取引
- **外国為替**: FXスポット取引、自動売買、スワップポイント

##### ポートフォリオ管理
- **資産状況**: 総資産、損益、資産配分グラフ
- **パフォーマンス分析**: リターン、リスク指標、ベンチマーク比較
- **リバランス提案**: 最適配分提案、自動リバランス
- **税務サポート**: 損益通算、確定申告書類作成
- **目標設定**: 投資目標設定、進捗追跡、アドバイス

##### マーケット情報
- **リアルタイム価格**: 株価、チャート、板情報、出来高
- **マーケットニュース**: 企業決算、経済指標、専門家解説
- **銘柄分析**: 財務データ、テクニカル指標、格付け情報
- **アラート機能**: 価格アラート、ニュースアラート、急騰急落通知
- **スクリーニング**: 条件検索、ランキング、おすすめ銘柄

##### 決済・送金機能
- **入出金**: 銀行振込、即時入金、コンビニ入金
- **P2P送金**: QRコード、電話番号、メールアドレス指定
- **デビット機能**: 投資資金での即時決済
- **自動積立**: 定期積立、余り投資、おつり投資
- **国際送金**: 外貨建て送金、為替レート優遇

#### ユーザー体験
- ユーザーフロー:
  - 新規ユーザー: ダウンロード → 本人確認 → リスク診断 → 初回投資
  - 既存ユーザー: ログイン → ポートフォリオ確認 → 取引実行 → 資産管理
- UI/UXデザイン原則: シンプルなデザイン、直感的操作、データ視覚化重視
- アクセシビリティ: 音声読み上げ対応、大きな文字、ハイコントラスト
- パフォーマンス: 瞬時の価格更新、高速取引実行、オフライン対応

#### 機能要件

##### セキュリティ要件
- 暗号化: AES-256（保存時）、TLS 1.3（通信時）、E2E暗号化（機密データ）
- 認証: 多要素認証必須、生体認証、デバイス証明書
- アクセス制御: RBAC、時間制限、IP制限、地理的制限
- 監査: 全操作ログ、リアルタイム異常検知、法的要求対応

##### コンプライアンス要件
- KYC: 犯罪収益移転防止法対応、継続的顧客管理
- AML: 疑わしい取引報告、制裁リスト照合、PEPs確認
- 適合性: 金融商品販売法、投資家保護、説明義務
- プライバシー: 個人情報保護法、データ保護、同意管理

#### 非機能要件
- パフォーマンス:
  - 価格更新: 100ms以内
  - 注文約定: 1秒以内
  - アプリ起動: 2秒以内
  - チャート表示: 500ms以内
- スケーラビリティ:
  - 同時接続: 50万ユーザー
  - 取引処理: 10万件/秒
  - 価格配信: 100万件/秒
- 可用性: 99.99%稼働率（金融機関水準）
- 信頼性: ゼロデータロス、自動フェイルオーバー、災害復旧

### システム

#### アーキテクチャ設計
- システムアーキテクチャ: マイクロサービス、イベント駆動、CQRS、金融グレード
- アプリケーションアーキテクチャ: Clean Architecture、Hexagonal Architecture
- API設計: GraphQL（リアルタイム）、REST（取引）、WebSocket（価格配信）
- データベースアーキテクチャ: PostgreSQL（主）、ClickHouse（分析）、Redis（キャッシュ）

#### 技術スタック
- プログラミング言語: TypeScript、Go（高頻度取引）
- フレームワーク: React Native 0.73、Expo 50
- 状態管理: Redux Toolkit、RTK Query、React Query
- データベース: PostgreSQL 15、ClickHouse、Redis Cluster
- インフラ: AWS（専用VPC）、Kubernetes、Istio Service Mesh
- 認証: Auth0、AWS Cognito、生体認証SDK
- リアルタイム: WebSocket、Server-Sent Events、gRPC streaming
- セキュリティ: HSM、WAF、DDoS Protection、SIEM

#### 開発環境
```bash
# 環境構築
npm install
npx expo install
npm run setup:env           # 環境変数設定
npm run setup:certs         # SSL証明書設定
npm run setup:hsm           # HSM初期化

# 開発サーバー起動
npx expo start --dev-client # Expo開発サーバー
npm run ios:dev             # iOS開発版
npm run android:dev         # Android開発版

# テスト実行
npm run test                # Jest ユニットテスト
npm run test:security       # セキュリティテスト
npm run test:e2e            # Detox E2Eテスト
npm run test:performance    # パフォーマンステスト
npm run test:penetration    # ペネトレーションテスト

# コード品質チェック
npm run lint                # ESLint + Prettier
npm run type-check          # TypeScript型チェック
npm run security-scan       # セキュリティ脆弱性スキャン
npm run compliance-check    # コンプライアンスチェック

# ビルド・デプロイ
npm run build:production    # 本番ビルド
npm run sign:ios           # iOS署名・公証
npm run sign:android       # Android署名
npm run deploy:store       # ストア申請
```

#### 開発プロセス
- 開発手法: ウォーターフォール（規制対応）+ アジャイル（機能開発）
- バージョン管理: Git Flow、セキュリティブランチ分離
- CI/CD: GitLab CI/CD、セキュリティゲート、段階的デプロイ
- テスト戦略:
  - ユニットテスト: 95%以上カバレッジ
  - セキュリティテスト: OWASP、ペネトレーションテスト
  - パフォーマンステスト: 負荷テスト、ストレステスト
  - 規制テスト: KYC、AML、適合性テスト
- コードレビュー: 4-eyes principle、セキュリティレビュー必須

#### データ管理
```sql
-- 金融グレードデータベース設計
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  phone_number VARCHAR(20) UNIQUE NOT NULL,
  first_name VARCHAR(100) NOT NULL,
  last_name VARCHAR(100) NOT NULL,
  date_of_birth DATE NOT NULL,
  nationality VARCHAR(3) NOT NULL,
  kyc_status INTEGER DEFAULT 0,
  risk_profile JSONB,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  -- 暗号化フィールド
  encrypted_ssn BYTEA, -- Social Security Number
  encrypted_address BYTEA
);

CREATE TABLE accounts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE RESTRICT,
  account_type VARCHAR(50) NOT NULL,
  currency VARCHAR(3) NOT NULL,
  balance DECIMAL(18,8) DEFAULT 0,
  available_balance DECIMAL(18,8) DEFAULT 0,
  frozen_amount DECIMAL(18,8) DEFAULT 0,
  account_number VARCHAR(50) UNIQUE NOT NULL,
  status VARCHAR(20) DEFAULT 'active',
  opened_at TIMESTAMP DEFAULT NOW(),
  last_transaction_at TIMESTAMP
);

CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE RESTRICT,
  account_id UUID REFERENCES accounts(id) ON DELETE RESTRICT,
  instrument_id UUID NOT NULL,
  symbol VARCHAR(20) NOT NULL,
  side VARCHAR(4) NOT NULL CHECK (side IN ('buy', 'sell')),
  order_type VARCHAR(20) NOT NULL,
  quantity DECIMAL(18,8) NOT NULL,
  price DECIMAL(18,8),
  status VARCHAR(20) DEFAULT 'pending',
  placed_at TIMESTAMP DEFAULT NOW(),
  executed_at TIMESTAMP,
  -- リスク管理
  risk_score INTEGER,
  compliance_status VARCHAR(20) DEFAULT 'pending'
);

-- 監査テーブル
CREATE TABLE audit_logs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID,
  action VARCHAR(100) NOT NULL,
  resource_type VARCHAR(50) NOT NULL,
  resource_id UUID,
  old_values JSONB,
  new_values JSONB,
  ip_address INET,
  user_agent TEXT,
  session_id UUID,
  timestamp TIMESTAMP DEFAULT NOW()
);

-- パーティショニング（パフォーマンス）
CREATE TABLE market_data_2024_01 PARTITION OF market_data
FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

-- インデックス設計（金融データ特化）
CREATE INDEX idx_orders_user_status ON orders(user_id, status);
CREATE INDEX idx_orders_symbol_timestamp ON orders(symbol, placed_at);
CREATE INDEX idx_audit_logs_timestamp ON audit_logs(timestamp);
CREATE INDEX idx_market_data_symbol_timestamp ON market_data(symbol, timestamp);
```

- データ保護: 暗号化、匿名化、データマスキング、論理削除
- バックアップ: 継続レプリケーション、ポイントインタイムリカバリ、地理的分散
- データ保持: 取引記録10年、個人情報7年、監査ログ永続保存

#### セキュリティ
- 認証・認可:
  - 多要素認証（MFA）必須
  - 生体認証（顔認証、指紋認証）
  - デバイス証明書認証
  - ハードウェアセキュリティモジュール（HSM）
- データセキュリティ:
  - フィールドレベル暗号化
  - 暗号化キー管理（AWS KMS）
  - データ分類・ラベリング
  - データ損失防止（DLP）
- ネットワークセキュリティ:
  - 専用VPC、プライベートサブネット
  - Web Application Firewall（WAF）
  - DDoS攻撃対策
  - VPN接続必須
- アプリケーションセキュリティ:
  - コード難読化、ルート化検知
  - 証明書ピニング
  - アンチデバッグ、アンチフーク
  - リアルタイム脅威検知

#### 運用・監視
- ログ管理:
  - セキュリティログ: SIEM連携
  - 取引ログ: 規制当局報告用
  - アプリケーションログ: ELK Stack
  - 監査ログ: 改ざん防止、法的要件
- 監視・アラート:
  - リアルタイム監視: Prometheus + Grafana
  - セキュリティ監視: Splunk SIEM
  - 不正取引検知: ML-based anomaly detection
  - インフラ監視: DataDog、PagerDuty
- インシデント対応:
  - 24/7 SOC（Security Operations Center）
  - 自動対応（アカウント停止、取引停止）
  - フォレンジック対応
  - 規制当局報告

#### 環境設定
```bash
# 必須環境変数（本番環境）
DATABASE_URL=postgresql://encrypted_connection
REDIS_CLUSTER_URL=redis://cluster_endpoint
HSM_ENDPOINT=hsm.company.internal
KMS_KEY_ID=arn:aws:kms:region:account:key/key-id

# 認証・セキュリティ
AUTH0_DOMAIN=company.auth0.com
AUTH0_CLIENT_ID=production_client_id
AUTH0_CLIENT_SECRET=encrypted_secret
BIOMETRIC_SDK_KEY=encrypted_sdk_key

# 金融データプロバイダー
MARKET_DATA_API_KEY=encrypted_api_key
REUTERS_API_KEY=encrypted_reuters_key
BLOOMBERG_API_KEY=encrypted_bloomberg_key

# 決済・銀行連携
BANK_API_ENDPOINT=https://secure.bank.com/api
BANK_CLIENT_CERTIFICATE=path/to/client.pem
BANK_PRIVATE_KEY=path/to/private.key

# 規制・コンプライアンス
KYC_PROVIDER_API_KEY=encrypted_kyc_key
AML_SCREENING_API_KEY=encrypted_aml_key
REGULATORY_REPORTING_ENDPOINT=https://regulator.gov.jp/api

# 監視・ログ
SIEM_ENDPOINT=https://siem.company.internal
SYSLOG_SERVER=syslog.company.internal:514
AUDIT_LOG_ENCRYPTION_KEY=audit_encryption_key

# 機能フラグ（規制対応）
ENABLE_CRYPTO_TRADING=true
ENABLE_MARGIN_TRADING=false
ENABLE_INTERNATIONAL_TRANSFERS=true
ENABLE_HIGH_FREQUENCY_TRADING=false

# 緊急時設定
EMERGENCY_STOP_ENDPOINT=https://emergency.company.internal
TRADING_HALT_TRIGGER=true
COMPLIANCE_ALERT_WEBHOOK=https://compliance.company.internal/webhook
```

#### デプロイ・運用
- 環境管理: Development、QA、UAT、Pre-production、Production
- デプロイ戦略: Blue-Green、段階的リリース、緊急停止機能
- 災害復旧: Multi-Region、RTO: 15分、RPO: 0（ゼロデータロス）
- 規制対応: 変更管理、リリース承認プロセス、監査証跡

#### 品質保証
- テスト自動化:
  - ユニットテスト: 95%以上カバレッジ
  - セキュリティテスト: 自動脆弱性スキャン
  - 負荷テスト: 想定トラフィックの300%
  - カオステスト: 本番環境耐性テスト
- 品質メトリクス:
  - 可用性: 99.99%以上
  - セキュリティインシデント: 0件
  - データ損失: 0件
  - 規制違反: 0件

#### 法規制・コンプライアンス
- 金融ライセンス: 金融商品取引業者登録、暗号資産交換業者登録
- 規制対応: 金融商品取引法、資金決済法、犯罪収益移転防止法
- 国際基準: PCI DSS、ISO 27001、SOC2 Type II
- 監査: 内部監査、外部監査、金融庁検査

#### 事業継続・リスク管理
- BCP: 事業継続計画、緊急時対応手順
- リスク管理: 市場リスク、信用リスク、オペレーショナルリスク
- 保険: サイバー保険、役員責任保険、業務過誤保険
- 危機管理: インシデント対応チーム、メディア対応、顧客対応

#### プロジェクト管理
- チーム構成: プロダクトマネージャー、金融システム開発者、セキュリティエンジニア、コンプライアンス担当、QAエンジニア
- 開発期間: 要件定義6ヶ月、開発18ヶ月、テスト6ヶ月、リリース準備3ヶ月
- リスク管理: 規制変更、技術的負債、セキュリティ脆弱性、市場環境変化
- ステークホルダー: 金融庁、取引所、監査法人、投資家、パートナー銀行