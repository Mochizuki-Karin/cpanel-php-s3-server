# PHP S3サーバー セキュリティガイド

このドキュメントでは、PHP S3サーバーのセキュリティ機能、ベストプラクティス、およびセキュリティ設定の推奨事項について詳しく説明します。

## セキュリティ概要

PHP S3サーバーは、認証、認可、データ保護、アクセス制御を含む多層セキュリティアーキテクチャを採用しています。これは軽量な実装ですが、エンタープライズレベルのセキュリティ機能を提供します。

## 認証メカニズム

### 1. 複数の認証方法

PHP S3サーバーは、柔軟なセキュリティオプションを提供するために3つの認証方法をサポートしています。

#### APIキー認証（推奨）
- カスタムAPIキーを使用して認証
- キーはサーバー側に保存され、URLでは転送されません
- キーのローテーションと失効をサポート

```http
X-Api-Key: your-secure-api-key-here
```

#### AWS署名バージョン4（簡易版）
- 標準のS3クライアントツールと互換性があります
- HMAC-SHA256署名アルゴリズムを使用
- リクエストのリプレイ攻撃を防止

```http
Authorization: AWS4-HMAC-SHA256 Credential=ACCESS_KEY/20240101/us-east-1/s3/aws4_request, SignedHeaders=host, Signature=signature
```

#### HTTP基本認証
- シンプルなテスト環境に適しています
- ユーザー名とパスワードはBase64エンコードされます
- HTTPS環境でのみ使用することを推奨

```http
Authorization: Basic base64(username:password)
```

### 2. 認証設定のベストプラクティス

#### 強力なパスワードポリシー
```php
// config/credentials.php 内
'access_keys' => [
    'AKIA...' => [
        'secret' => 'wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY', // 少なくとも40文字
        'api_key' => bin2hex(random_bytes(32)), // 64文字の16進数
        'password' => bin2hex(random_bytes(16)), // 32文字の16進数
        'permissions' => ['read', 'write'],
        'active' => true
    ]
]
```

#### 定期的なキーローテーション
- アクセスキーは90日ごとに変更することを推奨
- スクリプトを使用して強力なキーを自動生成
- 新しいキーが正常に機能することを確認するまで、古いキーのバックアップを保持

```bash
# 新しいAPIキーを生成
php -r "echo bin2hex(random_bytes(32)) . PHP_EOL;"

# 新しいシークレットキーを生成
php -r "echo base64_encode(random_bytes(30)) . PHP_EOL;"
```

## 認可と権限管理

### 1. ロールベースのアクセス制御

PHP S3サーバーは、きめ細かな権限制御をサポートしています。

```php
'permissions' => [
    'read',    // オブジェクトの読み取りとバケットのリスト
    'write',   // オブジェクトの作成と更新
    'delete',  // オブジェクトとバケットの削除
    'admin'    // 管理権限と設定
]
```

### 2. 権限レベルの説明

| 権限 | 説明 | 許可される操作 |
|------|------|------------|
| read | 読み取り専用アクセス | GET, HEAD, LIST |
| write | 読み書きアクセス | read + PUT, POST |
| delete | 完全アクセス | write + DELETE |
| admin | 管理権限 | delete + ユーザー管理 |

### 3. バケットレベルの権限

```php
// バケット設定例
'bucket_config' => [
    'public_read' => false,   // 公開読み取り
    'public_write' => false,  // 公開書き込み
    'owner_only' => true      // 所有者のみアクセス
]
```

## データ保護

### 1. パス横断保護

システムは自動的にパス横断攻撃を防止します。

```php
// Utils.php のセキュリティチェック
public static function sanitizePath($path)
{
    // パス横断の試みを削除
    $path = str_replace(['../', '..\\', '../', '..\\'], '', $path);
    
    // 先頭のスラッシュを削除
    $path = ltrim($path, '/\\');
    
    return $path;
}
```

### 2. ファイルタイプ検証

```php
// 許可されるファイルタイプ（設定可能）
'allowed_mime_types' => [
    'text/plain',
    'image/jpeg',
    'image/png',
    'application/pdf',
    // さらにタイプを追加...
]
```

### 3. ファイルサイズ制限

```php
// config/config.php 内
'max_file_size' => 100 * 1024 * 1024, // 100MB
'max_total_size' => 1024 * 1024 * 1024, // 合計1GBの制限
```

## ネットワークセキュリティ

### 1. HTTPSの強制

```php
// config/config.php で有効化
'require_https' => true,
```

```apache
# .htaccess でHTTPSを強制
RewriteCond %{HTTPS} off
RewriteRule ^(.*)$ https://%{HTTP_HOST}%{REQUEST_URI} [L,R=301]
```

### 2. CORS設定

```php
'allowed_origins' => [
    'https://yourdomain.com',
    'https://app.yourdomain.com'
], // 本番環境では ['*'] を使用しないでください
```

### 3. セキュリティヘッダーの設定

```apache
# .htaccess でセキュリティヘッダーを追加
Header always set X-Content-Type-Options nosniff
Header always set X-Frame-Options DENY
Header always set X-XSS-Protection "1; mode=block"
Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains"
Header always set Content-Security-Policy "default-src 'self'"
```

## アクセス制御

### 1. IPホワイトリスト

```php
// config/config.php で設定
'ip_whitelist' => [
    '192.168.1.0/24',    // ローカルネットワーク
    '203.0.113.0/24',    // オフィスネットワーク
    '198.51.100.50'      // 特定のIP
]
```

### 2. レート制限

```php
// レート制限設定
'rate_limit_enabled' => true,
'rate_limit_requests' => 1000,  // 1時間あたりのリクエスト数
'rate_limit_window' => 3600,    // 時間枠（秒）
'rate_limit_burst' => 10        // バーストリクエスト制限
```

### 3. セッション管理

```php
// セッションセキュリティ設定
'session_timeout' => 3600,      // 1時間
'max_failed_attempts' => 5,     // 最大失敗試行回数
'lockout_duration' => 900       // ロックアウト時間（15分）
```

## ファイルシステムセキュリティ

### 1. ファイル権限の設定

```bash
# 推奨されるファイル権限
chmod 755 php-s3-server/                    # ディレクトリ
chmod 644 php-s3-server/*.php               # PHPファイル
chmod 600 php-s3-server/config/credentials.php  # 機密設定
chmod 755 php-s3-server/storage/            # ストレージディレクトリ
chmod 755 php-s3-server/logs/               # ログディレクトリ
```

### 2. ディレクトリ保護

```apache
# 機密ディレクトリを保護
<Directory "/path/to/php-s3-server/config">
    Order deny,allow
    Deny from all
</Directory>

<Directory "/path/to/php-s3-server/logs">
    Order deny,allow
    Deny from all
</Directory>

<Directory "/path/to/php-s3-server/src">
    Order deny,allow
    Deny from all
</Directory>
```

### 3. ファイルアップロードのセキュリティ

```php
// ファイルアップロードの検証
public function validateUpload($filename, $content)
{
    // ファイル拡張子をチェック
    $allowedExtensions = ['txt', 'jpg', 'png', 'pdf'];
    $extension = strtolower(pathinfo($filename, PATHINFO_EXTENSION));
    
    if (!in_array($extension, $allowedExtensions)) {
        throw new Exception('File type not allowed');
    }
    
    // ファイルの内容をチェック
    $finfo = finfo_open(FILEINFO_MIME_TYPE);
    $mimeType = finfo_buffer($finfo, $content);
    finfo_close($finfo);
    
    // MIMEタイプを検証
    if (!in_array($mimeType, $this->allowedMimeTypes)) {
        throw new Exception('MIME type not allowed');
    }
    
    return true;
}
```

## ログと監視

### 1. セキュリティログ記録

```php
// セキュリティイベントを記録
Utils::logSecurity('Failed authentication attempt', [
    'ip' => $_SERVER['REMOTE_ADDR'],
    'user_agent' => $_SERVER['HTTP_USER_AGENT'],
    'timestamp' => time()
]);
```

### 2. 異常活動の検出

```php
// 異常活動を検出
public function detectAnomalousActivity($userId)
{
    $recentRequests = $this->getRecentRequests($userId, 3600); // 1時間以内
    
    // リクエスト頻度をチェック
    if (count($recentRequests) > 1000) {
        $this->triggerSecurityAlert('High request frequency', $userId);
    }
    
    // 失敗した認証試行をチェック
    $failedAttempts = $this->getFailedAttempts($userId, 900); // 15分以内
    if (count($failedAttempts) > 5) {
        $this->lockAccount($userId, 900); // 15分間ロック
    }
}
```

### 3. ログローテーションと保護

```bash
# ログローテーションを設定
# crontab に追加
0 0 * * * /path/to/rotate-logs.sh

# rotate-logs.sh の内容
#!/bin/bash
cd /path/to/php-s3-server/logs
mv s3-server.log s3-server.log.$(date +%Y%m%d)
touch s3-server.log
chmod 644 s3-server.log

# 30日以上前のログを削除
find . -name "s3-server.log.*" -mtime +30 -delete
```

## バックアップとリカバリ

### 1. 設定のバックアップ

```bash
#!/bin/bash
# backup-config.sh
BACKUP_DIR="/path/to/backups"
DATE=$(date +%Y%m%d_%H%M%S)

# 設定ファイルをバックアップ
tar -czf "$BACKUP_DIR/config_$DATE.tar.gz" config/

# ストレージデータをバックアップ
tar -czf "$BACKUP_DIR/storage_$DATE.tar.gz" storage/

# 最近30日間のバックアップを保持
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +30 -delete
```

### 2. 暗号化されたバックアップ

```bash
# GPGを使用してバックアップを暗号化
gpg --cipher-algo AES256 --compress-algo 1 --s2k-mode 3 \
    --s2k-digest-algo SHA512 --s2k-count 65536 --symmetric \
    --output "backup_$DATE.tar.gz.gpg" "backup_$DATE.tar.gz"

# 暗号化されていないバックアップを削除
rm "backup_$DATE.tar.gz"
```

## セキュリティ監査

### 1. 定期的なセキュリティチェックリスト

- [ ] すべてのアクセスキーの強度を確認
- [ ] ファイル権限設定を検証
- [ ] ユーザー権限の割り当てをレビュー
- [ ] ログ内の異常活動をチェック
- [ ] HTTPS設定を検証
- [ ] バックアップとリカバリプロセスをテスト
- [ ] PHPと依存関係を更新
- [ ] ディスクスペースの使用状況をチェック

### 2. 脆弱性スキャン

```bash
# ツールを使用して一般的な脆弱性をスキャン
# 例：nikto を使用してWebアプリケーションをスキャン
nikto -h https://yourdomain.com/s3-server/

# PHP設定をチェック
php -m | grep -E "(openssl|hash|json|xml)"
```

### 3. 侵入テスト

定期的に侵入テストを実施します。これには以下が含まれます。
- 認証バイパスの試行
- パス横断テスト
- ファイルアップロードの脆弱性テスト
- SQLインジェクションテスト（データベースを使用している場合）
- XSSテスト

## インシデント対応

### 1. セキュリティインシデントの分類

| レベル | 説明 | 応答時間 | 行動 |
|------|------|----------|------|
| 低 | 単一の認証失敗 | 24時間 | ログ記録 |
| 中 | 複数回の認証失敗 | 1時間 | 一時的なロックアウト |
| 高 | 疑わしいファイルアップロード | 15分 | 即座にブロック |
| 深刻 | システム侵入の兆候 | 即座 | サービス停止 |

### 2. 緊急対応プロセス

```php
// 緊急サービス停止
public function emergencyShutdown($reason)
{
    // 停止理由を記録
    Utils::logSecurity('Emergency shutdown', ['reason' => $reason]);
    
    // メンテナンスモードファイルを作成
    file_put_contents(BASE_PATH . '/.maintenance', time());
    
    // 通知を送信
    $this->sendSecurityAlert('Service emergency shutdown', $reason);
    
    // メンテナンスモード応答を返す
    http_response_code(503);
    header('Retry-After: 3600');
    exit('Service temporarily unavailable due to security incident');
}
```

### 3. リカバリプロセス

1. **損害範囲の評価**
2. **セキュリティ脆弱性の修正**
3. **データの復元（必要な場合）**
4. **監視の強化**
5. **セキュリティポリシーの更新**
6. **サービスの再起動**

## コンプライアンスに関する考慮事項

### 1. データ保護規制

- **GDPR**: データ最小化とユーザー権利の実装
- **CCPA**: データ透明性と制御の提供
- **HIPAA**: 医療データを扱う場合、追加の暗号化が必要

### 2. データレジデンシー

```php
// データ保存場所の設定
'data_residency' => [
    'allowed_regions' => ['us-east-1', 'eu-west-1'],
    'default_region' => 'us-east-1',
    'encryption_required' => true
]
```

### 3. 監査ログ

```php
// コンプライアンス監査ログ
public function logComplianceEvent($event, $data)
{
    $logEntry = [
        'timestamp' => date('c'),
        'event' => $event,
        'user' => $this->getCurrentUser(),
        'ip' => $_SERVER['REMOTE_ADDR'],
        'data' => $data,
        'hash' => hash('sha256', json_encode($data))
    ];
    
    file_put_contents(
        LOGS_PATH . '/compliance.log',
        json_encode($logEntry) . PHP_EOL,
        FILE_APPEND | LOCK_EX
    );
}
```

## ベストプラクティスまとめ

### 1. デプロイ前のチェック

```bash
#!/bin/bash
# security-check.sh
echo "Running security checks..."

# ファイル権限をチェック
echo "Checking file permissions..."
find . -name "*.php" -not -perm 644 -ls
find . -type d -not -perm 755 -ls

# 機密ファイルをチェック
echo "Checking sensitive files..."
if [ -f "config/credentials.php" ]; then
    PERMS=$(stat -c %a config/credentials.php)
    if [ "$PERMS" != "600" ]; then
        echo "WARNING: credentials.php permissions should be 600"
    fi
fi

# 設定をチェック
echo "Checking configuration..."
grep -r "your-api-key-here" config/ && echo "WARNING: Default API keys found"
grep -r "admin123" config/ && echo "WARNING: Default passwords found"

echo "Security check completed."
```

### 2. 実行時監視

```php
// リアルタイムセキュリティ監視
class SecurityMonitor
{
    public function monitorRequests()
    {
        // リクエスト頻度をチェック
        $this->checkRequestRate();
        
        // 異常パターンを検出
        $this->detectAnomalies();
        
        // ファイルの整合性を検証
        $this->verifyFileIntegrity();
    }
    
    private function checkRequestRate()
    {
        $ip = $_SERVER['REMOTE_ADDR'];
        $requests = $this->getRequestCount($ip, 60); // 1分以内
        
        if ($requests > 100) {
            $this->blockIP($ip, 3600); // 1時間ブロック
        }
    }
}
```

### 3. 定期的なメンテナンス作業

```bash
# crontab に追加
# 毎日のセキュリティチェック
0 2 * * * /path/to/security-check.sh

# 毎週のログ分析
0 3 * * 0 /path/to/analyze-logs.sh

# 毎月のキーローテーションリマインダー
0 9 1 * * /path/to/key-rotation-reminder.sh
```

これらのセキュリティガイドラインとベストプラクティスに従うことで、PHP S3サーバーのセキュリティを大幅に向上させることができます。セキュリティは継続的なプロセスであり、セキュリティ対策を定期的に見直し、更新する必要があることを忘れないでください。

