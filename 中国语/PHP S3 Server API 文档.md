# PHP S3 Server API 文档

本文档详细描述了PHP S3 Server支持的所有API端点和操作。

## API概述

PHP S3 Server提供与Amazon S3兼容的RESTful API，支持标准的HTTP方法和S3响应格式。所有API响应都使用XML格式，与AWS S3保持兼容。

### 基础URL

```
https://yourdomain.com/path-to-s3-server/
```

### 认证方式

支持三种认证方式：

1. **API密钥认证**（推荐）
2. **AWS签名版本4**（简化版）
3. **HTTP基本认证**

## 认证方法

### 1. API密钥认证

在请求头中包含API密钥：

```http
X-Api-Key: your-api-key-here
```

### 2. AWS签名认证

使用标准的AWS Authorization头：

```http
Authorization: AWS4-HMAC-SHA256 Credential=ACCESS_KEY/20240101/us-east-1/s3/aws4_request, SignedHeaders=host, Signature=signature
```

### 3. HTTP基本认证

```http
Authorization: Basic base64(username:password)
```

## 服务级API

### 列出所有存储桶

列出当前用户可访问的所有存储桶。

**请求**
```http
GET /
Host: yourdomain.com
X-Api-Key: your-api-key-here
```

**响应**
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

**状态码**
- `200 OK` - 成功
- `403 Forbidden` - 认证失败

## 存储桶API

### 创建存储桶

创建一个新的存储桶。

**请求**
```http
PUT /{bucket}
Host: yourdomain.com
X-Api-Key: your-api-key-here
```

**响应**
- `200 OK` - 存储桶创建成功
- `409 Conflict` - 存储桶已存在
- `400 Bad Request` - 存储桶名称无效

**存储桶命名规则**
- 长度：3-63个字符
- 只能包含小写字母、数字、连字符和点
- 必须以字母或数字开头和结尾
- 不能包含连续的点或连字符

### 检查存储桶是否存在

检查指定的存储桶是否存在。

**请求**
```http
HEAD /{bucket}
Host: yourdomain.com
X-Api-Key: your-api-key-here
```

**响应**
- `200 OK` - 存储桶存在
- `404 Not Found` - 存储桶不存在

### 列出存储桶中的对象

列出指定存储桶中的所有对象。

**请求**
```http
GET /{bucket}?prefix={prefix}&marker={marker}&max-keys={max-keys}
Host: yourdomain.com
X-Api-Key: your-api-key-here
```

**查询参数**
- `prefix` (可选) - 对象键前缀过滤
- `marker` (可选) - 分页标记
- `max-keys` (可选) - 返回的最大对象数量（默认1000）

**响应**
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

### 删除存储桶

删除一个空的存储桶。

**请求**
```http
DELETE /{bucket}
Host: yourdomain.com
X-Api-Key: your-api-key-here
```

**响应**
- `204 No Content` - 删除成功
- `404 Not Found` - 存储桶不存在
- `409 Conflict` - 存储桶不为空

## 对象API

### 上传对象

将对象上传到指定的存储桶。

**请求**
```http
PUT /{bucket}/{key}
Host: yourdomain.com
Content-Type: text/plain
Content-Length: 1024
X-Api-Key: your-api-key-here

[对象数据]
```

**可选请求头**
- `Content-Type` - 对象的MIME类型
- `Content-Length` - 对象大小
- `x-amz-meta-*` - 自定义元数据
- `Cache-Control` - 缓存控制
- `Content-Disposition` - 内容处置
- `Content-Encoding` - 内容编码

**响应**
- `200 OK` - 上传成功
- `404 Not Found` - 存储桶不存在
- `400 Bad Request` - 对象键无效

**响应头**
```http
ETag: "d41d8cd98f00b204e9800998ecf8427e"
x-amz-request-id: 1234567890ABCDEF
```

### 下载对象

从指定存储桶下载对象。

**请求**
```http
GET /{bucket}/{key}
Host: yourdomain.com
X-Api-Key: your-api-key-here
```

**可选请求头**
- `Range` - 部分内容请求（例如：`bytes=0-1023`）

**响应**
```http
HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: 1024
Last-Modified: Mon, 01 Jan 2024 12:00:00 GMT
ETag: "d41d8cd98f00b204e9800998ecf8427e"
x-amz-request-id: 1234567890ABCDEF

[对象数据]
```

**部分内容响应**
```http
HTTP/1.1 206 Partial Content
Content-Type: text/plain
Content-Length: 512
Content-Range: bytes 0-511/1024
Last-Modified: Mon, 01 Jan 2024 12:00:00 GMT
ETag: "d41d8cd98f00b204e9800998ecf8427e"

[部分对象数据]
```

### 获取对象元数据

获取对象的元数据信息，不返回对象内容。

**请求**
```http
HEAD /{bucket}/{key}
Host: yourdomain.com
X-Api-Key: your-api-key-here
```

**响应**
```http
HTTP/1.1 200 OK
Content-Type: text/plain
Content-Length: 1024
Last-Modified: Mon, 01 Jan 2024 12:00:00 GMT
ETag: "d41d8cd98f00b204e9800998ecf8427e"
x-amz-meta-custom: custom-value
x-amz-request-id: 1234567890ABCDEF
```

### 删除对象

从指定存储桶删除对象。

**请求**
```http
DELETE /{bucket}/{key}
Host: yourdomain.com
X-Api-Key: your-api-key-here
```

**响应**
- `204 No Content` - 删除成功（即使对象不存在）

## 错误响应

所有错误响应都使用XML格式：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Error>
    <Code>NoSuchBucket</Code>
    <Message>The specified bucket does not exist.</Message>
    <Resource>/nonexistent-bucket</Resource>
    <RequestId>1234567890ABCDEF</RequestId>
</Error>
```

### 常见错误码

| 错误码 | HTTP状态码 | 描述 |
|--------|------------|------|
| AccessDenied | 403 | 访问被拒绝 |
| BucketAlreadyExists | 409 | 存储桶已存在 |
| BucketNotEmpty | 409 | 存储桶不为空 |
| InvalidBucketName | 400 | 存储桶名称无效 |
| InvalidKey | 400 | 对象键无效 |
| NoSuchBucket | 404 | 存储桶不存在 |
| NoSuchKey | 404 | 对象不存在 |
| InternalError | 500 | 内部服务器错误 |
| MethodNotAllowed | 405 | 方法不允许 |
| InvalidRange | 416 | 请求范围无效 |

## 使用示例

### cURL示例

```bash
# 创建存储桶
curl -X PUT https://yourdomain.com/s3-server/my-bucket \
  -H "X-Api-Key: your-api-key-here"

# 上传文件
curl -X PUT https://yourdomain.com/s3-server/my-bucket/hello.txt \
  -H "X-Api-Key: your-api-key-here" \
  -H "Content-Type: text/plain" \
  -d "Hello, World!"

# 下载文件
curl -X GET https://yourdomain.com/s3-server/my-bucket/hello.txt \
  -H "X-Api-Key: your-api-key-here"

# 列出对象
curl -X GET https://yourdomain.com/s3-server/my-bucket \
  -H "X-Api-Key: your-api-key-here"

# 删除对象
curl -X DELETE https://yourdomain.com/s3-server/my-bucket/hello.txt \
  -H "X-Api-Key: your-api-key-here"

# 删除存储桶
curl -X DELETE https://yourdomain.com/s3-server/my-bucket \
  -H "X-Api-Key: your-api-key-here"
```

### Python示例

```python
import requests

# 配置
base_url = "https://yourdomain.com/s3-server"
api_key = "your-api-key-here"
headers = {"X-Api-Key": api_key}

# 创建存储桶
response = requests.put(f"{base_url}/my-bucket", headers=headers)
print(f"Create bucket: {response.status_code}")

# 上传文件
data = "Hello, World!"
response = requests.put(
    f"{base_url}/my-bucket/hello.txt",
    headers={**headers, "Content-Type": "text/plain"},
    data=data
)
print(f"Upload file: {response.status_code}")

# 下载文件
response = requests.get(f"{base_url}/my-bucket/hello.txt", headers=headers)
print(f"Download file: {response.status_code}")
print(f"Content: {response.text}")

# 列出对象
response = requests.get(f"{base_url}/my-bucket", headers=headers)
print(f"List objects: {response.status_code}")
```

### JavaScript示例

```javascript
const baseUrl = 'https://yourdomain.com/s3-server';
const apiKey = 'your-api-key-here';
const headers = {
    'X-Api-Key': apiKey
};

// 创建存储桶
fetch(`${baseUrl}/my-bucket`, {
    method: 'PUT',
    headers: headers
})
.then(response => console.log('Create bucket:', response.status));

// 上传文件
fetch(`${baseUrl}/my-bucket/hello.txt`, {
    method: 'PUT',
    headers: {
        ...headers,
        'Content-Type': 'text/plain'
    },
    body: 'Hello, World!'
})
.then(response => console.log('Upload file:', response.status));

// 下载文件
fetch(`${baseUrl}/my-bucket/hello.txt`, {
    method: 'GET',
    headers: headers
})
.then(response => response.text())
.then(data => console.log('File content:', data));
```

## 限制和注意事项

### 文件大小限制

- 默认最大文件大小：100MB
- 可在 `config/config.php` 中调整 `max_file_size` 设置
- 受PHP和Web服务器配置限制

### 存储桶限制

- 默认最大存储桶数量：50个
- 存储桶名称必须符合S3命名规范
- 删除存储桶前必须先删除所有对象

### 对象键限制

- 最大长度：1024个字符
- 不能包含路径遍历字符（`../`）
- 不能以斜杠开头

### 性能考虑

- 大文件上传可能需要较长时间
- 并发请求数量受主机环境限制
- 建议对大量小文件进行批量操作

### 兼容性说明

- 支持核心S3 API操作
- 简化的AWS签名验证
- 不支持高级功能（如版本控制、生命周期管理等）
- 适用于开发和测试环境

## 故障排除

### 常见API错误

1. **403 Access Denied**
   - 检查API密钥是否正确
   - 验证用户权限设置

2. **404 Not Found**
   - 确认存储桶或对象是否存在
   - 检查URL路径是否正确

3. **500 Internal Error**
   - 查看服务器错误日志
   - 检查磁盘空间和权限

### 调试技巧

- 启用详细日志记录
- 使用浏览器开发者工具检查请求
- 验证请求头和参数格式

通过这个API文档，您应该能够成功地与PHP S3 Server进行交互。如果遇到问题，请参考错误响应部分或查看服务器日志以获取更多信息。

