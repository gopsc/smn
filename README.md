# Flask 文件浏览器系统

一个功能完整的文件浏览器系统，支持用户认证、文件管理、HTTP/WebSocket代理和审计日志功能。

## ✨ 主要功能

### 🔐 用户管理
- 用户注册和登录
- 密码强度验证（长度、大小写、数字）
- 登录失败限制（5次失败锁定15分钟）
- 会话管理（24小时有效期）
- 管理员权限控制

### 📁 文件管理
- 文件/目录浏览
- 文件上传（支持多种格式）
- 文件/目录重命名、移动、删除
- 批量删除
- 文件下载

### 🔀 代理功能
- HTTP/HTTPS代理请求
- WebSocket代理（通过Socket.IO）
- 目标地址白名单控制
- 代理状态监控

### 📊 审计与监控
- 文件操作日志记录
- 登录尝试记录
- 系统统计信息
- 健康检查接口

## 🚀 快速开始

### 环境要求

- Python 3.7+
- pip

### 安装依赖

```bash
pip install flask flask-cors flask-sqlalchemy flask-limiter flask-socketio werkzeug requests websocket-client
```

### 配置

创建 `config.ini` 配置文件（可选）：

```ini
[server]
host = 0.0.0.0
port = 5000

[directory]
root = /path/to/your/files

[ssl]
enabled = false
cert_file = cert.pem
key_file = key.pem

[cors]
allowed_origins = http://localhost:5000,http://127.0.0.1:5000

[security]
max_content_length_mb = 100
session_lifetime_hours = 24
rate_limit_default = 200 per day;50 per hour

[proxy]
enabled = true
allowed_targets = http://localhost:8000,http://localhost:8080,ws://localhost:8765
```

### 初始化数据库

```bash
python app.py --init-db
```

这将创建SQLite数据库和默认管理员账户：
- 用户名：`admin`
- 密码：`Admin@123456`

**⚠️ 重要：请在生产环境中立即修改默认密码！**

### 启动服务

```bash
# 使用配置文件启动
python app.py --config config.ini

# 不使用配置文件（文件系统功能需配置）
python app.py
```

## 📖 API文档

### 认证相关

#### 用户登录
```http
POST /api/login
Content-Type: application/json

{
    "username": "admin",
    "password": "Admin@123456"
}
```

#### 用户注册（需管理员权限）
```http
POST /api/register
Authorization: (会话Cookie)

{
    "username": "newuser",
    "password": "Password123"
}
```

#### 修改密码
```http
POST /api/change-password
Authorization: (会话Cookie)

{
    "old_password": "Admin@123456",
    "new_password": "NewPassword123"
}
```

### 文件管理

#### 获取文件列表
```http
GET /api/files
Authorization: (会话Cookie)
```

#### 上传文件
```http
POST /upload
Authorization: (会话Cookie)
Content-Type: multipart/form-data

file: (文件)
path: (可选) 上传目录
```

#### 创建文件夹
```http
POST /api/folders
Authorization: (会话Cookie)

{
    "name": "new_folder",
    "path": "parent/path"  // 可选
}
```

#### 重命名
```http
POST /api/rename
Authorization: (会话Cookie)

{
    "old_path": "oldname.txt",
    "new_name": "newname.txt",
    "type": "file"  // 或 "dir"
}
```

#### 移动文件/目录
```http
POST /api/move
Authorization: (会话Cookie)

{
    "source_paths": ["file1.txt", "folder1"],
    "target_path": "destination/"
}
```

#### 删除（单个）
```http
POST /delete
Authorization: (会话Cookie)

{
    "name": "filename.txt",
    "recursive": false  // 删除目录时是否递归
}
```

#### 批量删除
```http
POST /delete-multiple
Authorization: (会话Cookie)

{
    "items": ["file1.txt", "folder1"],
    "recursive": false
}
```

### 代理功能

#### HTTP代理
```http
GET/POST/PUT/DELETE /proxy/<target_url>
Authorization: (会话Cookie)

# 示例：
GET /proxy/http://localhost:8000/api/data
```

#### WebSocket代理（通过Socket.IO）
```javascript
// 连接WebSocket
const socket = io('http://localhost:5000');

// 连接目标WebSocket
socket.emit('ws_connect', {
    target_url: 'ws://localhost:8765'
});

// 发送消息
socket.emit('ws_send', {
    message: 'Hello WebSocket!'
});

// 接收消息
socket.on('ws_message', (data) => {
    console.log('Received:', data);
});
```

### 管理功能（需管理员权限）

#### 获取用户列表
```http
GET /api/users
```

#### 删除用户
```http
DELETE /api/users/{user_id}
```

#### 切换用户状态
```http
POST /api/users/{user_id}/toggle-status
```

#### 获取审计日志
```http
GET /api/audit/file-operations?page=1&per_page=50
GET /api/audit/login-attempts?page=1&per_page=50
```

#### 系统统计
```http
GET /api/stats
```

### 系统监控

#### 健康检查
```http
GET /health
```

#### 文件系统状态
```http
GET /api/filesystem-status
```

#### 代理状态
```http
GET /api/proxy/status
Authorization: (会话Cookie)
```

## 🔧 配置说明

### 安全配置
- **密码策略**：最小长度8，必须包含大小写字母和数字
- **登录限制**：5次失败后锁定15分钟
- **会话超时**：24小时
- **文件上传限制**：默认100MB

### 代理安全
- 目标地址白名单控制
- 代理操作需登录认证
- 请求头自动添加用户标识（`X-Proxy-User`, `X-Proxy-User-ID`）

## 📝 日志文件

系统自动生成 `app.log` 文件，采用轮转日志：
- 最大文件大小：10MB
- 保留备份数：10个

## 🔒 安全建议

1. **立即修改默认密码**：首次启动后立即修改admin密码
2. **使用HTTPS**：生产环境务必启用SSL
3. **限制代理目标**：仅添加可信的代理目标到白名单
4. **定期审计日志**：检查异常操作和登录尝试
5. **备份数据库**：定期备份 `instance/users.db` 文件
6. **限制访问IP**：可通过防火墙限制访问来源

## 🐛 故障排除

### 数据库连接问题
```bash
# 检查数据库文件权限
ls -la instance/users.db

# 手动验证数据库
sqlite3 instance/users.db "SELECT * FROM users;"
```

### 文件系统权限
确保配置的根目录具有正确的读写权限：
```bash
chmod 755 /path/to/your/files
```

### 代理连接问题
- 检查目标地址是否在白名单中
- 确认目标服务正在运行
- 查看防火墙设置

## 📄 许可证

本项目遵循MIT许可证。

## 🤝 贡献

欢迎提交Issue和Pull Request！

## 📧 联系方式

如有问题，请通过GitHub Issues联系。
