# PHP S3 Server 安装指南

本文档提供了在cPanel免费版虚拟主机上安装和配置PHP S3 Server的详细步骤。

## 系统要求

### 最低要求

- **PHP版本**: 7.4或更高版本
- **PHP扩展**: 
  - json (JSON处理)
  - xml (XML响应生成)
  - mbstring (多字节字符串处理)
- **Web服务器**: Apache (支持.htaccess和mod_rewrite)
- **磁盘空间**: 至少50MB可用空间
- **内存**: 建议128MB PHP内存限制

### 推荐配置

- **PHP版本**: 8.0或更高版本
- **磁盘空间**: 1GB或更多（用于存储文件）
- **内存**: 256MB PHP内存限制
- **执行时间**: 300秒或更多（用于大文件上传）

## 安装步骤

### 步骤1: 准备文件

1. 下载PHP S3 Server的所有文件
2. 确保文件结构完整：

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

### 步骤2: 上传到cPanel主机

#### 使用文件管理器

1. 登录到您的cPanel控制面板
2. 打开"文件管理器"
3. 导航到 `public_html` 目录（或您希望安装的子目录）
4. 创建一个新文件夹，例如 `s3-server`
5. 将所有文件上传到该文件夹

#### 使用FTP客户端

```bash
# 使用FTP上传文件
ftp your-domain.com
# 输入用户名和密码
cd public_html
mkdir s3-server
cd s3-server
# 上传所有文件
```

### 步骤3: 设置文件权限

正确的文件权限对于安全性和功能性至关重要：

#### 通过cPanel文件管理器

1. 选择 `php-s3-server` 目录
2. 右键点击 → "权限"
3. 设置为 755 (rwxr-xr-x)
4. 对以下目录重复此操作：
   - `storage/` → 755
   - `storage/buckets/` → 755
   - `logs/` → 755
   - `config/` → 755

5. 对文件设置权限：
   - 所有 `.php` 文件 → 644 (rw-r--r--)
   - `config/credentials.php` → 600 (rw-------)
   - `.htaccess` → 644

#### 通过SSH（如果可用）

```bash
# 设置目录权限
chmod 755 php-s3-server/
chmod 755 php-s3-server/storage/
chmod 755 php-s3-server/storage/buckets/
chmod 755 php-s3-server/logs/
chmod 755 php-s3-server/config/

# 设置文件权限
find php-s3-server/ -name "*.php" -exec chmod 644 {} \;
chmod 600 php-s3-server/config/credentials.php
chmod 644 php-s3-server/.htaccess
```

### 步骤4: 配置PHP设置

#### 检查PHP版本和扩展

创建一个临时的 `phpinfo.php` 文件来检查您的PHP配置：

```php
<?php
phpinfo();
?>
```

上传并访问此文件，确认：
- PHP版本 ≥ 7.4
- 已安装 json, xml, mbstring 扩展

#### 调整PHP设置（如果需要）

如果您有权限修改PHP设置，可以在 `.htaccess` 文件中添加：

```apache
# PHP设置优化
php_value memory_limit 128M
php_value max_execution_time 300
php_value upload_max_filesize 100M
php_value post_max_size 100M
php_value max_input_time 300
```

### 步骤5: 配置认证凭据

编辑 `config/credentials.php` 文件：

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

**重要安全提示**:
- 使用强密码和随机密钥
- 定期更换认证凭据
- 不要在公共场所分享这些信息

### 步骤6: 调整主配置

编辑 `config/config.php` 以适应您的主机环境：

```php
return [
    // 根据您的主机限制调整
    'max_file_size' => 50 * 1024 * 1024, // 50MB
    'max_buckets' => 20,
    'default_bucket_quota' => 500 * 1024 * 1024, // 500MB
    
    // 安全设置
    'public_access' => false,
    'require_https' => true, // 生产环境推荐
    
    // 性能设置
    'memory_limit' => '128M',
    'max_execution_time' => 300,
];
```

### 步骤7: 测试安装

#### 运行内置测试

访问 `https://yourdomain.com/s3-server/test.php` 运行自动测试套件。

成功的测试输出应显示：
```
✓ All components initialized successfully
✓ Bucket created
✓ Object uploaded
✓ All tests completed successfully!
```

#### 手动API测试

使用cURL测试基本API功能：

```bash
# 测试服务器响应
curl -X GET https://yourdomain.com/s3-server/ \
  -H "X-Api-Key: YOUR_API_KEY"

# 应该返回空的存储桶列表XML
```

### 步骤8: SSL/HTTPS配置

#### 通过cPanel启用SSL

1. 在cPanel中找到"SSL/TLS"
2. 启用"强制HTTPS重定向"
3. 确保有效的SSL证书已安装

#### 更新配置以要求HTTPS

在 `config/config.php` 中设置：

```php
'require_https' => true,
```

## 高级配置

### URL重写配置

确保 `.htaccess` 文件正确配置了URL重写规则：

```apache
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php [QSA,L]
```

### 日志配置

配置日志记录以便于调试：

```php
// 在 config/config.php 中
'debug' => false, // 生产环境设为false
'log_level' => 'info',
'max_log_size' => 10 * 1024 * 1024, // 10MB
```

### 性能优化

#### 启用压缩

在 `.htaccess` 中启用gzip压缩：

```apache
<IfModule mod_deflate.c>
    AddOutputFilterByType DEFLATE text/plain
    AddOutputFilterByType DEFLATE application/xml
    AddOutputFilterByType DEFLATE application/json
</IfModule>
```

#### 缓存配置

为静态资源设置缓存头：

```apache
<IfModule mod_expires.c>
    ExpiresActive On
    ExpiresByType application/xml "access plus 1 hour"
    ExpiresByType application/json "access plus 1 hour"
</IfModule>
```

## 故障排除

### 常见安装问题

#### 1. 500内部服务器错误

**可能原因**:
- 文件权限不正确
- PHP语法错误
- 缺少PHP扩展

**解决方案**:
```bash
# 检查错误日志
tail -f logs/error.log

# 验证文件权限
ls -la config/credentials.php  # 应该是 -rw-------
```

#### 2. 认证失败

**可能原因**:
- 认证凭据配置错误
- 时间同步问题

**解决方案**:
- 验证 `config/credentials.php` 中的密钥
- 检查服务器时间设置

#### 3. 文件上传失败

**可能原因**:
- PHP上传限制
- 磁盘空间不足
- 权限问题

**解决方案**:
```php
// 检查PHP限制
echo ini_get('upload_max_filesize');
echo ini_get('post_max_size');

// 检查磁盘空间
echo disk_free_space('.');
```

#### 4. URL重写不工作

**可能原因**:
- mod_rewrite未启用
- .htaccess权限问题

**解决方案**:
- 联系主机提供商启用mod_rewrite
- 确保.htaccess文件权限为644

### 调试技巧

#### 启用调试模式

在 `config/config.php` 中临时启用：

```php
'debug' => true,
'show_errors' => true,
```

#### 查看日志文件

```bash
# 查看应用日志
tail -f logs/s3-server.log

# 查看错误日志
tail -f logs/error.log
```

#### 测试PHP配置

创建测试脚本：

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

## 安全最佳实践

### 1. 文件权限

- 配置文件: 600 (仅所有者可读写)
- PHP文件: 644 (所有者可读写，其他人只读)
- 目录: 755 (所有者全权限，其他人可读执行)

### 2. 访问控制

```apache
# 在.htaccess中保护敏感目录
<Files "config/*">
    Order deny,allow
    Deny from all
</Files>

<Files "logs/*">
    Order deny,allow
    Deny from all
</Files>
```

### 3. 定期维护

- 定期更换访问密钥
- 监控日志文件大小
- 检查磁盘空间使用情况
- 更新PHP版本（如果可能）

## 升级和维护

### 备份配置

在升级前备份重要文件：

```bash
# 备份配置文件
cp config/credentials.php config/credentials.php.backup
cp config/config.php config/config.php.backup

# 备份存储数据
tar -czf storage-backup.tar.gz storage/
```

### 监控性能

定期检查：
- 磁盘空间使用情况
- 日志文件大小
- 响应时间
- 错误率

通过遵循这个详细的安装指南，您应该能够成功在cPanel免费版虚拟主机上部署PHP S3 Server。如果遇到问题，请参考故障排除部分或查看日志文件以获取更多信息。

