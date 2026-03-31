# Flask 目录浏览器与用户认证系统

一个基于 Flask 的 Web 应用，提供目录浏览、文件管理和用户认证功能。

## 功能特点

- 📁 **目录浏览** - 浏览 `public` 目录下的文件和文件夹
- 🔐 **用户认证** - 基于 Session 的登录/登出系统
- 👑 **管理员功能** - 管理员可以管理用户（创建、禁用、删除）
- 📤 **文件上传** - 登录用户可以上传文件
- 🗑️ **文件删除** - 支持单个删除和批量删除
- 🔑 **JWT 支持** - 可选的 JWT 令牌认证
- 📱 **响应式界面** - 支持自定义 `__view.html` 模板

## 快速开始

### 安装依赖

```bash
pip install -r requirements.txt
```

### 初始化数据库

首次运行需要初始化数据库并创建默认管理员账户：

```bash
python app.py --init-db
```

默认管理员：
- 用户名：`admin`
- 密码：`admin123`

### 启动服务

```bash
python app.py
```

服务默认运行在 `http://localhost:5000`

### 配置文件

创建 `config.ini` 文件自定义配置：

```ini
[server]
host = 0.0.0.0
port = 5000

[directory]
root = public
```

## 项目结构

```
.
├── app.py              # 主程序
├── requirements.txt    # 依赖列表
├── config.ini         # 配置文件（可选）
├── public/            # 根目录（自动创建）
├── users.db           # SQLite 数据库
├── cert.pem           # SSL 证书（可选）
└── key.pem            # SSL 私钥（可选）
```

## API 接口

### 认证相关

| 方法 | 端点 | 说明 | 权限 |
|------|------|------|------|
| POST | `/api/login` | 用户登录 | 公开 |
| POST | `/api/logout` | 用户登出 | 已登录 |
| GET | `/api/check-auth` | 检查认证状态 | 公开 |
| POST | `/api/change-password` | 修改密码 | 已登录 |
| POST | `/api/register` | 创建用户 | 管理员 |

### 用户管理（管理员）

| 方法 | 端点 | 说明 |
|------|------|------|
| GET | `/api/users` | 获取用户列表 |
| DELETE | `/api/users/<id>` | 删除用户 |
| POST | `/api/users/<id>/toggle-status` | 启用/禁用用户 |

### 文件操作

| 方法 | 端点 | 说明 | 权限 |
|------|------|------|------|
| GET | `/` 或 `/<path>` | 浏览文件/目录 | 公开 |
| GET | `/api/files` | 获取文件列表 | 公开 |
| POST | `/upload` | 上传文件 | 已登录 |
| POST | `/delete` | 删除文件/目录 | 已登录 |
| POST | `/delete-multiple` | 批量删除 | 已登录 |

## 目录浏览

- 访问根路径 `/` 显示 `public` 目录内容
- 如果存在 `public/__view.html`，将作为目录模板使用
- 模板中可通过 `window.__INITIAL_DATA__` 获取文件列表数据

## 安全说明

1. 生产环境请修改 `SECRET_KEY`
2. 首次登录后请修改默认管理员密码
3. 建议在生产环境使用 HTTPS（配置 `cert.pem` 和 `key.pem`）
4. 所有文件操作都有路径安全检查，防止目录遍历攻击