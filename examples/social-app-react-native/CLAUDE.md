# CLAUDE.md - ソーシャルメディアアプリ

## 設計

### モデル
<!-- TypeScriptで厳密に型定義したコアデータ構造と関係性 -->
```typescript
// ユーザー・プロフィール管理
interface User {
  id: string;
  username: string;
  email: string;
  phoneNumber?: string;
  displayName: string;
  bio?: string;
  avatar?: Media;
  coverImage?: Media;
  website?: string;
  location?: string;
  birthDate?: Date;
  isVerified: boolean;
  isPrivate: boolean;
  followerCount: number;
  followingCount: number;
  postCount: number;
  settings: UserSettings;
  status: 'active' | 'suspended' | 'deactivated';
  lastSeenAt: Date;
  createdAt: Date;
  updatedAt: Date;
}

interface UserSettings {
  privacy: {
    discoverByEmail: boolean;
    discoverByPhone: boolean;
    showActivity: boolean;
    showOnlineStatus: boolean;
  };
  notifications: {
    push: boolean;
    email: boolean;
    likes: boolean;
    comments: boolean;
    follows: boolean;
    messages: boolean;
    mentions: boolean;
  };
  content: {
    autoplayVideos: boolean;
    qualityPreference: 'low' | 'medium' | 'high' | 'auto';
    sensitiveContent: boolean;
  };
  blocking: {
    blockedUsers: string[];
    mutedUsers: string[];
    mutedWords: string[];
  };
}

// 投稿・コンテンツ
interface Post {
  id: string;
  authorId: string;
  type: 'text' | 'image' | 'video' | 'story';
  content: string;
  media: Media[];
  location?: Location;
  tags: string[];
  mentions: string[];
  visibility: 'public' | 'followers' | 'private';
  commentsEnabled: boolean;
  likesEnabled: boolean;
  sharesEnabled: boolean;
  likeCount: number;
  commentCount: number;
  shareCount: number;
  viewCount: number;
  isEdited: boolean;
  editHistory?: PostEdit[];
  reportCount: number;
  status: 'active' | 'flagged' | 'removed';
  expiresAt?: Date; // Story posts
  createdAt: Date;
  updatedAt: Date;
}

interface PostEdit {
  editedAt: Date;
  content: string;
  media: Media[];
}

interface Media {
  id: string;
  type: 'image' | 'video' | 'audio';
  url: string;
  thumbnailUrl?: string;
  width?: number;
  height?: number;
  duration?: number; // For video/audio
  size: number;
  mimeType: string;
  altText?: string;
  caption?: string;
  metadata?: MediaMetadata;
}

interface MediaMetadata {
  camera?: string;
  location?: Location;
  timestamp?: Date;
  filters?: string[];
  tags?: string[];
}

interface Location {
  name: string;
  address?: string;
  latitude: number;
  longitude: number;
  city?: string;
  country?: string;
}

// ソーシャル機能
interface Follow {
  id: string;
  followerId: string;
  followeeId: string;
  status: 'pending' | 'accepted' | 'rejected';
  createdAt: Date;
}

interface Like {
  id: string;
  userId: string;
  postId: string;
  type: 'like' | 'love' | 'laugh' | 'angry' | 'sad';
  createdAt: Date;
}

interface Comment {
  id: string;
  postId: string;
  authorId: string;
  content: string;
  parentCommentId?: string; // For nested comments
  likeCount: number;
  replyCount: number;
  mentions: string[];
  media?: Media[];
  isEdited: boolean;
  status: 'active' | 'flagged' | 'removed';
  createdAt: Date;
  updatedAt: Date;
}

interface Share {
  id: string;
  userId: string;
  postId: string;
  type: 'repost' | 'quote' | 'story' | 'message';
  content?: string; // For quote shares
  visibility: 'public' | 'followers' | 'private';
  createdAt: Date;
}

// メッセージング
interface Conversation {
  id: string;
  type: 'direct' | 'group';
  name?: string; // For group chats
  avatar?: Media;
  participants: ConversationParticipant[];
  lastMessage?: Message;
  lastActivity: Date;
  isActive: boolean;
  createdAt: Date;
  updatedAt: Date;
}

interface ConversationParticipant {
  userId: string;
  role: 'member' | 'admin';
  joinedAt: Date;
  lastReadAt: Date;
  notifications: boolean;
}

interface Message {
  id: string;
  conversationId: string;
  senderId: string;
  type: 'text' | 'image' | 'video' | 'audio' | 'file' | 'location' | 'contact';
  content?: string;
  media?: Media[];
  replyToId?: string;
  reactions: MessageReaction[];
  readBy: MessageRead[];
  isEdited: boolean;
  isDeleted: boolean;
  deliveredAt?: Date;
  createdAt: Date;
  updatedAt: Date;
}

interface MessageReaction {
  userId: string;
  emoji: string;
  createdAt: Date;
}

interface MessageRead {
  userId: string;
  readAt: Date;
}

// 通知システム
interface Notification {
  id: string;
  userId: string;
  type: 'like' | 'comment' | 'follow' | 'mention' | 'message' | 'story_view' | 'post_tag';
  title: string;
  body: string;
  data: NotificationData;
  isRead: boolean;
  actionUrl?: string;
  createdAt: Date;
}

interface NotificationData {
  actorId?: string;
  postId?: string;
  commentId?: string;
  conversationId?: string;
  [key: string]: any;
}

// フィード・アルゴリズム
interface FeedItem {
  id: string;
  type: 'post' | 'story' | 'suggestion';
  postId?: string;
  userId?: string;
  relevanceScore: number;
  reason: 'following' | 'popular' | 'trending' | 'suggested' | 'nearby';
  seenAt?: Date;
  interactedAt?: Date;
  createdAt: Date;
}

// リアルタイム通信
interface SocketEvent {
  type: 'new_message' | 'typing' | 'online_status' | 'like' | 'comment' | 'follow';
  data: any;
  timestamp: Date;
  room?: string;
}

interface TypingStatus {
  conversationId: string;
  userId: string;
  isTyping: boolean;
  timestamp: Date;
}

interface OnlineStatus {
  userId: string;
  isOnline: boolean;
  lastSeen?: Date;
}

// 検索・発見
interface SearchResult {
  type: 'user' | 'post' | 'hashtag' | 'location';
  id: string;
  score: number;
  highlights?: string[];
}

interface Hashtag {
  tag: string;
  postCount: number;
  trending: boolean;
  category?: string;
}

interface TrendingTopic {
  hashtag: string;
  postCount: number;
  growthRate: number;
  category: 'general' | 'sports' | 'politics' | 'entertainment' | 'technology';
  region?: string;
}

// コンテンツモデレーション
interface Report {
  id: string;
  reporterId: string;
  targetType: 'post' | 'comment' | 'user' | 'message';
  targetId: string;
  reason: 'spam' | 'harassment' | 'hate_speech' | 'violence' | 'sexual_content' | 'copyright' | 'other';
  description?: string;
  evidence?: Media[];
  status: 'pending' | 'reviewed' | 'resolved' | 'dismissed';
  reviewedBy?: string;
  reviewedAt?: Date;
  action?: 'none' | 'warning' | 'content_removal' | 'account_suspension';
  createdAt: Date;
}

// API レスポンス
interface PaginatedResponse<T> {
  data: T[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    hasNext: boolean;
    hasPrev: boolean;
    cursor?: string; // For cursor-based pagination
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
    version: string;
  };
}
```

#### データアーキテクチャ
- データモデル設計: NoSQL (MongoDB)、リアルタイムデータベース、グラフ構造（フォロー関係）
- データフロー: ユーザー行動 → フィード更新 → リアルタイム通知 → エンゲージメント分析
- 状態管理: Redux Toolkit（グローバル状態）、React Query（サーバー状態）、Zustand（UI状態）
- キャッシュ戦略: Redis（セッション、フィード）、CDN（メディア）、SQLite（オフライン）

### アプリケーション
<!-- 自然言語での機能仕様とビジネス要件 -->

#### ビジネス要件
- ターゲットユーザー: 18-35歳、コンテンツクリエイター、若年層コミュニティ
- ビジネス価値: エンゲージメント創出、コミュニティ形成、デジタル広告収益
- 成功指標: DAU 50万人、投稿エンゲージメント率3%、セッション時間30分/日
- 競合分析: Instagram、TikTok、Twitterとの差別化（プライバシー重視、AI機能）

#### 主要機能

##### コア機能
- **ユーザー管理**: 会員登録、プロフィール設定、アカウント認証、プライバシー設定
- **投稿機能**: テキスト・画像・動画投稿、ストーリー、ライブ配信
- **タイムライン**: パーソナライズドフィード、フォロー投稿、人気投稿
- **ソーシャル機能**: フォロー・フォロワー、いいね、コメント、シェア、メンション
- **メッセージング**: 1対1チャット、グループチャット、音声・ビデオ通話
- **検索・発見**: ユーザー検索、ハッシュタグ、トレンド、おすすめユーザー

##### エンゲージメント機能
- **通知システム**: プッシュ通知、アプリ内通知、メール通知
- **ライブ機能**: ライブストリーミング、リアルタイムコメント
- **ストーリー**: 24時間限定投稿、閲覧者確認
- **リアクション**: 絵文字リアクション、カスタムスタンプ
- **コレクション**: 投稿保存、プレイリスト作成

##### コミュニティ機能
- **グループ**: プライベートグループ、トピック別コミュニティ
- **イベント**: イベント作成・参加、カレンダー連携
- **コラボレーション**: 共同投稿、リミックス機能
- **チャレンジ**: ハッシュタグチャレンジ、コンテスト

#### ユーザー体験
- ユーザーフロー:
  - 新規ユーザー: サインアップ → プロフィール設定 → 興味フォロー → フィード体験
  - 既存ユーザー: アプリ起動 → フィード閲覧 → 投稿作成/エンゲージメント → 通知確認
- UI/UXデザイン原則: モバイルファースト、直感的操作、ダークモード対応、アクセシビリティ
- ナビゲーション: タブベースナビゲーション、ジェスチャー操作、音声コマンド
- パフォーマンス: 瞬間ローディング、プリローディング、オフライン対応

#### 機能要件

##### コンテンツ管理
- バリデーションルール:
  - 投稿文字数: 2000文字以内
  - 画像: 最大10枚、20MB以内、JPG/PNG
  - 動画: 最大60秒、100MB以内、MP4
  - プロフィール: ユーザー名は英数字とアンダースコア
- 権限・ロール:
  - 一般ユーザー: 投稿・コメント・フォロー
  - 認証ユーザー: 拡張機能、優先表示
  - モデレーター: コンテンツ審査、ユーザー管理
  - 管理者: システム全般管理

##### プライバシー・セキュリティ
- プライベートアカウント: フォロー承認制、投稿非公開
- ブロック・ミュート: ユーザーブロック、キーワードミュート
- データ保護: E2E暗号化（DM）、データ削除権、エクスポート機能
- 年齢制限: 18歳未満の機能制限、保護者承認

#### 非機能要件
- パフォーマンス:
  - アプリ起動: 2秒以内
  - フィード読み込み: 1秒以内
  - 画像表示: 500ms以内
  - リアルタイム通信: 100ms以内
- スケーラビリティ:
  - 同時接続: 100万ユーザー
  - 日次投稿: 1000万投稿
  - ストレージ: ペタバイト級メディア
- 可用性: 99.9%稼働率、グローバル分散配信
- 信頼性: リアルタイム同期、データ整合性、自動復旧

### システム

#### アーキテクチャ設計
- システムアーキテクチャ: マイクロサービス、イベント駆動、CQRS
- アプリケーションアーキテクチャ: Clean Architecture、Redux Pattern
- API設計: GraphQL、REST（一部）、WebSocket、Server-Sent Events
- データベースアーキテクチャ: MongoDB、Redis、Elasticsearch、PostgreSQL（分析）

#### 技術スタック
- プログラミング言語: TypeScript、Swift（iOS）、Kotlin（Android）
- フレームワーク: React Native 0.73、Expo 50
- 状態管理: Redux Toolkit、RTK Query、React Query
- データベース: MongoDB Atlas、Redis Cloud、Elasticsearch
- インフラ: AWS（EKS、S3、CloudFront）、Cloudflare（CDN）
- 認証: Auth0、Firebase Auth、生体認証
- リアルタイム: Socket.io、WebRTC、AWS IoT Core
- プッシュ通知: Firebase Cloud Messaging、APNs
- メディア処理: FFmpeg、ImageKit、AWS MediaConvert

#### 開発環境
```bash
# 環境構築
npm install
npx expo install
npx pod-install ios

# 開発サーバー起動
npx expo start           # Expo開発サーバー
npm run ios             # iOS シミュレータ
npm run android         # Android エミュレータ

# テスト実行
npm run test            # Jest ユニットテスト
npm run test:e2e        # Detox E2Eテスト
npm run test:integration # 統合テスト

# コード品質チェック
npm run lint            # ESLint + Prettier
npm run type-check      # TypeScript型チェック
npm run audit           # セキュリティ監査

# ビルド・リリース
npm run build:ios       # iOS ビルド
npm run build:android   # Android ビルド
eas build --platform all # Expo Application Services
eas submit              # ストア申請
```

#### 開発プロセス
- 開発手法: アジャイル、2週間スプリント、継続的デリバリー
- バージョン管理: Git Flow、feature/develop/main ブランチ
- CI/CD: GitHub Actions、EAS Build、CodePush
- テスト戦略:
  - ユニットテスト: Jest（ビジネスロジック）
  - コンポーネントテスト: React Native Testing Library
  - 統合テスト: Detox（ユーザーフロー）
  - 性能テスト: Flipper、Reactotron
- コードレビュー: PR必須、ペアプログラミング推奨

#### データ管理
```javascript
// MongoDB スキーマ例
const userSchema = new Schema({
  _id: { type: String, required: true },
  username: { type: String, unique: true, required: true },
  email: { type: String, unique: true, required: true },
  displayName: { type: String, required: true },
  bio: String,
  avatar: {
    url: String,
    width: Number,
    height: Number
  },
  followerCount: { type: Number, default: 0 },
  followingCount: { type: Number, default: 0 },
  settings: {
    privacy: {
      isPrivate: { type: Boolean, default: false },
      showActivity: { type: Boolean, default: true }
    },
    notifications: {
      push: { type: Boolean, default: true },
      email: { type: Boolean, default: true }
    }
  },
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now }
});

const postSchema = new Schema({
  _id: { type: String, required: true },
  authorId: { type: String, ref: 'User', required: true },
  content: { type: String, maxlength: 2000 },
  media: [{
    type: { type: String, enum: ['image', 'video'] },
    url: String,
    thumbnailUrl: String,
    width: Number,
    height: Number,
    duration: Number
  }],
  tags: [String],
  mentions: [{ type: String, ref: 'User' }],
  likeCount: { type: Number, default: 0 },
  commentCount: { type: Number, default: 0 },
  shareCount: { type: Number, default: 0 },
  visibility: { type: String, enum: ['public', 'followers', 'private'], default: 'public' },
  createdAt: { type: Date, default: Date.now },
  updatedAt: { type: Date, default: Date.now }
});

// インデックス設計
db.users.createIndex({ "username": 1 });
db.users.createIndex({ "email": 1 });
db.posts.createIndex({ "authorId": 1, "createdAt": -1 });
db.posts.createIndex({ "tags": 1 });
db.posts.createIndex({ "createdAt": -1 });
```

- データ保護: MongoDB暗号化、GDPR準拠データ処理、Right to be forgotten
- バックアップ: 自動バックアップ、地理的分散、ポイントインタイムリカバリ
- データ保持: 投稿データ無期限、アクティビティログ1年、分析データ2年

#### セキュリティ
- 認証・認可: OAuth 2.0、JWT、生体認証、2FA
- データセキュリティ:
  - 保存時暗号化: AES-256
  - 通信時暗号化: TLS 1.3、Certificate Pinning
  - E2E暗号化: Signal Protocol（DM）
- API セキュリティ: レート制限、CORS、CSP、API キー管理
- モバイルセキュリティ: App Transport Security、Root/Jailbreak検出、Code Obfuscation
- プライバシー: データ最小化、匿名化、同意管理

#### 運用・監視
- ログ管理:
  - アプリケーションログ: CloudWatch Logs
  - ユーザー行動ログ: Mixpanel、Amplitude
  - エラーログ: Sentry、Bugsnag
  - パフォーマンスログ: New Relic、Datadog
- 監視・アラート:
  - アプリ監視: Firebase Performance
  - インフラ監視: AWS CloudWatch
  - リアルタイム監視: Socket.io管理画面
  - ビジネス監視: カスタムダッシュボード
- A/Bテスト: Firebase Remote Config、Optimizely
- 分析: Firebase Analytics、カスタムイベント追跡

#### 環境設定
```bash
# 必須環境変数
EXPO_PUBLIC_API_URL=https://api.yourapp.com
EXPO_PUBLIC_WS_URL=wss://ws.yourapp.com

# 認証設定
EXPO_PUBLIC_AUTH0_DOMAIN=yourapp.auth0.com
EXPO_PUBLIC_AUTH0_CLIENT_ID=your_auth0_client_id

# Firebase設定
EXPO_PUBLIC_FIREBASE_API_KEY=your_firebase_api_key
EXPO_PUBLIC_FIREBASE_PROJECT_ID=your_project_id
EXPO_PUBLIC_FIREBASE_APP_ID=your_app_id

# メディア処理
EXPO_PUBLIC_IMAGEKIT_URL=https://ik.imagekit.io/yourapp
EXPO_PUBLIC_IMAGEKIT_PUBLIC_KEY=your_imagekit_public_key

# 分析・監視
EXPO_PUBLIC_MIXPANEL_TOKEN=your_mixpanel_token
EXPO_PUBLIC_SENTRY_DSN=your_sentry_dsn

# 機能フラグ
EXPO_PUBLIC_ENABLE_STORIES=true
EXPO_PUBLIC_ENABLE_LIVE_STREAMING=true
EXPO_PUBLIC_ENABLE_VIDEO_CALLS=false

# 外部サービス連携
EXPO_PUBLIC_GOOGLE_MAPS_API_KEY=your_google_maps_key
EXPO_PUBLIC_AGORA_APP_ID=your_agora_app_id
```

#### デプロイ・運用
- 環境管理: Development、Staging、Production、Pre-production
- デプロイ戦略: Expo Application Services、CodePush（ホットアップデート）
- ストア配信: App Store Connect、Google Play Console、段階的ロールアウト
- スケーリング: Kubernetes、Auto Scaling、Load Balancing
- 災害復旧: Multi-Region、RTO: 30分、RPO: 5分

#### 品質保証
- テスト自動化:
  - ユニットテスト: 85%以上のカバレッジ
  - E2Eテスト: クリティカルフロー（投稿、メッセージ、決済）
  - 性能テスト: Maestro、Detox Performance
  - セキュリティテスト: OWASP Mobile Top 10
- 品質メトリクス:
  - クラッシュ率: 0.1%以下
  - ANR率: 0.05%以下
  - アプリサイズ: 100MB以下
- ベータテスト: TestFlight、Google Play Console Internal Testing

#### コンテンツモデレーション
- 自動検出: AI画像・テキスト分析、不適切コンテンツフィルタ
- 人的レビュー: 24時間モデレーションチーム、エスカレーション
- ユーザー報告: 簡単報告機能、迅速対応
- 透明性: モデレーションポリシー公開、異議申し立て制度

#### 国際化・ローカライゼーション
- 多言語対応: 日本語、英語、韓国語、中国語（簡体字・繁体字）
- 地域適応: タイムゾーン、通貨、文化的配慮
- 法規制対応: GDPR、CCPA、個人情報保護法、未成年者保護法

#### プロジェクト管理
- チーム構成: プロダクトマネージャー、iOS/Android開発者、バックエンド開発者、デザイナー、QAエンジニア
- タイムライン: MVP開発6ヶ月、ベータ版3ヶ月、正式リリース
- リスク管理: 技術的負債、スケーラビリティ、法規制変更
- ステークホルダー: 経営陣、投資家、ユーザーコミュニティ、広告主