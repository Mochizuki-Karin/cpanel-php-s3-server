# PHP S3サーバー インストールガイド

このドキュメントでは、cPanel無料ホスティングでPHP S3サーバーをインストールおよび設定するための詳細な手順を説明します。

## システム要件

### 最小要件

- **PHPバージョン**: 7.4以上
- **PHP拡張機能**: 
  - json (JSON処理)
  - xml (XML応答生成)
  - mbstring (マルチバイト文字列処理)
- **Webサーバー**: Apache (.htaccessおよびmod_rewriteをサポート)
- **ディスクスペース**: 少なくとも50MBの利用可能なスペース
- **メモリ**: 推奨128MB PHPメモリ制限

### 推奨設定

- **PHPバージョン**: 8.0以上
- **ディスクスペース**: 1GB以上（ファイルを保存するため）
- **メモリ**: 256MB PHPメモリ制限
- **実行時間**: 300秒以上（大容量ファイルのアップロード用）

## インストール手順

### ステップ1: ファイルの準備

1. PHP S3サーバーのすべてのファイルをダウンロードします。
2. ファイル構造が完全であることを確認します。

```
php-s3-server/
├── index.php
├── test.php
├── .htaccess
├── config/
│   ├── config.php
│   └── credentials.php
├── src/
│   ├── Auth.php
│   ├── BucketManager.php
│   ├── FileSystem.php
│   ├── ObjectManager.php
│   ├── Response.php
│   ├── Router.php
│   └── Utils.php
├── storage/
│   └── buckets/
├── logs/
└── docs/
```

### ステップ2: cPanelホストへのアップロード

#### ファイルマネージャーの使用

1. cPanelコントロールパネルにログインします。
2. 「ファイルマネージャー」を開きます。
3. `public_html` ディレクトリ（またはインストールしたいサブディレクトリ）に移動します。
4. 新しいフォルダ（例: `s3-server`）を作成します。
5. すべてのファイルをそのフォルダにアップロードします。

#### FTPクライアントの使用

```bash
# FTPを使用してファイルをアップロード
ftp your-domain.com
# ユーザー名とパスワードを入力
cd public_html
mkdir s3-server
cd s3-server
# すべてのファイルをアップロード
```

### ステップ3: ファイル権限の設定

適切なファイル権限は、セキュリティと機能にとって非常に重要です。

#### cPanelファイルマネージャー経由

1. `php-s3-server` ディレクトリを選択します。
2. 右クリック → 「権限」
3. 755 (rwxr-xr-x) に設定します。
4. 以下のディレクトリに対してこの操作を繰り返します。
   - `storage/` → 755
   - `storage/buckets/` → 755
   - `logs/` → 755
   - `config/` → 755

5. ファイルの権限を設定します。
   - すべての `.php` ファイル → 644 (rw-r--r--)
   - `config/credentials.php` → 600 (rw-------)
   - `.htaccess` → 644

#### SSH経由（利用可能な場合）

```bash
# ディレクトリ権限の設定
chmod 755 php-s3-server/
chmod 755 php-s3-server/storage/
chmod 755 php-s3-server/storage/buckets/
chmod 755 php-s3-server/logs/
chmod 755 php-s3-server/config/

# ファイル権限の設定
find php-s3-server/ -name "*.php" -exec chmod 644 {} \;
chmod 600 php-s3-server/config/credentials.php
chmod 644 php-s3-server/.htaccess
```

### ステップ4: PHP設定の構成

#### PHPバージョンと拡張機能の確認

一時的な `phpinfo.php` ファイルを作成して、PHP設定を確認します。

```php
<?php
phpinfo();
?>
```

このファイルをアップロードしてアクセスし、以下を確認します。
- PHPバージョン ≥ 7.4
- json, xml, mbstring 拡張機能がインストールされていること

#### PHP設定の調整（必要な場合）

PHP設定を変更する権限がある場合は、`.htaccess` ファイルに以下を追加できます。

```apache
# PHP設定の最適化
php_value memory_limit 128M
php_value max_execution_time 300
php_value upload_max_filesize 100M
php_value post_max_size 100M
php_value max_input_time 300
```

### ステップ5: 認証情報の構成

`config/credentials.php` ファイルを編集します。

```php
<?php
return [
    'access_keys' => [
        'YOUR_ACCESS_KEY' => [
            'secret' => 'YOUR_SECRET_KEY',
            'api_key' => 'YOUR_API_KEY',
            'password' => 'YOUR_PASSWORD',
            'permissions' => ['read', 'write'],
            'created' => '2024-01-01T00:00:00Z',
            'active' => true,
            'description' => 'Main admin user'
        ]
    ]
];
```

**重要なセキュリティのヒント**:
- 強力なパスワードとランダムなキーを使用してください。
- 認証情報を定期的に変更してください。
- これらの情報を公開の場所で共有しないでください。

### ステップ6: メイン設定の調整

`config/config.php` を編集して、ホスティング環境に合わせて調整します。

```php
return [
    // ホストの制限に合わせて調整
    'max_file_size' => 50 * 1024 * 1024, // 50MB
    'max_buckets' => 20,
    'default_bucket_quota' => 500 * 1024 * 1024, // 500MB
    
    // セキュリティ設定
    'public_access' => false,
    'require_https' => true, // 本番環境で推奨
    
    // パフォーマンス設定
    'memory_limit' => '128M',
    'max_execution_time' => 300,
];
```

### ステップ7: インストールのテスト

#### 組み込みテストの実行

`https://yourdomain.com/s3-server/test.php` にアクセスして、自動テストスイートを実行します。

成功したテスト出力は次のように表示されます。
```
✓ All components initialized successfully
✓ Bucket created
✓ Object uploaded
✓ All tests completed successfully!
```

#### 手動APIテスト

cURLを使用して基本的なAPI機能をテストします。

```bash
# サーバー応答をテスト
curl -X GET https://yourdomain.com/s3-server/ \
  -H "X-Api-Key: YOUR_API_KEY"

# 空のバケットリストXMLが返されるはずです。
```

### ステップ8: SSL/HTTPS設定

#### cPanel経由でSSLを有効にする

1. cPanelで「SSL/TLS」を見つけます。
2. 「HTTPSリダイレクトを強制」を有効にします。
3. 有効なSSL証明書がインストールされていることを確認します。

#### HTTPSを要求するように設定を更新

`config/config.php` で以下を設定します。

```php
'require_https' => true,
```

## 高度な設定

### URL書き換え設定

`.htaccess` ファイルがURL書き換えルールを正しく設定していることを確認します。

```apache
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php [QSA,L]
```

### ログ設定

デバッグのためにログ記録を設定します。

```php
// config/config.php 内
'debug' => false, // 本番環境ではfalseに設定
'log_level' => 'info',
'max_log_size' => 10 * 1024 * 1024, // 10MB
```

### パフォーマンス最適化

#### 圧縮の有効化

`.htaccess` でgzip圧縮を有効にします。

```apache
<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE text/plain
    AddOutputFilterByType DEFLATE application/xml
    AddOutputFilterByType DEFLATE application/json
</IfModule>
```

#### キャッシュ設定

静的リソースのキャッシュヘッダーを設定します。

```apache
<IfModule mod_expires.c>
    ExpiresActive On
    ExpiresByType application/xml "access plus 1 hour"
    ExpiresByType application/json "access plus 1 hour"
</IfModule>
```

## トラブルシューティング

### よくあるインストール問題

#### 1. 500内部サーバーエラー

**考えられる原因**:
- ファイル権限が正しくない
- PHP構文エラー
- PHP拡張機能の不足

**解決策**:
```bash
# エラーログを確認
tail -f logs/error.log

# ファイル権限を確認
ls -la config/credentials.php  # -rw------- であるべきです
```

#### 2. 認証失敗

**考えられる原因**:
- 認証情報の構成エラー
- 時間同期の問題

**解決策**:
- `config/credentials.php` のキーを確認します。
- サーバーの時間設定を確認します。

#### 3. ファイルアップロード失敗

**考えられる原因**:
- PHPのアップロード制限
- ディスクスペース不足
- 権限の問題

**解決策**:
```php
# PHPの制限を確認
echo ini_get('upload_max_filesize');
echo ini_get('post_max_size');

# ディスクスペースを確認
echo disk_free_space('.');
```

#### 4. URL書き換えが機能しない

**考えられる原因**:
- mod_rewriteが有効になっていない
- .htaccessの権限の問題

**解決策**:
- ホストプロバイダーにmod_rewriteを有効にするよう連絡します。
- .htaccessファイルの権限が644であることを確認します。

### デバッグのヒント

#### デバッグモードの有効化

`config/config.php` で一時的に有効にします。

```php
'debug' => true,
'show_errors' => true,
```

#### ログファイルの表示

```bash
# アプリケーションログを表示
tail -f logs/s3-server.log

# エラーログを表示
tail -f logs/error.log
```

#### PHP設定のテスト

テストスクリプトを作成します。

```php
<?php
// test-config.php
echo "PHP Version: " . phpversion() . "\n";
echo "JSON Extension: " . (extension_loaded('json') ? 'Yes' : 'No') . "\n";
echo "XML Extension: " . (extension_loaded('xml') ? 'Yes' : 'No') . "\n";
echo "MBString Extension: " . (extension_loaded('mbstring') ? 'Yes' : 'No') . "\n";
echo "Memory Limit: " . ini_get('memory_limit') . "\n";
echo "Max Execution Time: " . ini_get('max_execution_time') . "\n";
echo "Upload Max Filesize: " . ini_get('upload_max_filesize') . "\n";
?>
```

## セキュリティのベストプラクティス

### 1. ファイル権限

- 設定ファイル: 600 (所有者のみ読み書き可能)
- PHPファイル: 644 (所有者読み書き可能、その他読み取り専用)
- ディレクトリ: 755 (所有者フル権限、その他読み取り実行可能)

### 2. アクセス制御

```apache
# .htaccessで機密ディレクトリを保護
<Files "config/*">
    Order deny,allow
    Deny from all
</Files>

<Files "logs/*">
    Order deny,allow
    Deny from all
</Files>
```

### 3. 定期的なメンテナンス

- アクセスキーを定期的に変更します。
- ログファイルのサイズを監視します。
- ディスクスペースの使用状況を確認します。
- PHPバージョンを更新します（可能な場合）。

## アップグレードとメンテナンス

### 設定のバックアップ

アップグレード前に重要なファイルをバックアップします。

```bash
# 設定ファイルをバックアップ
cp config/credentials.php config/credentials.php.backup
cp config/config.php config/config.php.backup

# ストレージデータをバックアップ
tar -czf storage-backup.tar.gz storage/
```

### パフォーマンスの監視

定期的に以下を確認します。
- ディスクスペースの使用状況
- ログファイルのサイズ
- 応答時間
- エラー率

この詳細なインストールガイドに従うことで、cPanel無料ホスティングにPHP S3サーバーを正常にデプロイできるはずです。問題が発生した場合は、トラブルシューティングセクションを参照するか、ログファイルを確認して詳細情報を入手してください。

