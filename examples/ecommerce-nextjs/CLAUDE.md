# CLAUDE.md - Eコマースサイト

## 設計

### モデル
<!-- TypeScriptで厳密に型定義したコアデータ構造と関係性 -->
```typescript
// ユーザー管理
interface User {
  id: string;
  email: string;
  name: string;
  phone?: string;
  emailVerified: boolean;
  role: 'customer' | 'admin' | 'staff';
  preferences: UserPreferences;
  addresses: Address[];
  createdAt: Date;
  updatedAt: Date;
}

interface UserPreferences {
  currency: 'JPY' | 'USD' | 'EUR';
  language: 'ja' | 'en';
  newsletter: boolean;
  notifications: {
    orderStatus: boolean;
    promotions: boolean;
    stockAlert: boolean;
  };
}

interface Address {
  id: string;
  userId: string;
  type: 'shipping' | 'billing';
  isDefault: boolean;
  firstName: string;
  lastName: string;
  company?: string;
  address1: string;
  address2?: string;
  city: string;
  state: string;
  postalCode: string;
  country: string;
  phone?: string;
}

// 商品管理
interface Product {
  id: string;
  name: string;
  slug: string;
  description: string;
  shortDescription: string;
  sku: string;
  price: number;
  compareAtPrice?: number;
  costPerItem?: number;
  trackQuantity: boolean;
  quantity: number;
  allowBackorder: boolean;
  weight?: number;
  dimensions?: ProductDimensions;
  categoryId: string;
  brand: string;
  tags: string[];
  images: ProductImage[];
  variants: ProductVariant[];
  seoTitle?: string;
  seoDescription?: string;
  status: 'draft' | 'active' | 'archived';
  featured: boolean;
  createdAt: Date;
  updatedAt: Date;
}

interface ProductVariant {
  id: string;
  productId: string;
  name: string;
  sku: string;
  price: number;
  compareAtPrice?: number;
  quantity: number;
  options: { [key: string]: string }; // color: "Red", size: "M"
  image?: string;
  weight?: number;
}

interface ProductImage {
  id: string;
  url: string;
  altText: string;
  position: number;
}

interface ProductDimensions {
  length: number;
  width: number;
  height: number;
  unit: 'cm' | 'in';
}

interface Category {
  id: string;
  name: string;
  slug: string;
  description?: string;
  parentId?: string;
  image?: string;
  seoTitle?: string;
  seoDescription?: string;
  position: number;
  isVisible: boolean;
  createdAt: Date;
  updatedAt: Date;
}

// ショッピング・注文
interface Cart {
  id: string;
  userId?: string; // ゲストの場合はnull
  sessionId: string;
  items: CartItem[];
  subtotal: number;
  taxAmount: number;
  shippingAmount: number;
  discountAmount: number;
  total: number;
  currency: string;
  expiresAt: Date;
  createdAt: Date;
  updatedAt: Date;
}

interface CartItem {
  id: string;
  cartId: string;
  productId: string;
  variantId?: string;
  quantity: number;
  price: number;
  total: number;
}

interface Order {
  id: string;
  orderNumber: string;
  userId?: string;
  email: string;
  status: 'pending' | 'confirmed' | 'processing' | 'shipped' | 'delivered' | 'cancelled' | 'refunded';
  paymentStatus: 'pending' | 'paid' | 'failed' | 'refunded' | 'partially_refunded';
  fulfillmentStatus: 'unfulfilled' | 'partial' | 'fulfilled';
  currency: string;
  subtotal: number;
  taxAmount: number;
  shippingAmount: number;
  discountAmount: number;
  total: number;
  items: OrderItem[];
  shippingAddress: Address;
  billingAddress: Address;
  paymentMethod: PaymentMethod;
  shippingMethod: ShippingMethod;
  notes?: string;
  trackingNumber?: string;
  tags: string[];
  createdAt: Date;
  updatedAt: Date;
}

interface OrderItem {
  id: string;
  orderId: string;
  productId: string;
  variantId?: string;
  name: string;
  sku: string;
  quantity: number;
  price: number;
  total: number;
  fulfillmentStatus: 'unfulfilled' | 'fulfilled' | 'cancelled';
}

// 決済・配送
interface PaymentMethod {
  type: 'stripe' | 'paypal' | 'bank_transfer' | 'cod';
  stripePaymentIntentId?: string;
  last4?: string;
  brand?: string;
  gateway: string;
}

interface ShippingMethod {
  id: string;
  name: string;
  description: string;
  price: number;
  estimatedDays: number;
  carrier: string;
  trackingSupported: boolean;
}

// レビュー・評価
interface Review {
  id: string;
  productId: string;
  userId: string;
  orderId: string;
  rating: number; // 1-5
  title: string;
  content: string;
  verified: boolean;
  helpful: number;
  images?: string[];
  status: 'pending' | 'approved' | 'rejected';
  createdAt: Date;
  updatedAt: Date;
}

// プロモーション
interface Coupon {
  id: string;
  code: string;
  type: 'fixed' | 'percentage';
  value: number;
  description: string;
  minOrderAmount?: number;
  maxDiscountAmount?: number;
  usageLimit?: number;
  usageCount: number;
  isActive: boolean;
  startsAt: Date;
  expiresAt: Date;
  createdAt: Date;
  updatedAt: Date;
}

// API レスポンス
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
  };
  meta?: {
    timestamp: string;
    requestId: string;
  };
}
```

#### データアーキテクチャ
- データモデル設計: 正規化されたリレーショナル設計、商品バリエーション対応
- データフロー: 商品閲覧 → カート追加 → 決済 → 注文管理 → 配送追跡
- 状態管理: Zustand（カート状態）、React Query（サーバー状態）
- キャッシュ戦略: Redis（セッション、商品データ）、CDN（画像）、ISR（商品ページ）

### アプリケーション
<!-- 自然言語での機能仕様とビジネス要件 -->

#### ビジネス要件
- ターゲットユーザー: 中小企業・個人事業主向けBtoC ECサイト
- ビジネス価値: オンライン販売による売上拡大、24時間営業、顧客データ蓄積
- 成功指標: CVR 2%以上、平均注文単価向上、リピート率30%以上
- 競合分析: Shopify、BASE、STORES.jpとの差別化（カスタマイズ性、手数料）

#### 主要機能

##### フロントエンド（顧客向け）
- **商品カタログ**: カテゴリ別商品一覧、検索・フィルタ機能、商品詳細
- **ショッピングカート**: 商品追加・削除、数量変更、保存機能
- **ユーザー管理**: 会員登録・ログイン、プロフィール管理、注文履歴
- **決済**: Stripe決済、ゲスト購入、複数配送先対応
- **レビューシステム**: 商品レビュー投稿・閲覧、評価機能

##### 管理画面（運営者向け）
- **商品管理**: 商品登録・編集、在庫管理、バリエーション設定
- **注文管理**: 注文一覧、ステータス管理、配送管理
- **顧客管理**: 顧客データ閲覧、コミュニケーション履歴
- **売上分析**: 売上レポート、人気商品分析、顧客分析
- **プロモーション**: クーポン作成、セール設定

#### ユーザー体験
- ユーザーフロー: 
  - 商品発見（SEO、SNS） → 商品閲覧 → カート追加 → 購入検討 → 決済 → 購入完了
  - 会員登録 → プロフィール設定 → リピート購入 → レビュー投稿
- UI/UXデザイン原則: モバイルファースト、直感的なナビゲーション、高速表示
- アクセシビリティ: WCAG 2.1 AA準拠、キーボード操作対応、スクリーンリーダー対応
- レスポンシブデザイン: モバイル（375px-）、タブレット（768px-）、デスクトップ（1024px-）
- 国際化: 日本語・英語対応、多通貨対応（JPY, USD）

#### 機能要件

##### 商品・在庫管理
- バリデーションルール: 
  - 商品名: 必須、1-100文字
  - 価格: 必須、正数
  - SKU: 必須、ユニーク
  - 在庫数: 0以上の整数
- 権限・ロール: 
  - 管理者: 全機能アクセス
  - スタッフ: 商品・注文管理のみ
  - 顧客: 購入機能のみ
- 在庫管理: リアルタイム在庫更新、在庫切れ通知、予約注文対応

##### 決済・注文処理
- 決済手段: Stripe（クレジットカード）、代金引換、銀行振込
- 配送: 日本郵便、ヤマト運輸、佐川急便との連携
- 税計算: 消費税自動計算、軽減税率対応
- 注文処理: 在庫確保 → 決済処理 → 注文確定 → 配送準備

#### 非機能要件
- パフォーマンス: 
  - ページ読み込み: 2秒以内（LCP）
  - API レスポンス: 500ms以内
  - 画像最適化: WebP、適応的サイズ配信
- スケーラビリティ: 
  - 同時接続: 1,000ユーザー
  - 月間PV: 100万PV対応
  - 注文処理: 1日1,000注文対応
- 可用性: 99.9%稼働率、計画メンテナンス月1回以下
- 信頼性: 自動バックアップ、災害復旧計画、データ整合性保証

### システム

#### アーキテクチャ設計
- システムアーキテクチャ: モノリス（スモールスタート）、将来的マイクロサービス移行
- アプリケーションアーキテクチャ: Next.js App Router、Clean Architecture適用
- API設計: RESTful API、OpenAPI仕様書、バージョニング（/api/v1）
- データベースアーキテクチャ: PostgreSQL、読み書き分離、レプリケーション

#### 技術スタック
- プログラミング言語: TypeScript 5.0+
- フレームワーク: Next.js 14 (App Router)、React 18
- データベース: PostgreSQL 15、Prisma ORM
- インフラ: Vercel（ホスティング）、Railway（DB）、Cloudinary（画像）
- 認証: NextAuth.js v5、OAuth（Google, Apple）
- 決済: Stripe Payment Links、Stripe Checkout
- メール配信: Resend、トランザクションメール対応
- 監視: Sentry（エラー追跡）、Vercel Analytics

#### 開発環境
```bash
# 環境構築
npm install
npx prisma generate
npx prisma db push

# 開発サーバー起動
npm run dev            # Next.js開発サーバー
npm run studio         # Prisma Studio

# テスト実行
npm run test           # Jest + React Testing Library
npm run test:e2e       # Playwright E2Eテスト

# コード品質チェック
npm run lint           # ESLint + Prettier
npm run type-check     # TypeScript型チェック

# ビルド・デプロイ
npm run build          # プロダクションビルド
npm run start          # プロダクションサーバー
vercel deploy          # Vercelデプロイ
```

#### 開発プロセス
- 開発手法: アジャイル開発、2週間スプリント
- バージョン管理: Git Flow、feature/staging/main ブランチ
- CI/CD: GitHub Actions、自動テスト・デプロイ
- テスト戦略: 
  - 単体テスト: Jest（ユーティリティ関数）
  - 統合テスト: React Testing Library（コンポーネント）
  - E2Eテスト: Playwright（購入フロー）
- コードレビュー: PR必須、2名承認制

#### データ管理
```sql
-- 主要テーブル設計
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email VARCHAR(255) UNIQUE NOT NULL,
  name VARCHAR(255) NOT NULL,
  phone VARCHAR(20),
  email_verified BOOLEAN DEFAULT FALSE,
  role VARCHAR(20) DEFAULT 'customer',
  preferences JSONB,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE products (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  slug VARCHAR(255) UNIQUE NOT NULL,
  description TEXT,
  sku VARCHAR(100) UNIQUE NOT NULL,
  price DECIMAL(10,2) NOT NULL,
  quantity INTEGER DEFAULT 0,
  category_id UUID REFERENCES categories(id),
  images JSONB,
  variants JSONB,
  status VARCHAR(20) DEFAULT 'draft',
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE orders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  order_number VARCHAR(50) UNIQUE NOT NULL,
  user_id UUID REFERENCES users(id),
  email VARCHAR(255) NOT NULL,
  status VARCHAR(20) DEFAULT 'pending',
  payment_status VARCHAR(20) DEFAULT 'pending',
  currency VARCHAR(3) DEFAULT 'JPY',
  subtotal DECIMAL(10,2) NOT NULL,
  tax_amount DECIMAL(10,2) DEFAULT 0,
  shipping_amount DECIMAL(10,2) DEFAULT 0,
  total DECIMAL(10,2) NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

-- インデックス設計
CREATE INDEX idx_products_status ON products(status);
CREATE INDEX idx_products_category ON products(category_id);
CREATE INDEX idx_orders_user ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_orders_created ON orders(created_at);
```

- データ保護: PostgreSQL暗号化、PII匿名化、GDPR準拠
- バックアップ: 自動日次バックアップ、ポイントインタイムリカバリ
- データ保持: 注文データ7年保管、アクセスログ1年保管

#### セキュリティ
- 認証・認可: NextAuth.js、JWT トークン、RBAC（ロールベースアクセス制御）
- データセキュリティ: 
  - 保存時暗号化（PostgreSQL、Prisma）
  - 通信時暗号化（HTTPS、TLS 1.3）
  - 決済情報はStripeで管理（PCI DSS準拠）
- 通信セキュリティ: HTTPS強制、CSP設定、CSRF保護
- セキュリティテスト: OWASP ZAP、Snyk脆弱性チェック
- コンプライアンス: 特定商取引法、個人情報保護法、Cookie同意

#### 運用・監視
- ログ管理: 
  - アプリケーションログ: Vercel Functions Logs
  - アクセスログ: Vercel Analytics
  - エラーログ: Sentry
  - 決済ログ: Stripe Dashboard
- 監視・アラート: 
  - アップタイム監視: Vercel Monitoring
  - パフォーマンス監視: Web Vitals
  - エラー率監視: Sentry Alerts
  - 売上監視: 日次売上レポート
- パフォーマンス監視: Core Web Vitals、Real User Monitoring
- エラーハンドリング: Sentry統合、エラー境界、Graceful degradation

#### 環境設定
```bash
# 必須環境変数
DATABASE_URL=postgresql://user:pass@localhost:5432/ecommerce
NEXTAUTH_SECRET=your_nextauth_secret
NEXTAUTH_URL=http://localhost:3000

# Stripe設定
STRIPE_SECRET_KEY=sk_test_...
STRIPE_PUBLISHABLE_KEY=pk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...

# 画像管理
CLOUDINARY_CLOUD_NAME=your_cloud_name
CLOUDINARY_API_KEY=your_api_key
CLOUDINARY_API_SECRET=your_api_secret

# メール配信
RESEND_API_KEY=your_resend_api_key
FROM_EMAIL=noreply@yourstore.com

# 機能フラグ
ENABLE_REVIEWS=true
ENABLE_COUPONS=true
ENABLE_GUEST_CHECKOUT=true

# 外部サービス連携
GOOGLE_CLIENT_ID=your_google_client_id
GOOGLE_CLIENT_SECRET=your_google_client_secret
```

#### デプロイ・運用
- 環境管理: Development（ローカル）、Staging（Vercel Preview）、Production（Vercel）
- デプロイ戦略: GitHub連携自動デプロイ、プレビューデプロイ
- スケーリング: Vercel Serverless Functions、自動スケーリング
- 災害復旧: 
  - RTO: 1時間（復旧時間目標）
  - RPO: 15分（復旧ポイント目標）
  - マルチリージョンバックアップ

#### 品質保証
- テスト自動化: 
  - ユニットテスト: 80%以上のカバレッジ
  - E2Eテスト: 重要フロー（購入、決済）のテスト
  - 性能テスト: Lighthouse CI
- 品質メトリクス: 
  - コード品質: SonarQube
  - セキュリティ: Snyk脆弱性スキャン
  - パフォーマンス: Core Web Vitals

#### SEO・マーケティング
- SEO最適化:
  - メタタグ最適化（商品、カテゴリ）
  - 構造化データ（JSON-LD）
  - サイトマップ自動生成
  - ページ速度最適化
- 分析ツール: Google Analytics 4、Google Search Console
- マーケティング: メールマーケティング、リターゲティング広告

#### 法的要件・コンプライアンス
- 特定商取引法: 事業者情報表示、返品・交換規約
- 個人情報保護法: プライバシーポリシー、データ削除権対応
- 消費者契約法: 不当勧誘防止、クーリングオフ対応
- 景品表示法: 価格表示、誇大広告防止