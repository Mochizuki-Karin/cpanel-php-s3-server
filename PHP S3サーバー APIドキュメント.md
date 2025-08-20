# PHP S3サーバー APIドキュメント

このドキュメントでは、PHP S3サーバーがサポートするすべてのAPIエンドポイントと操作について詳しく説明します。

## API概要

PHP S3サーバーは、Amazon S3と互換性のあるRESTful APIを提供し、標準のHTTPメソッドとS3応答形式をサポートしています。すべてのAPI応答はXML形式を使用し、AWS S3との互換性を維持しています。

### ベースURL

```
https://yourdomain.com/path-to-s3-server/
```

### 認証方法

3つの認証方法をサポートしています。

1. **APIキー認証**（推奨）
2. **AWS署名バージョン4**（簡易版）
3. **HTTP基本認証**

## 認証方法

### 1. APIキー認証

リクエストヘッダーにAPIキーを含めます。

```http
X-Api-Key: your-api-key-here
```

### 2. AWS署名認証

標準のAWS Authorizationヘッダーを使用します。

```http
Authorization: AWS4-HMAC-SHA256 Credential=ACCESS_KEY/20240101/us-east-1/s3/aws4_request, SignedHeaders=host, Signature=signature
```

### 3. HTTP基本認証

```http
Authorization: Basic base64(username:password)
```

## サービスレベルAPI

### すべてのバケットを一覧表示

現在のユーザーがアクセスできるすべてのバケットを一覧表示します。

**リクエスト**
```http
GET /
Host: yourdomain.com
X-Api-Key: your-api-key-here
```

**応答**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<ListAllMyBucketsResult xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
    <Owner>
        <ID>php-s3-server</ID>
        <DisplayName>PHP S3 Server</DisplayName>
    </Owner>
    <Buckets>
        <Bucket>
            <Name>my-bucket</Name>
            <CreationDate>2024-01-01T12:00:00.000Z</CreationDate>
        </Bucket>
        <Bucket>
            <Name>another-bucket</Name>
            <CreationDate>2024-01-02T12:00:00.000Z</CreationDate>
        </Bucket>
    </Buckets>
</ListAllMyBucketsResult>
```

**ステータスコード**
- `200 OK` - 成功
- `403 Forbidden` - 認証失敗

## バケットAPI

### バケットを作成

新しいバケットを作成します。

**リクエスト**
```http
PUT /{bucket}
Host: yourdomain.com
X-Api-Key: your-api-key-here
```

**応答**
- `200 OK` - バケット作成成功
- `409 Conflict` - バケットが既に存在します
- `400 Bad Request` - バケット名が無効です

**バケット命名規則**
- 長さ：3〜63文字
- 小文字、数字、ハイフン、ピリオドのみを含めることができます
- 文字または数字で始まり、文字または数字で終わる必要があります
- 連続するピリオドまたはハイフンを含めることはできません

### バケットが存在するかどうかを確認

指定されたバケットが存在するかどうかを確認します。

**リクエスト**
```http
HEAD /{bucket}
Host: yourdomain.com
X-Api-Key: your-api-key-here
```

**応答**
- `200 OK` - バケットが存在します
- `404 Not Found` - バケットが見つかりません

### バケット内のオブジェクトを一覧表示

指定されたバケット内のすべてのオブジェクトを一覧表示します。

**リクエスト**
```http
GET /{bucket}?prefix={prefix}&marker={marker}&max-keys={max-keys}
Host: yourdomain.com
X-Api-Key: your-api-key-here
```

**クエリパラメータ**
- `prefix` (オプション) - オブジェクトキーのプレフィックスフィルタ
- `marker` (オプション) - ページネーションマーカー
- `max-keys` (オプション) - 返される最大オブジェクト数（デフォルト1000）

**応答**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<ListBucketResult xmlns="http://s3.amazonaws.com/doc/2006-03-01/">
    <Name>my-bucket</Name>
    <Prefix></Prefix>
    <Marker></Marker>
    <MaxKeys>1000</MaxKeys>
    <IsTruncated>false</IsTruncated>
    <Contents>
        <Key>file1.txt</Key>
        <LastModified>2024-01-01T12:00:00.000Z</LastModified>
        <ETag>"d41d8cd98f00b204e9800998ecf8427e"</ETag>
        <Size>1024</Size>
        <StorageClass>STANDARD</StorageClass>
        <Owner>
            <ID>php-s3-server</ID>
            <DisplayName>PHP S3 Server</DisplayName>
        </Owner>
    </Contents>
</ListBucketResult>
```

### バケットを削除

空のバケットを削除します。

**リクエスト**
```http
DELETE /{bucket}
Host: yourdomain.com
X-Api-Key: your-api-key-here
```

**応答**
- `204 No Content` - 削除成功
- `404 Not Found` - バケットが見つかりません
- `409 Conflict` - バケットが空ではありません

## オブジェクトAPI

### オブジェクトをアップロード

指定されたバケットにオブジェクトをアップロードします。

**リクエスト**
```http
PUT /{bucket}/{key}
Host: yourdomain.com
Content-Type: text/plain
Content-Length: 1024
X-Api-Key: your-api-key-here

[オブジェクトデータ]
```

**オプションのリクエストヘッダー**
- `Content-Type` - オブジェクトのMIMEタイプ
- `Content-Length` - オブジェクトサイズ
- `x-amz-meta-*` - カスタムメタデータ
- `Cache-Control` - キャッシュ制御
- `Content-Disposition` - コンテンツの配置
- `Content-Encoding` - コンテンツのエンコーディング

**応答**
- `200 OK` - アップロード成功
- `404 Not Found` - バケットが見つかりません
- `400 Bad Request` - オブジェクトキーが無効です

**応答ヘッダー**
```http
ETag: "d41d8cd98f00b204e9800998ecf8427e"
x-amz-request-id: 1234567890ABCDEF
```

### オブジェクトをダウンロード

指定されたバケットからオブジェクトをダウンロードします。

**リクエスト**
```http
GET /{bucket}/{key}
Host: yourdomain.com
X-Api-Key: your-api-key-here
```

**オプションのリクエストヘッダー**
- `Range` - 部分コンテンツリクエスト（例：`bytes=0-1023`）

**応答**
```http
HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: 1024
Last-Modified: Mon, 01 Jan 2024 12:00:00 GMT
ETag: "d41d8cd98f00b204e9800998ecf8427e"
x-amz-request-id: 1234567890ABCDEF

[オブジェクトデータ]
```

**部分コンテンツ応答**
```http
HTTP/1.1 206 Partial Content
Content-Type: text/plain
Content-Length: 512
Content-Range: bytes 0-511/1024
Last-Modified: Mon, 01 Jan 2024 12:00:00 GMT
ETag: "d41d8cd98f00b204e9800998ecf8427e"

[部分オブジェクトデータ]
```

### オブジェクトのメタデータを取得

オブジェクトのメタデータ情報を取得します。オブジェクトの内容は返されません。

**リクエスト**
```http
HEAD /{bucket}/{key}
Host: yourdomain.com
X-Api-Key: your-api-key-here
```

**応答**
```http
HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: 1024
Last-Modified: Mon, 01 Jan 2024 12:00:00 GMT
ETag: "d41d8cd98f00b204e9800998ecf8427e"
x-amz-meta-custom: custom-value
x-amz-request-id: 1234567890ABCDEF
```

### オブジェクトを削除

指定されたバケットからオブジェクトを削除します。

**リクエスト**
```http
DELETE /{bucket}/{key}
Host: yourdomain.com
X-Api-Key: your-api-key-here
```

**応答**
- `204 No Content` - 削除成功（オブジェクトが存在しない場合でも）

## エラー応答

すべてのエラー応答はXML形式を使用します。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Error>
    <Code>NoSuchBucket</Code>
    <Message>The specified bucket does not exist.</Message>
    <Resource>/nonexistent-bucket</Resource>
    <RequestId>1234567890ABCDEF</RequestId>
</Error>
```

### 一般的なエラーコード

| エラーコード | HTTPステータスコード | 説明 |
|--------|------------|------|
| AccessDenied | 403 | アクセスが拒否されました |
| BucketAlreadyExists | 409 | バケットが既に存在します |
| BucketNotEmpty | 409 | バケットが空ではありません |
| InvalidBucketName | 400 | バケット名が無効です |
| InvalidKey | 400 | オブジェクトキーが無効です |
| NoSuchBucket | 404 | バケットが見つかりません |
| NoSuchKey | 404 | オブジェクトが見つかりません |
| InternalError | 500 | 内部サーバーエラー |
| MethodNotAllowed | 405 | メソッドが許可されていません |
| InvalidRange | 416 | リクエストされた範囲が満たされません |

## 使用例

### cURLの例

```bash
# バケットを作成
curl -X PUT https://yourdomain.com/s3-server/my-bucket \
  -H "X-Api-Key: your-api-key-here"

# ファイルをアップロード
curl -X PUT https://yourdomain.com/s3-server/my-bucket/hello.txt \
  -H "X-Api-Key: your-api-key-here" \
  -H "Content-Type: text/plain" \
  -d "Hello, World!"

# ファイルをダウンロード
curl -X GET https://yourdomain.com/s3-server/my-bucket/hello.txt \
  -H "X-Api-Key: your-api-key-here"

# オブジェクトを一覧表示
curl -X GET https://yourdomain.com/s3-server/my-bucket \
  -H "X-Api-Key: your-api-key-here"

# オブジェクトを削除
curl -X DELETE https://yourdomain.com/s3-server/my-bucket/hello.txt \
  -H "X-Api-Key: your-api-key-here"

# バケットを削除
curl -X DELETE https://yourdomain.com/s3-server/my-bucket \
  -H "X-Api-Key: your-api-key-here"
```

### Pythonの例

```python
import requests

# 設定
base_url = "https://yourdomain.com/s3-server"
api_key = "your-api-key-here"
headers = {"X-Api-Key": api_key}

# バケットを作成
response = requests.put(f"{base_url}/my-bucket", headers=headers)
print(f"Create bucket: {response.status_code}")

# ファイルをアップロード
data = "Hello, World!"
response = requests.put(
    f"{base_url}/my-bucket/hello.txt",
    headers={**headers, "Content-Type": "text/plain"},
    data=data
)
print(f"Upload file: {response.status_code}")

# ファイルをダウンロード
response = requests.get(f"{base_url}/my-bucket/hello.txt", headers=headers)
print(f"Download file: {response.status_code}")
print(f"Content: {response.text}")

# オブジェクトを一覧表示
response = requests.get(f"{base_url}/my-bucket", headers=headers)
print(f"List objects: {response.status_code}")
```

### JavaScriptの例

```javascript
const baseUrl = 'https://yourdomain.com/s3-server';
const apiKey = 'your-api-key-here';
const headers = {
    'X-Api-Key': apiKey
};

// バケットを作成
fetch(`${baseUrl}/my-bucket`, {
    method: 'PUT',
    headers: headers
})
.then(response => console.log('Create bucket:', response.status));

// ファイルをアップロード
fetch(`${baseUrl}/my-bucket/hello.txt`, {
    method: 'PUT',
    headers: {
        ...headers,
        'Content-Type': 'text/plain'
    },
    body: 'Hello, World!'
})
.then(response => console.log('Upload file:', response.status));

// ファイルをダウンロード
fetch(`${baseUrl}/my-bucket/hello.txt`, {
    method: 'GET',
    headers: headers
})
.then(response => response.text())
.then(data => console.log('File content:', data));
```

## 制限事項と注意事項

### ファイルサイズ制限

- デフォルトの最大ファイルサイズ：100MB
- `config/config.php` で `max_file_size` 設定を調整できます
- PHPおよびWebサーバーの設定によって制限されます

### バケット制限

- デフォルトの最大バケット数：50個
- バケット名はS3命名規則に準拠する必要があります
- バケットを削除する前に、すべてのオブジェクトを削除する必要があります

### オブジェクトキー制限

- 最大長：1024文字
- パス横断文字（`../`）を含めることはできません
- スラッシュで始めることはできません

### パフォーマンスに関する考慮事項

- 大容量ファイルのアップロードには時間がかかる場合があります
- 同時リクエスト数はホスティング環境によって制限されます
- 大量の小さなファイルはバッチ操作で処理することをお勧めします

### 互換性に関する注意

- コアS3 API操作をサポート
- 簡易版のAWS署名検証
- 高度な機能（バージョン管理、ライフサイクル管理など）はサポートしていません
- 開発およびテスト環境に適しています

## トラブルシューティング

### 一般的なAPIエラー

1. **403 Access Denied**
   - APIキーが正しいか確認してください
   - ユーザー権限の設定を確認してください

2. **404 Not Found**
   - バケットまたはオブジェクトが存在するか確認してください
   - URLパスが正しいか確認してください

3. **500 Internal Error**
   - サーバーのエラーログを確認してください
   - ディスクスペースと権限を確認してください

### デバッグのヒント

- 詳細なログ記録を有効にします
- ブラウザの開発者ツールを使用してリクエストを検査します
- リクエストヘッダーとパラメータの形式を確認します

このAPIドキュメントを通じて、PHP S3サーバーと正常にやり取りできるはずです。問題が発生した場合は、エラー応答セクションを参照するか、サーバーログを確認して詳細情報を入手してください。

