# PHP S3 Server

一个轻量级的S3兼容服务器，专为cPanel免费版虚拟主机设计，将您的虚拟主机转换为功能齐全的S3存储桶服务。

## 项目概述

PHP S3 Server是一个用纯PHP编写的轻量级S3兼容存储服务器，专门针对cPanel免费版虚拟主机环境进行了优化。它提供了与Amazon S3兼容的RESTful API，允许您使用标准的S3客户端工具和SDK来管理文件存储。

### 主要特性

- **S3 API兼容性**: 支持核心的S3 API操作，包括存储桶和对象管理
- **轻量级设计**: 最小化资源占用，适应共享主机环境的限制
- **多种认证方式**: 支持AWS签名、API密钥和基本认证
- **安全性**: 内置路径遍历保护、访问控制和速率限制
- **易于部署**: 无需额外依赖，直接上传到cPanel主机即可使用
- **完整日志记录**: 详细的操作日志和错误追踪
- **CORS支持**: 完整的跨域资源共享支持

### 系统要求

- PHP 7.4 或更高版本
- PHP扩展: json, xml, mbstring
- Apache服务器（支持.htaccess）
- 至少50MB可用磁盘空间

## 快速开始

### 1. 下载和上传

将所有文件上传到您的cPanel虚拟主机的公共目录或子目录中。

### 2. 设置文件权限

```bash
# 设置目录权限
chmod 755 php-s3-server/
chmod 755 php-s3-server/storage/
chmod 755 php-s3-server/storage/buckets/
chmod 755 php-s3-server/logs/

# 设置文件权限
chmod 644 php-s3-server/*.php
chmod 644 php-s3-server/src/*.php
chmod 600 php-s3-server/config/credentials.php
chmod 644 php-s3-server/config/config.php
```

### 3. 配置认证

编辑 `config/credentials.php` 文件，更新默认的访问密钥：

```php
'AKIAIOSFODNN7EXAMPLE' => [
    'secret' => 'your-secret-key-here',
    'api_key' => 'your-api-key-here',
    'password' => 'your-password-here',
    'permissions' => ['read', 'write'],
    'active' => true
]
```

### 4. 测试安装

访问 `https://yourdomain.com/path-to-s3-server/test.php` 运行内置测试套件。

## API使用示例

### 使用cURL

```bash
# 列出所有存储桶
curl -X GET https://yourdomain.com/s3-server/ \
  -H "X-Api-Key: your-api-key-here"

# 创建存储桶
curl -X PUT https://yourdomain.com/s3-server/my-bucket \
  -H "X-Api-Key: your-api-key-here"

# 上传文件
curl -X PUT https://yourdomain.com/s3-server/my-bucket/file.txt \
  -H "X-Api-Key: your-api-key-here" \
  -H "Content-Type: text/plain" \
  --data-binary @localfile.txt

# 下载文件
curl -X GET https://yourdomain.com/s3-server/my-bucket/file.txt \
  -H "X-Api-Key: your-api-key-here"
```

### 使用AWS CLI

配置AWS CLI以使用您的PHP S3服务器：

```bash
aws configure set aws_access_key_id AKIAIOSFODNN7EXAMPLE
aws configure set aws_secret_access_key your-secret-key-here
aws configure set default.region us-east-1

# 使用自定义端点
aws s3 ls --endpoint-url https://yourdomain.com/s3-server/
```

## 支持的API操作

### 服务级操作
- `GET /` - 列出所有存储桶

### 存储桶操作
- `PUT /{bucket}` - 创建存储桶
- `GET /{bucket}` - 列出存储桶中的对象
- `DELETE /{bucket}` - 删除存储桶
- `HEAD /{bucket}` - 检查存储桶是否存在

### 对象操作
- `PUT /{bucket}/{key}` - 上传对象
- `GET /{bucket}/{key}` - 下载对象
- `DELETE /{bucket}/{key}` - 删除对象
- `HEAD /{bucket}/{key}` - 获取对象元数据

## 配置选项

主要配置文件位于 `config/config.php`，包含以下重要设置：

- `max_file_size`: 最大文件上传大小
- `max_buckets`: 最大存储桶数量
- `public_access`: 是否允许匿名访问
- `rate_limit_requests`: 每小时请求限制

## 安全考虑

1. **文件权限**: 确保 `config/credentials.php` 文件权限设置为600
2. **HTTPS**: 在生产环境中始终使用HTTPS
3. **访问控制**: 根据需要配置IP白名单
4. **定期更新**: 定期更换访问密钥和密码

## 故障排除

### 常见问题

1. **500内部服务器错误**: 检查PHP错误日志和文件权限
2. **认证失败**: 验证访问密钥和密钥配置
3. **文件上传失败**: 检查磁盘空间和PHP上传限制
4. **CORS错误**: 确保.htaccess文件正确配置

### 日志文件

- 应用日志: `logs/s3-server.log`
- 错误日志: `logs/error.log`

## 许可证

本项目采用MIT许可证。详见LICENSE文件。

## 支持

如需技术支持或报告问题，请访问项目GitHub页面或联系开发团队。

---

**注意**: 这是一个轻量级实现，主要用于开发和测试环境。对于生产环境，建议使用专业的S3兼容存储解决方案。

