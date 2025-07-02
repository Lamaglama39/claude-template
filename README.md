# Claude Code テンプレート

Claude Codeプロジェクト用のテンプレートファイル集です。

## 使用方法

1. 新しいプロジェクトを開始する際に、このディレクトリのファイルをプロジェクトルートにコピー
2. `CLAUDE.md`をプロジェクトに合わせてカスタマイズ

## ファイル構成

- `CLAUDE.md` - Claude Code用プロジェクト指示ファイル
- `.gitignore` - 基本的なファイル除外設定

## CLAUDE.mdカスタマイズガイド

### プラットフォーム別

#### Webアプリケーション
```typescript
// モデル例
interface WebApp {
  routing: 'SPA' | 'MPA';
  rendering: 'CSR' | 'SSR' | 'SSG';
  responsive: boolean;
}
```
- 技術スタック: React/Vue/Angular + Node.js/Django/Rails
- 考慮事項: SEO、パフォーマンス、アクセシビリティ、ブラウザ互換性

#### モバイルアプリ
```typescript
interface MobileApp {
  platform: 'iOS' | 'Android' | 'CrossPlatform';
  framework: 'Native' | 'ReactNative' | 'Flutter' | 'Ionic';
}
```
- 技術スタック: Swift/Kotlin、React Native、Flutter
- 考慮事項: デバイス固有機能、パフォーマンス、ストア申請

#### デスクトップアプリケーション
```typescript
interface DesktopApp {
  platform: 'Windows' | 'macOS' | 'Linux' | 'CrossPlatform';
  framework: 'Electron' | 'Tauri' | 'Native';
}
```
- 技術スタック: Electron、Tauri、Qt、.NET
- 考慮事項: OS統合、配布方法、セキュリティ

### 用途・機能別

#### ビジネス・生産性ツール
```typescript
interface BusinessTool {
  userManagement: boolean;
  reporting: boolean;
  integrations: string[];
  compliance: string[];
}
```
- 重要機能: 認証・認可、データエクスポート、API統合
- 考慮事項: セキュリティ、スケーラビリティ、法規制対応

#### SNS・コミュニケーション
```typescript
interface SocialApp {
  realtime: boolean;
  media: ('text' | 'image' | 'video' | 'audio')[];
  moderation: boolean;
}
```
- 重要機能: リアルタイム通信、メディア処理、モデレーション
- 考慮事項: プライバシー、スケーラビリティ、コンテンツ管理

#### Eコマース
```typescript
interface EcommerceApp {
  payment: string[];
  inventory: boolean;
  shipping: boolean;
  analytics: boolean;
}
```
- 重要機能: 決済処理、在庫管理、配送管理
- 考慮事項: PCI DSS準拠、パフォーマンス、SEO

#### 教育・学習アプリ
```typescript
interface EducationApp {
  content: ('video' | 'quiz' | 'assignment' | 'forum')[];
  progress: boolean;
  certificate: boolean;
}
```
- 重要機能: コンテンツ管理、進捗追跡、評価システム
- 考慮事項: アクセシビリティ、オフライン対応

#### 金融・決済アプリ
```typescript
interface FinanceApp {
  transactions: boolean;
  encryption: boolean;
  compliance: ('PCI' | 'SOX' | 'GDPR')[];
}
```
- 重要機能: セキュア決済、暗号化、監査ログ
- 考慮事項: セキュリティ、法規制、可用性

### 技術・開発者向け

#### 開発ツール・IDE
```typescript
interface DevTool {
  codeAnalysis: boolean;
  debugging: boolean;
  versionControl: boolean;
  plugins: boolean;
}
```
- 重要機能: シンタックスハイライト、デバッグ、拡張性
- 考慮事項: パフォーマンス、プラグイン生態系

#### 監視・分析ツール
```typescript
interface MonitoringTool {
  metrics: string[];
  alerting: boolean;
  dashboard: boolean;
  retention: number;
}
```
- 重要機能: データ収集、アラート、可視化
- 考慮事項: スケーラビリティ、データ保存期間

### システム・インフラ

#### セキュリティアプリ
```typescript
interface SecurityApp {
  encryption: boolean;
  authentication: '2FA' | 'MFA' | 'SSO';
  audit: boolean;
}
```
- 重要機能: 暗号化、多要素認証、監査ログ
- 考慮事項: ゼロトラスト、脅威検知

#### システム管理ツール
```typescript
interface SystemTool {
  monitoring: boolean;
  automation: boolean;
  configuration: boolean;
}
```
- 重要機能: リソース監視、自動化、設定管理
- 考慮事項: 可用性、スケーラビリティ

### 特定業界向け

#### 医療・ヘルスケア
```typescript
interface HealthcareApp {
  patientData: boolean;
  compliance: ('HIPAA' | 'GDPR')[];
  integration: ('EHR' | 'FHIR')[];
}
```
- 重要機能: 患者データ管理、医療機器連携
- 考慮事項: HIPAA準拠、データセキュリティ

#### 不動産アプリ
```typescript
interface RealEstateApp {
  listings: boolean;
  maps: boolean;
  mortgage: boolean;
  virtual: boolean;
}
```
- 重要機能: 物件検索、地図連携、ローン計算
- 考慮事項: 地理情報、画像処理

## カスタマイズ例

### React Webアプリの場合
```bash
# システム > 開発環境
npm install
npm run dev
npm test
npm run build

# システム > 技術スタック
- フレームワーク: React 18 + TypeScript
- データベース: PostgreSQL
- 認証: NextAuth.js
- スタイリング: Tailwind CSS
```

### モバイルアプリの場合
```bash
# システム > 開発環境
npx react-native start
npx react-native run-ios
npx react-native run-android
npm test

# システム > 技術スタック
- フレームワーク: React Native
- 状態管理: Redux Toolkit
- ナビゲーション: React Navigation
```