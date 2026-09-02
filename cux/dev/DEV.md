## 项目结构

```
sub2api/
├── backend/                  # Go 后端服务
│   ├── cmd/server/           # 应用入口
│   ├── internal/             # 内部模块
│   │   ├── config/           # 配置管理
│   │   ├── model/            # 数据模型
│   │   ├── service/          # 业务逻辑
│   │   ├── handler/          # HTTP 处理器
│   │   └── gateway/          # API 网关核心
│   └── resources/            # 静态资源
│
├── frontend/                 # Vue 3 前端
│   └── src/
│       ├── api/              # API 调用
│       ├── stores/           # 状态管理
│       ├── views/            # 页面组件
│       └── components/       # 通用组件
│
└── deploy/                   # 部署文件
    ├── docker-compose.yml    # Docker Compose 配置
    ├── .env.example          # Docker Compose 环境变量
    ├── config.example.yaml   # 二进制部署完整配置文件
    └── install.sh            # 一键安装脚本
```


## 前期准备

**`.env` 必须配置项：**

```bash
# PostgreSQL 密码（必需）
POSTGRES_PASSWORD=your_secure_password_here

# JWT 密钥（推荐 - 重启后保持用户登录状态）
JWT_SECRET=your_jwt_secret_here

# TOTP 加密密钥（推荐 - 重启后保留双因素认证）
TOTP_ENCRYPTION_KEY=your_totp_key_here

# 可选：管理员账号
ADMIN_EMAIL=admin@example.com
ADMIN_PASSWORD=your_admin_password

# 可选：自定义端口
SERVER_PORT=8080
```

**生成安全密钥：**

```bash
# 生成 JWT_SECRET
openssl rand -hex 32

# 生成 TOTP_ENCRYPTION_KEY
openssl rand -hex 32

# 生成 POSTGRES_PASSWORD
openssl rand -hex 32
```


## 编译步骤

```bash
cd sub2api

# 2. 安装 pnpm（如果还没有安装）
npm install -g pnpm

# 3. 编译前端
cd frontend
pnpm install
pnpm run build
# 构建产物输出到 ../backend/internal/web/dist/

# 4. 编译后端（嵌入前端）
cd ../backend
VERSION="$(./scripts/resolve-version.sh)"
go build -tags embed -ldflags="-X main.Version=${VERSION}" -o sub2api ./cmd/server

# 5. 创建配置文件
cp ../deploy/config.example.yaml ./config.yaml

# 6. 编辑配置
nano config.yaml
```

> **注意：** `-tags embed` 参数会将前端嵌入到二进制文件中。不使用此参数编译的程序将不包含前端界面。

**`config.yaml` 关键配置：**

```yaml
server:
  host: "0.0.0.0"
  port: 8080
  mode: "release"

database:
  host: "localhost"
  port: 5432
  user: "postgres"
  password: "your_password"
  dbname: "sub2api"

redis:
  host: "localhost"
  port: 6379
  password: ""

jwt:
  secret: "change-this-to-a-secure-random-string"
  expire_hour: 24

default:
  user_concurrency: 5
  user_balance: 0
  api_key_prefix: "sk-"
  rate_multiplier: 1.0

default:
  # 初始管理员账户（首次运行时创建）
  admin_email: "lutpot@123.net"
  admin_password: "Aa!23456"

totp:
  # Generate with / 生成命令: openssl rand -hex 32
  encryption_key: "12aa52102f513931c594233d715b9b578ee2763f5dfcdeae6331b8d9c4a8d379"
```

## 本地启动

**中间件数据库**
```bash
cd cux/dev

docker compose -f docker-compose-db.yml up -d

docker compose -f docker-compose-db.yml ps

```

**前后端代码**
```bash
# 后端（支持热重载）
cd backend
$env:GOPROXY = "https://goproxy.cn,direct"

go run ./cmd/server

# 前端（支持热重载）
cd frontend
pnpm run dev
```