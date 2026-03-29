# 第九章：交付——部署与上线

> **核心哲学**：从"在我电脑上能跑"到"别人打开链接就能用"

---

## 9.0 部署的基本概念

部署（Deployment）是把开发好的项目放到服务器上，让其他人可以通过网络访问。理解部署的关键概念，才能选择合适的部署方案。

### 核心概念

| 概念 | 含义 | 示例 |
|------|------|------|
| **服务器** | 运行项目的计算机，有公网 IP | 云服务器（阿里云 ECS）、VPS |
| **域名** | 便于记忆的网址，指向服务器 IP | `example.com` → `123.45.67.89` |
| **DNS** | 域名解析服务，把域名翻译成 IP | Cloudflare、阿里云 DNS |
| **端口** | 服务器上的服务入口 | 80（HTTP）、443（HTTPS）、8080（自定义） |
| **Nginx** | Web 服务器，处理 HTTP 请求 | 反向代理、静态文件服务 |
| **HTTPS** | 加密的 HTTP，需要 SSL 证书 | Let's Encrypt 免费证书 |
| **进程管理** | 保持服务持续运行 | systemd、pm2、supervisor |

### 部署流程概览

```
┌─────────────────────────────────────────────────────────────────────┐
│  部署流程                                                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  1. 本地开发                                                         │
│     └── 代码编写、测试、调试                                          │
│                                                                      │
│  2. 代码托管                                                         │
│     └── 推送到 GitHub/GitLab                                         │
│                                                                      │
│  3. 服务器准备                                                        │
│     └── 购买云服务器、配置环境                                        │
│                                                                      │
│  4. 部署代码                                                         │
│     └── git clone 或 CI/CD 自动部署                                  │
│                                                                      │
│  5. 启动服务                                                         │
│     └── 后端 API、前端静态文件                                        │
│                                                                      │
│  6. 域名配置                                                         │
│     └── 购买域名、DNS 解析、HTTPS 证书                               │
│                                                                      │
│  7. 对外访问                                                         │
│     └── 用户通过域名访问项目                                          │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 部署方式对比

| 方式 | 复杂度 | 成本 | 灵活性 | 适用场景 |
|------|--------|------|--------|----------|
| **静态托管** | 低 | 低/免费 | 低 | 纯前端项目、文档站 |
| **PaaS 平台** | 低 | 中 | 中 | 快速部署、原型验证 |
| **云服务器** | 中 | 中 | 高 | 需要完整控制权的项目 |
| **容器化部署** | 高 | 中 | 最高 | 微服务、需要环境一致性 |

---

## 9.1 前端部署：让网页可访问

前端项目通常是静态文件（HTML、CSS、JS），部署相对简单。核心是把构建后的文件放到能被访问的地方。

### 前端部署选项对比

| 方式 | 特点 | 适用场景 |
|------|------|----------|
| **GitHub Pages** | 免费、简单 | 文档站、个人博客、静态展示 |
| **Vercel / Netlify** | 免费、自动部署、支持 SSR | React/Vue/Next.js 项目 |
| **云服务器 + Nginx** | 完全控制、可配合后端 | 需要自定义配置的项目 |
| **云存储 + CDN** | 高性能、低成本 | 静态资源、图片、视频 |

### 方式一：静态托管（GitHub Pages）

适合纯静态网站（HTML + CSS + JS），无需后端。

```bash
# === GitHub Pages 部署流程 ===

# 1. 构建项目
npm run build           # 生成 dist/ 或 build/ 目录

# 2. 推送到 GitHub
git add .
git commit -m "准备部署"
git push origin main

# 3. 在 GitHub 仓库设置中启用 Pages
# Settings → Pages → Source: Deploy from a branch → main / (root) 或 /docs

# 4. 访问地址
# https://username.github.io/repo-name/

# === 使用 gh-pages 分支（更灵活）===
# 安装 gh-pages 工具
npm install --save-dev gh-pages

# package.json 添加脚本
{
  "scripts": {
    "deploy": "gh-pages -d dist"
  }
}

# 执行部署
npm run deploy

# 自动创建 gh-pages 分支并部署
```

### 方式二：Vercel / Netlify（推荐）

适合现代前端框架（React、Vue、Next.js），自动构建部署、支持 Serverless 函数。

**Vercel 部署**：

```bash
# === Vercel 部署流程 ===

# 方式一：通过网页（推荐新手）
# 1. 登录 vercel.com
# 2. Import Project → 选择 GitHub 仓库
# 3. 自动检测框架，点击 Deploy
# 4. 部署完成后获得 .vercel.app 域名

# 方式二：命令行
# 安装 Vercel CLI
npm install -g vercel

# 登录
vercel login

# 部署
cd your-project
vercel

# 生产部署
vercel --prod

# 访问地址
# https://your-project.vercel.app
```

**Netlify 部署**：

```bash
# === Netlify 部署流程 ===

# 方式一：通过网页
# 1. 登录 netlify.com
# 2. Add new site → Import an existing project
# 3. 连接 GitHub，选择仓库
# 4. 配置构建命令和输出目录
#    Build command: npm run build
#    Publish directory: dist 或 build
# 5. Deploy site

# 方式二：命令行
npm install -g netlify-cli
netlify login
netlify deploy --prod

# 访问地址
# https://your-site.netlify.app
```

**Vercel vs Netlify 对比**：

| 特性 | Vercel | Netlify |
|------|--------|---------|
| Next.js 支持 | 原生优化（同公司） | 支持 |
| 免费额度 | 100GB 带宽/月 | 100GB 带宽/月 |
| Serverless 函数 | 支持 | 支持 |
| 自定义域名 | 免费 | 免费 |
| 国内访问 | 较慢 | 较慢 |
| 推荐场景 | Next.js 项目 | 通用前端项目 |

### 方式三：云服务器 + Nginx

适合需要完全控制、配合后端 API 的项目。

```bash
# === 云服务器部署前端 ===

# 1. 在本地构建项目
npm run build           # 生成 dist/ 目录

# 2. 上传到服务器
scp -r dist/* user@your-server:/var/www/html/

# 或用 rsync（更快，支持增量）
rsync -avz dist/ user@your-server:/var/www/html/

# 3. 服务器上配置 Nginx
sudo nano /etc/nginx/sites-available/myapp
```

**Nginx 配置示例**：

```nginx
# /etc/nginx/sites-available/myapp

server {
    listen 80;
    server_name your-domain.com;  # 或服务器 IP

    # 前端静态文件
    root /var/www/html;
    index index.html;

    # SPA 路由支持（所有路径返回 index.html）
    location / {
        try_files $uri $uri/ /index.html;
    }

    # 静态资源缓存
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # Gzip 压缩
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml;
}
```

```bash
# 启用站点配置
sudo ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/

# 测试配置
sudo nginx -t

# 重载 Nginx
sudo systemctl reload nginx

# 访问
# http://your-domain.com 或 http://your-server-ip
```

### 前端环境变量处理

前端构建时注入环境变量（API 地址、配置等）：

```bash
# === 环境变量配置 ===

# 方式一：.env 文件（Vite/Next.js 支持）
# .env.production
VITE_API_URL=https://api.example.com
VITE_APP_TITLE=我的应用

# 构建时自动注入
npm run build

# 方式二：构建时传入
VITE_API_URL=https://api.example.com npm run build

# 在代码中使用
const apiUrl = import.meta.env.VITE_API_URL;
```

---

## 9.2 后端部署：让 API 可访问

后端部署需要保持服务持续运行，处理 HTTP 请求，通常与数据库配合。

### 后端部署选项对比

| 方式 | 特点 | 适用场景 |
|------|------|----------|
| **PaaS 平台** | 简单、自动扩展 | 小型项目、快速验证 |
| **云服务器** | 完全控制、灵活 | 中大型项目、需要定制 |
| **Docker 部署** | 环境一致、易迁移 | 团队协作、微服务 |
| **Kubernetes** | 自动扩展、高可用 | 大规模生产环境 |

### 方式一：云服务器部署（传统方式）

```bash
# === 云服务器部署 Python 后端 ===

# 1. 连接服务器
ssh user@your-server-ip

# 2. 安装依赖
sudo apt update
sudo apt install -y python3 python3-pip python3-venv nginx

# 3. 克隆代码
git clone https://github.com/user/project.git
cd project

# 4. 创建虚拟环境
python3 -m venv .venv
source .venv/bin/activate

# 5. 安装依赖
pip install -r requirements.txt
# 或
pip install uv && uv sync

# 6. 配置环境变量
cp .env.example .env
nano .env  # 填入真实配置

# 7. 测试运行
python run_server.py

# 8. 配置进程管理（见下一节）
```

### 进程管理：保持服务运行

直接运行 `python run_server.py` 在关闭终端后会停止。需要进程管理工具。

**方式一：systemd（推荐 Linux 系统服务）**

```bash
# === systemd 服务配置 ===

# 创建服务文件
sudo nano /etc/systemd/system/myapp.service
```

```ini
# /etc/systemd/system/myapp.service

[Unit]
Description=My Python Application
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/home/user/project
Environment="PATH=/home/user/project/.venv/bin"
ExecStart=/home/user/project/.venv/bin/python run_server.py
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

```bash
# 启用并启动服务
sudo systemctl daemon-reload
sudo systemctl enable myapp     # 开机自启
sudo systemctl start myapp      # 启动服务

# 管理命令
sudo systemctl status myapp     # 查看状态
sudo systemctl restart myapp    # 重启
sudo systemctl stop myapp       # 停止
sudo journalctl -u myapp -f     # 查看日志
```

**方式二：Supervisor（Python 进程管理）**

```bash
# === Supervisor 配置 ===

# 安装
sudo apt install supervisor

# 创建配置
sudo nano /etc/supervisor/conf.d/myapp.conf
```

```ini
# /etc/supervisor/conf.d/myapp.conf

[program:myapp]
directory=/home/user/project
command=/home/user/project/.venv/bin/python run_server.py
user=www-data
autostart=true
autorestart=true
stderr_logfile=/var/log/myapp.err.log
stdout_logfile=/var/log/myapp.out.log
```

```bash
# 管理命令
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start myapp
sudo supervisorctl status myapp
sudo supervisorctl restart myapp
sudo supervisorctl stop myapp
```

**方式三：pm2（Node.js 项目）**

```bash
# === pm2 配置 ===

# 安装
npm install -g pm2

# 启动
pm2 start server.js --name myapp

# 常用命令
pm2 list
pm2 logs myapp
pm2 restart myapp
pm2 stop myapp

# 开机自启
pm2 startup
pm2 save
```

### Nginx 反向代理

后端服务通常监听本地端口（如 8000），通过 Nginx 代理对外提供 80/443 端口服务。

```nginx
# /etc/nginx/sites-available/myapp

server {
    listen 80;
    server_name api.example.com;

    # 反向代理到后端
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # WebSocket 支持（如果需要）
    location /ws {
        proxy_pass http://127.0.0.1:8000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### 方式二：Docker 部署

Docker 部署确保环境一致性，适合团队协作和 CI/CD。

```bash
# === Docker 部署后端 ===

# 1. 编写 Dockerfile（已在第五章讲解）
# 2. 构建镜像
docker build -t myapp:v1 .

# 3. 运行容器
docker run -d \
  --name myapp \
  -p 8000:8000 \
  -e DATABASE_URL=postgresql://... \
  -e SECRET_KEY=... \
  --restart unless-stopped \
  myapp:v1

# 4. 使用 docker-compose（推荐）
docker-compose up -d

# 5. 查看日志
docker logs -f myapp
docker-compose logs -f

# 6. 更新部署
docker-compose pull          # 拉取新镜像
docker-compose up -d         # 重新创建容器
docker image prune -f        # 清理旧镜像
```

**生产环境 docker-compose.yml**：

```yaml
# docker-compose.prod.yml

services:
  api:
    image: registry.example.com/myapp:v1
    ports:
      - "127.0.0.1:8000:8000"  # 只监听本地，由 Nginx 代理
    environment:
      - DATABASE_URL=postgresql://app:secret@db:5432/appdb
    env_file:
      - .env
    restart: always
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:8000/health"]
      interval: 30s
      timeout: 10s
      retries: 3

  db:
    image: postgres:16-alpine
    volumes:
      - pgdata:/var/lib/postgresql/data
    environment:
      POSTGRES_USER: app
      POSTGRES_PASSWORD: secret
      POSTGRES_DB: appdb
    restart: always

volumes:
  pgdata:
```

---

## 9.3 前后端联调部署

实际项目通常是前端 + 后端 + 数据库的组合。需要正确配置它们之间的通信。

### 架构方案

```
┌─────────────────────────────────────────────────────────────────────┐
│  方案一：同一服务器部署                                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  用户浏览器                                                          │
│       │                                                              │
│       ▼                                                              │
│  ┌─────────┐                                                         │
│  │ Nginx   │ ← 处理所有请求                                          │
│  │ :80/443 │                                                         │
│  └────┬────┘                                                         │
│       │                                                              │
│       ├──────────────────┐                                           │
│       │                  │                                           │
│       ▼                  ▼                                           │
│  ┌─────────┐       ┌─────────┐                                       │
│  │ 前端    │       │ 后端    │                                       │
│  │ 静态文件│       │ :8000   │                                       │
│  └─────────┘       └────┬────┘                                       │
│                         │                                            │
│                         ▼                                            │
│                    ┌─────────┐                                       │
│                    │ 数据库   │                                       │
│                    │ :5432   │                                       │
│                    └─────────┘                                       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  方案二：前后端分离部署                                              │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  用户浏览器                                                          │
│       │                                                              │
│       ├──────────────────────┐                                       │
│       │                      │                                       │
│       ▼                      ▼                                       │
│  前端托管服务             后端服务器                                 │
│  (Vercel/Netlify)         (云服务器)                                 │
│  app.example.com          api.example.com                            │
│                           (Nginx → 后端 → 数据库)                    │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 方案一：同服务器部署配置

```nginx
# /etc/nginx/sites-available/myapp

server {
    listen 80;
    server_name example.com;

    # 前端静态文件
    location / {
        root /var/www/frontend;
        try_files $uri $uri/ /index.html;
    }

    # API 请求代理到后端
    location /api/ {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # 其他后端路由（如 WebSocket）
    location /ws/ {
        proxy_pass http://127.0.0.1:8000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### 方案二：前后端分离部署

**前端（Vercel）**：

```javascript
// 前端配置 API 地址
// .env.production
NEXT_PUBLIC_API_URL=https://api.example.com

// 或在代码中
const API_URL = process.env.NEXT_PUBLIC_API_URL || 'http://localhost:8000';

// API 调用
fetch(`${API_URL}/api/users`)
```

**后端（云服务器）**：

```python
# FastAPI 配置 CORS（允许前端域名访问）

from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "https://app.example.com",      # 前端域名
        "https://example.com",
        "http://localhost:3000",        # 本地开发
    ],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

---

## 9.5 Windows 服务器部署

Windows 服务器在企业和一些场景中仍然常见，部署方式与 Linux 有所不同。

### 远程连接 Windows 服务器

| 方式 | 适用场景 | 操作 |
|------|----------|------|
| **远程桌面（RDP）** | 图形界面操作 | Windows 自带 mstsc |
| **PowerShell 远程** | 命令行操作 | Enter-PSSession |
| **SSH** | 跨平台 | Windows OpenSSH 服务 |

**远程桌面连接**：

```bash
# === Windows 远程桌面 ===

# 1. 在本地 Windows 上打开远程桌面
# Win + R → mstsc → 回车

# 2. 输入服务器 IP 或域名
# 计算机: your-server-ip

# 3. 输入用户名和密码
# 用户名: Administrator 或其他账户

# 4. 连接成功后，像操作本地电脑一样操作服务器
```

**PowerShell 远程**：

```powershell
# === PowerShell 远程管理 ===

# 启用远程（在服务器上执行）
Enable-PSRemoting -Force

# 客户端连接
Enter-PSSession -ComputerName your-server-ip -Credential Administrator

# 执行命令
Get-Service
Get-Process
```

### Windows 服务器环境配置

```powershell
# === Windows 服务器环境配置 ===

# 1. 安装 Python
# 下载 https://www.python.org/downloads/
# 或用 winget（Windows 11 / Server 2022）
winget install Python.Python.3.12

# 2. 安装 Node.js
winget install OpenJS.NodeJS.LTS

# 3. 安装 Git
winget install Git.Git

# 4. 安装 Nginx（Windows 版本）
# 下载 http://nginx.org/en/download.html
# 解压到 C:\nginx

# 5. 配置环境变量
# 系统属性 → 高级 → 环境变量
# 或用 PowerShell
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";C:\Python312", "Machine")
```

### Windows 服务管理

```powershell
# === Windows 服务管理 ===

# 方式一：注册为 Windows 服务（推荐）

# 使用 NSSM（Non-Sucking Service Manager）
# 下载 https://nssm.cc/download

# 安装服务
nssm install MyPythonApp "C:\Python312\python.exe" "C:\app\run_server.py"
nssm set MyPythonApp AppDirectory "C:\app"
nssm set MyPythonApp DisplayName "My Python Application"
nssm set MyPythonApp Description "Backend API Service"
nssm set MyPythonApp Start SERVICE_AUTO_START

# 启动服务
nssm start MyPythonApp

# 管理命令
nssm status MyPythonApp
nssm restart MyPythonApp
nssm stop MyPythonApp
nssm remove MyPythonApp

# 方式二：用 sc 命令（Windows 原生）
sc create MyService binPath= "C:\app\run.bat" start= auto
sc start MyService
sc stop MyService
sc delete MyService
```

### Windows 防火墙配置

```powershell
# === Windows 防火墙配置 ===

# 开放端口
New-NetFirewallRule -DisplayName "HTTP" -Direction Inbound -LocalPort 80 -Protocol TCP -Action Allow
New-NetFirewallRule -DisplayName "HTTPS" -Direction Inbound -LocalPort 443 -Protocol TCP -Action Allow
New-NetFirewallRule -DisplayName "API" -Direction Inbound -LocalPort 8000 -Protocol TCP -Action Allow

# 查看规则
Get-NetFirewallRule | Where-Object {$_.Enabled -eq $true}

# 删除规则
Remove-NetFirewallRule -DisplayName "API"
```

### Windows + WSL 部署

在 Windows 服务器上使用 WSL 可以获得 Linux 开发体验：

```powershell
# === Windows Server + WSL 部署 ===

# 1. 安装 WSL
wsl --install

# 2. 安装 Ubuntu
wsl --install -d Ubuntu-22.04

# 3. 进入 WSL
wsl

# 4. 在 WSL 中按 Linux 方式部署
# （参考前面的 Linux 部署流程）

# 5. 配置 systemd（WSL 2 支持）
# /etc/wsl.conf
[boot]
systemd=true

# 6. 重启 WSL
wsl --shutdown
wsl
```

---

## 9.6 远程开发与控制

除了部署，日常开发中经常需要远程连接到其他电脑或服务器。

### 远程连接方式对比

| 方式 | 适用场景 | 特点 |
|------|----------|------|
| **RDP（远程桌面）** | Windows 图形操作 | 完整桌面体验 |
| **SSH** | Linux/服务器命令行 | 轻量、安全 |
| **VNC** | 跨平台图形界面 | 需要额外安装 |
| **TeamViewer/向日葵** | 跨平台远程协助 | 穿透内网、易用 |
| **VS Code Remote** | 远程开发 | IDE 集成 |

### SSH 远程开发

```bash
# === SSH 远程开发 ===

# 1. 生成密钥对（推荐，免密码登录）
ssh-keygen -t ed25519 -C "your_email@example.com"

# 2. 复制公钥到服务器
ssh-copy-id user@your-server-ip

# 3. 配置 SSH 别名
# ~/.ssh/config
Host myserver
    HostName your-server-ip
    User username
    IdentityFile ~/.ssh/id_ed25519

# 4. 连接
ssh myserver

# 5. 文件传输
scp file.txt myserver:/home/user/
scp myserver:/home/user/file.txt ./

# 6. 端口转发（访问内网服务）
ssh -L 8080:localhost:8080 myserver
# 本地访问 localhost:8080 → 服务器 localhost:8080

# 7. 保持连接（防止断开）
# ~/.ssh/config
Host *
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

### VS Code Remote 开发

```bash
# === VS Code Remote 开发 ===

# 1. 安装 Remote-SSH 扩展

# 2. 配置 SSH 连接
# F1 → Remote-SSH: Connect to Host → myserver

# 3. 连接后体验
# - 代码在服务器上
# - 终端在服务器上
# - 扩展安装在服务器上
# - 编辑在本地 VS Code

# 4. 端口转发
# 端口面板 → 添加端口 → 8080
# 自动创建 localhost:8080 → server:8080 的转发

# 5. 远程调试
# 在服务器上运行代码
# 本地 VS Code 断点调试
```

### 远程桌面到另一台 Windows 电脑

```bash
# === 远程桌面到 Windows 电脑 ===

# 1. 目标电脑启用远程桌面
# 设置 → 系统 → 远程桌面 → 启用

# 2. 获取目标电脑 IP
ipconfig    # 在目标电脑上执行

# 3. 从本地连接
# Win + R → mstsc
# 输入目标电脑 IP

# 4. 如果目标电脑在内网
# 方式一：VPN 连接后远程桌面
# 方式二：使用向日葵、TeamViewer 等穿透工具
# 方式三：SSH 隧道 + RDP
ssh -L 3390:target-pc-ip:3389 jump-server
# 然后连接 localhost:3390
```

---

## 9.7 微服务架构简介

随着项目规模增长，单体应用可能演变为微服务架构。

### 单体 vs 微服务

```
┌─────────────────────────────────────────────────────────────────────┐
│  单体架构                                                            │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────────────────────────────────────────┐                    │
│  │              单一应用程序                    │                    │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐       │                    │
│  │  │ 用户模块 │ │ 订单模块 │ │ 支付模块 │       │                    │
│  │  └─────────┘ └─────────┘ └─────────┘       │                    │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐       │                    │
│  │  │ 商品模块 │ │ 库存模块 │ │ 消息模块 │       │                    │
│  │  └─────────┘ └─────────┘ └─────────┘       │                    │
│  │                                             │                    │
│  │           共享数据库                        │                    │
│  └─────────────────────────────────────────────┘                    │
│                                                                      │
│  优点：简单、易部署、易调试                                          │
│  缺点：难以扩展、技术栈受限、单点故障风险                              │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────┐
│  微服务架构                                                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐                          │
│  │ 用户服务 │    │ 订单服务 │    │ 支付服务 │                          │
│  │ :8001   │    │ :8002   │    │ :8003   │                          │
│  │ Python  │    │ Go      │    │ Java    │                          │
│  └────┬────┘    └────┬────┘    └────┬────┘                          │
│       │              │              │                                │
│       ▼              ▼              ▼                                │
│  ┌─────────┐    ┌─────────┐    ┌─────────┐                          │
│  │ 用户 DB  │    │ 订单 DB  │    │ 支付 DB  │                          │
│  └─────────┘    └─────────┘    └─────────┘                          │
│                                                                      │
│  ┌─────────────────────────────────────┐                            │
│  │          API 网关 / 负载均衡         │                            │
│  └─────────────────────────────────────┘                            │
│                                                                      │
│  优点：独立部署、独立扩展、技术栈灵活、故障隔离                        │
│  缺点：复杂度高、分布式问题、运维成本大                                │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

### 微服务关键组件

| 组件 | 作用 | 常见工具 |
|------|------|----------|
| **API 网关** | 统一入口、路由、认证 | Nginx、Kong、APISIX |
| **服务注册/发现** | 服务地址管理 | Consul、Nacos、Eureka |
| **配置中心** | 集中管理配置 | Nacos、Apollo、Consul |
| **消息队列** | 异步通信、解耦 | RabbitMQ、Kafka、Redis |
| **链路追踪** | 请求链路监控 | Jaeger、Zipkin、SkyWalking |
| **容器编排** | 服务部署管理 | Docker Compose、Kubernetes |

### 什么时候需要微服务

| 项目阶段 | 推荐架构 | 原因 |
|----------|----------|------|
| 初创/原型 | **单体架构** | 快速迭代，简单可靠 |
| 中型项目 | **单体 + 模块化** | 清晰分层，必要时拆分 |
| 大型/高并发 | **微服务** | 独立扩展，团队分工 |
| 遗留系统改造 | **绞杀者模式** | 逐步拆分，平滑迁移 |

**不建议过早微服务**：
- 增加部署复杂度
- 分布式事务问题
- 调试困难
- 运维成本高

### 微服务部署示例

```yaml
# docker-compose.yml（简化版微服务）

services:
  # API 网关
  gateway:
    image: nginx:alpine
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    depends_on:
      - user-service
      - order-service

  # 用户服务
  user-service:
    build: ./services/user
    environment:
      - DATABASE_URL=postgresql://user-db:5432/users
    depends_on:
      - user-db

  # 订单服务
  order-service:
    build: ./services/order
    environment:
      - DATABASE_URL=postgresql://order-db:5432/orders
      - USER_SERVICE_URL=http://user-service:8001
    depends_on:
      - order-db

  # 用户数据库
  user-db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: users

  # 订单数据库
  order-db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: orders
```

---

## 9.8 域名与 HTTPS

### 域名购买与配置

```bash
# === 域名配置流程 ===

# 1. 购买域名
# 常见域名注册商：阿里云、腾讯云、Cloudflare、Namecheap

# 2. DNS 解析配置
# 在域名管理控制台添加 DNS 记录：

# A 记录（指向 IPv4 地址）
# 主机记录: @ 或 www
# 记录值: 你的服务器 IP（如 123.45.67.89）

# CNAME 记录（指向另一个域名）
# 主机记录: api
# 记录值: your-project.vercel.app

# 3. 等待生效（几分钟到几小时）
# 检查解析是否生效
ping your-domain.com
dig your-domain.com
```

**常用 DNS 记录类型**：

| 类型 | 用途 | 示例 |
|------|------|------|
| A | 域名 → IPv4 | `@ → 123.45.67.89` |
| AAAA | 域名 → IPv6 | `@ → 2001:db8::1` |
| CNAME | 域名 → 域名 | `www → example.com` |
| MX | 邮件服务器 | `@ → mail.example.com` |
| TXT | 文本记录（验证、SPF） | `_verification → xxx` |

### HTTPS 配置（Let's Encrypt 免费证书）

```bash
# === HTTPS 证书配置 ===

# 1. 安装 Certbot
sudo apt install certbot python3-certbot-nginx

# 2. 获取证书
sudo certbot --nginx -d example.com -d www.example.com

# 按提示输入邮箱，同意条款

# 3. 自动续期测试
sudo certbot renew --dry-run

# Certbot 会自动配置 Nginx 的 HTTPS

# 4. 手动配置（如果需要）
```

**Nginx HTTPS 配置**：

```nginx
server {
    listen 80;
    server_name example.com;
    # HTTP 重定向到 HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com;

    # SSL 证书（Certbot 自动配置）
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    # SSL 优化配置
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;

    # HSTS
    add_header Strict-Transport-Security "max-age=63072000" always;

    # ... 其他配置
}
```

---

## 9.9 CI/CD：自动化部署

手动部署容易出错，CI/CD 实现代码推送后自动构建部署。

### GitHub Actions 自动部署

```yaml
# .github/workflows/deploy.yml

name: Deploy

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Deploy to server
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /home/user/project
            git pull origin main
            source .venv/bin/activate
            pip install -r requirements.txt
            sudo systemctl restart myapp
```

**配置 Secrets**：

在 GitHub 仓库 Settings → Secrets 中添加：
- `SERVER_HOST`: 服务器 IP
- `SERVER_USER`: SSH 用户名
- `SSH_PRIVATE_KEY`: SSH 私钥

### Vercel/Netlify 自动部署

连接 GitHub 后，每次推送自动触发部署，无需额外配置。

---

## 9.10 部署检查清单

### 部署前检查

```bash
# === 部署前检查清单 ===

# 代码
□ 所有改动已提交
□ 测试已通过
□ 敏感信息已移到环境变量

# 配置
□ .env 配置正确
□ 数据库连接正确
□ API 地址正确

# 安全
□ 没有硬编码密码
□ 没有调试模式（DEBUG=False）
□ 依赖版本已锁定

# 文档
□ README 更新
□ API 文档更新
```

### 部署后验证

```bash
# === 部署后验证 ===

# 1. 健康检查
curl https://your-domain.com/health

# 2. API 测试
curl https://api.your-domain.com/api/test

# 3. 前端访问
# 浏览器打开 https://your-domain.com

# 4. 日志检查
sudo journalctl -u myapp -f

# 5. 性能检查
# 浏览器开发者工具 Network 面板
# 检查加载时间、资源大小

# 6. HTTPS 检查
# 浏览器地址栏显示锁图标
# https://www.ssllabs.com/ssltest/ 测试 SSL 配置
```

---

## 9.11 常见部署平台对比

| 平台 | 类型 | 免费额度 | 国内访问 | 适用场景 |
|------|------|----------|----------|----------|
| **Vercel** | 前端 PaaS | 100GB/月 | 较慢 | Next.js、React |
| **Netlify** | 前端 PaaS | 100GB/月 | 较慢 | 静态站点、JAMstack |
| **GitHub Pages** | 静态托管 | 免费 | 较慢 | 文档、博客 |
| **Railway** | 全栈 PaaS | $5/月额度 | 中等 | 小型全栈应用 |
| **Render** | 全栈 PaaS | 有限免费 | 较慢 | Node.js、Python |
| **云服务器** | IaaS | 无 | 快 | 完全控制、生产环境 |
| **Docker 云托管** | 容器服务 | 无 | 快 | 容器化应用 |

### 国内部署推荐

| 平台 | 特点 | 适用场景 |
|------|------|----------|
| **阿里云 ECS** | 稳定、生态完善 | 企业级应用 |
| **腾讯云 CVM** | 价格有竞争力 | 个人项目 |
| **华为云 ECS** | 政企合规 | 企业项目 |
| **七牛云** | 对象存储、CDN | 静态资源 |
| **又拍云** | CDN 加速 | 静态站点 |

---

## 9.12 关键认知

### 三个核心原则

1. **部署是为了让用户能用**：选择用户能快速访问的方案，而非最简单的方案
2. **HTTPS 是标配**：没有 HTTPS 的网站会被浏览器标记为不安全
3. **自动化减少错误**：CI/CD 让部署可重复、可追溯

### 部署方案选择

```
项目类型？
├─ 纯静态前端
│   ├─ 文档/博客 → GitHub Pages
│   └─ SPA/SSR → Vercel/Netlify
│
├─ 前端 + 后端 API
│   ├─ 个人项目/原型 → Vercel + Railway
│   └─ 生产环境 → 云服务器（前后端同部署或分离）
│
└─ 复杂后端服务
    ├─ 单体应用 → 云服务器 + Docker
    └─ 微服务 → Kubernetes
```

### 常见陷阱

| 陷阱 | 症状 | 解决方案 |
|------|------|----------|
| CORS 错误 | 前端无法调用 API | 配置后端 CORS 允许前端域名 |
| HTTPS 混合内容 | 部分资源加载失败 | 所有资源都用 HTTPS |
| 环境变量错误 | API 地址不对 | 检查 .env 配置 |
| 端口未开放 | 无法访问服务 | 检查防火墙规则 |
| 进程意外停止 | 服务中断 | 使用 systemd/supervisor 自动重启 |
| 磁盘满 | 服务异常 | 定期清理日志、监控磁盘 |

---

> **结语**：从本地开发到用户可访问，部署是项目交付的最后一步。选择合适的部署方案，配置好域名和 HTTPS，你的项目就能真正服务用户了。