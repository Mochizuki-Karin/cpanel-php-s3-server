# PHP S3サーバー

cPanel無料ホスティング向けに設計された軽量のS3互換サーバーで、仮想ホストをフル機能のS3バケットストレージサービスに変換します。

## プロジェクト概要

PHP S3サーバーは、純粋なPHPで書かれた軽量のS3互換ストレージサーバーであり、cPanel無料ホスティング環境向けに最適化されています。Amazon S3と互換性のあるRESTful APIを提供し、標準のS3クライアントツールとSDKを使用してファイルストレージを管理できます。

### 主な機能

- **S3 API互換性**: コアS3 API操作（バケットおよびオブジェクト管理を含む）をサポート
- **軽量設計**: リソース使用量を最小限に抑え、共有ホスティング環境の制限に対応
- **複数の認証方法**: AWS署名、APIキー、基本認証をサポート
- **セキュリティ**: パス横断保護、アクセス制御、レート制限を内蔵
- **簡単なデプロイ**: 追加の依存関係なしで、cPanelホストに直接アップロードして使用可能
- **完全なログ記録**: 詳細な操作ログとエラー追跡
- **CORSサポート**: 完全なクロスオリジンリソース共有サポート

### システム要件

- PHP 7.4以上
- PHP拡張機能: json, xml, mbstring
- Apacheサーバー（.htaccessをサポート）
- 少なくとも50MBの利用可能なディスクスペース

## クイックスタート

### 1. ダウンロードとアップロード

すべてのファイルをcPanel仮想ホストの公開ディレクトリまたはサブディレクトリにアップロードします。

### 2. ファイル権限の設定

```bash
# ディレクトリ権限の設定
chmod 755 php-s3-server/
chmod 755 php-s3-server/storage/
chmod 755 php-s3-server/storage/buckets/
chmod 755 php-s3-server/logs/

# ファイル権限の設定
chmod 644 php-s3-server/*.php
chmod 644 php-s3-server/src/*.php
chmod 600 php-s3-server/config/credentials.php
chmod 644 php-s3-server/config/config.php
```

### 3. 認証の設定

`config/credentials.php` ファイルを編集し、デフォルトのアクセスキーを更新します。

```php
'AKIAIOSFODNN7EXAMPLE' => [
    'secret' => 'your-secret-key-here',
    'api_key' => 'your-api-key-here',
    'password' => 'your-password-here',
    'permissions' => ['read', 'write'],
    'active' => true
]
```

### 4. インストールのテスト

`https://yourdomain.com/path-to-s3-server/test.php` にアクセスして、組み込みのテストスイートを実行します。

## API使用例

### cURLの使用

```bash
# すべてのバケットを一覧表示
curl -X GET https://yourdomain.com/s3-server/ \
  -H "X-Api-Key: your-api-key-here"

# バケットを作成
curl -X PUT https://yourdomain.com/s3-server/my-bucket \
  -H "X-Api-Key: your-api-key-here"

# ファイルをアップロード
curl -X PUT https://yourdomain.com/s3-server/my-bucket/file.txt \
  -H "X-Api-Key: your-api-key-here" \
  -H "Content-Type: text/plain" \
  --data-binary @localfile.txt

# ファイルをダウンロード
curl -X GET https://yourdomain.com/s3-server/my-bucket/file.txt \
  -H "X-Api-Key: your-api-key-here"
```

### AWS CLIの使用

PHP S3サーバーを使用するようにAWS CLIを設定します。

```bash
aws configure set aws_access_key_id AKIAIOSFODNN7EXAMPLE
aws configure set aws_secret_access_key your-secret-key-here
aws configure set default.region us-east-1

# カスタムエンドポイントを使用
aws s3 ls --endpoint-url https://yourdomain.com/s3-server/
```

## サポートされているAPI操作

### サービスレベル操作
- `GET /` - すべてのバケットを一覧表示

### バケット操作
- `PUT /{bucket}` - バケットを作成
- `GET /{bucket}` - バケット内のオブジェクトを一覧表示
- `DELETE /{bucket}` - バケットを削除
- `HEAD /{bucket}` - バケットが存在するかどうかを確認

### オブジェクト操作
- `PUT /{bucket}/{key}` - オブジェクトをアップロード
- `GET /{bucket}/{key}` - オブジェクトをダウンロード
- `DELETE /{bucket}/{key}` - オブジェクトを削除
- `HEAD /{bucket}/{key}` - オブジェクトのメタデータを取得

## 設定オプション

主要な設定ファイルは `config/config.php` にあり、以下の重要な設定が含まれています。

- `max_file_size`: 最大ファイルアップロードサイズ
- `max_buckets`: 最大バケット数
- `public_access`: 匿名アクセスを許可するかどうか
- `rate_limit_requests`: 1時間あたりのリクエスト制限

## セキュリティに関する考慮事項

1. **ファイル権限**: `config/credentials.php` のファイル権限が600に設定されていることを確認してください。
2. **HTTPS**: 本番環境では常にHTTPSを使用してください。
3. **アクセス制御**: 必要に応じてIPホワイトリストを設定してください。
4. **定期的な更新**: アクセスキーとシークレットキーを定期的に変更してください。

## トラブルシューティング

### よくある問題

1. **500内部サーバーエラー**: PHPエラーログとファイル権限を確認してください。
2. **認証失敗**: アクセスキーとシークレットキーの設定を確認してください。
3. **ファイルアップロード失敗**: ディスクスペースとPHPのアップロード制限を確認してください。
4. **CORSエラー**: .htaccessファイルが正しく設定されていることを確認してください。

### ログファイル

- アプリケーションログ: `logs/s3-server.log`
- エラーログ: `logs/error.log`

## ライセンス

このプロジェクトはMITライセンスの下で提供されています。詳細についてはLICENSEファイルを参照してください。

## サポート

技術サポートや問題報告については、プロジェクトのGitHubページにアクセスするか、開発チームにお問い合わせください。

---

**注意**: これは軽量な実装であり、主に開発およびテスト環境での使用を目的としています。本番環境での使用には、専門的なS3互換ストレージソリューションを推奨します。

