# 🚀 AI Gateway - LocNguyen AI Platform

Nền tảng AI tích hợp PHP + Python, cho phép truy cập 12 mô hình AI (OpenAI, Anthropic, Google, DeepSeek, Meta, Mistral) qua **một API key duy nhất**. Hỗ trợ OpenAI-compatible API, tương thích với mọi client hiện có (Open WebUI, ChatGPT clients, LangChain, v.v.).

## 🏗️ Kiến trúc

```
┌──────────────────────────────────────────────────────┐
│                     Docker                            │
│                                                       │
│  ┌──────────┐   ┌────────────┐   ┌───────────────┐   │
│  │  MySQL   │   │ PHP+Apache │   │ Python FastAPI│   │
│  │  :3306   │   │   :80      │   │   :8000       │   │
│  │ (shared) │   │ (Frontend) │   │ (AI Gateway)  │   │
│  └────┬─────┘   └─────┬──────┘   └──────┬────────┘   │
│       │               │                  │            │
│       └───────────────┴──────────────────┘            │
│                       │                               │
│           Dùng chung database MySQL                   │
│           API Keys, Users, Models, History            │
└──────────────────────────────────────────────────────┘
```

---

## 📋 Hướng dẫn cài đặt trên Ubuntu (từ A-Z)

> Hướng dẫn này dành cho **Ubuntu Server 20.04/22.04/24.04** mới cài, chưa có gì. Nếu bạn đã có sẵn Docker/Python thì bỏ qua các bước tương ứng.

---

### 🔹 Bước 1: Cập nhật hệ thống

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y curl wget git unzip
```

---

### 🔹 Bước 2: Cài Docker & Docker Compose

```bash
# Cài Docker
curl -fsSL https://get.docker.com | sudo sh

# Thêm user hiện tại vào group docker (khỏi cần sudo mỗi lần chạy)
sudo usermod -aG docker $USER

# Khởi động Docker
sudo systemctl enable docker
sudo systemctl start docker

# Kiểm tra
docker --version
# Docker version 27.x.x

# Cài Docker Compose plugin
sudo apt install -y docker-compose-plugin

# Kiểm tra
docker compose version
# Docker Compose version v2.x.x

# ⚠️ QUAN TRỌNG: Logout & login lại để group docker có hiệu lực
exit
# Rồi SSH/login lại
```

Sau khi login lại, kiểm tra Docker chạy không cần sudo:

```bash
docker ps
# Phải hiển thị bảng trống (không báo lỗi permission denied)
```

---

### 🔹 Bước 3: Mở port tường lửa (nếu dùng UFW)

```bash
# Mở port 80 (PHP Frontend) và 8000 (AI Gateway)
sudo ufw allow 80/tcp
sudo ufw allow 8000/tcp

# Nếu muốn truy cập MySQL từ xa (không khuyến khích)
# sudo ufw allow 3306/tcp

sudo ufw reload
sudo ufw status
```

---

### 🔹 Bước 4: Clone hoặc upload source code

**Cách A: Upload từ máy local lên server**

```bash
# Trên máy local (Windows), dùng SCP:
scp -r ai-gateway.zip user@your-server-ip:/home/user/

# Trên server Ubuntu, giải nén:
cd ~
unzip ai-gateway.zip -d ai-gateway
cd ai-gateway
```

**Cách B: Clone từ Git (nếu có)**

```bash
cd ~
git clone <your-repo-url> ai-gateway
cd ai-gateway
```

---

### 🔹 Bước 5: Build & Chạy bằng Docker

```bash
# Đảm bảo đang trong thư mục ai-gateway
cd ~/ai-gateway

# Build images và khởi động containers
docker compose up -d --build

# Đợi khoảng 30-60 giây cho MySQL khởi động xong
# Kiểm tra trạng thái
docker compose ps
```

Output mong đợi:
```
NAME                STATUS          PORTS
ai-gateway-mysql    running         0.0.0.0:3306->3306/tcp
ai-gateway-php      running         0.0.0.0:80->80/tcp
ai-gateway-python   running         0.0.0.0:8000->8000/tcp
```

---

### 🔹 Bước 6: Kiểm tra hoạt động

```bash
# Health check Python gateway
curl http://localhost:8000/health

# Trả về: {"status":"healthy","app":"AI Gateway","version":"1.0.0"}

# Kiểm tra PHP frontend
curl -I http://localhost/

# Trả về: HTTP/1.1 200 OK
```

---

### 🔹 Bước 7: Truy cập

| Service | URL |
|---------|-----|
| **PHP Frontend** | `http://YOUR_SERVER_IP/` |
| **Admin PHP** | `http://YOUR_SERVER_IP/admin/` |
| **AI Gateway Dashboard** | `http://YOUR_SERVER_IP:8000/dashboard` |
| **Health Check** | `http://YOUR_SERVER_IP:8000/health` |

---

## 🔑 Tài khoản mặc định

| Vai trò | Username | Password | Nơi đăng nhập |
|---------|----------|----------|---------------|
| Admin PHP | `locnguyen` | `Loc12345s@` | `http://IP/admin/` |
| Admin Gateway | `locnguyen` | `Loc12345s@` | `http://IP:8000/dashboard` |

---

## 🔧 Cấu hình AI Providers

Sau khi chạy, vào Gateway Dashboard để thêm API keys của các nhà cung cấp AI:

1. Mở `http://YOUR_SERVER_IP:8000/dashboard`
2. Đăng nhập: `locnguyen` / `Loc12345s@`
3. Vào mục **Providers** → Thêm API key cho từng provider:

| Provider | Cần API Key |
|----------|-------------|
| OpenAI | https://platform.openai.com/api-keys |
| Anthropic | https://console.anthropic.com/ |
| Google Gemini | https://aistudio.google.com/apikey |
| DeepSeek | https://platform.deepseek.com/api_keys |
| Meta Llama | https://console.groq.com/keys (qua Groq) |
| Mistral | https://console.mistral.ai/api-keys/ |

**Người dùng cuối chỉ cần 1 API key của hệ thống để gọi tất cả các model!**

---

## 🛠️ API Endpoints

### PHP API (Port 80)

```bash
# OpenAI-compatible Chat Completion
curl http://YOUR_SERVER_IP/api/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "model": "gpt-4o-mini",
    "messages": [{"role": "user", "content": "Hello!"}]
  }'

# List models
curl http://YOUR_SERVER_IP/api/v1/models \
  -H "Authorization: Bearer YOUR_API_KEY"
```

### Python AI Gateway (Port 8000)

```bash
# Chat Completion qua Gateway
curl http://YOUR_SERVER_IP:8000/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_GATEWAY_KEY" \
  -d '{
    "model": "gpt-4o-mini",
    "messages": [{"role": "user", "content": "Xin chào!"}],
    "stream": false
  }'

# List models từ Gateway
curl http://YOUR_SERVER_IP:8000/v1/models \
  -H "Authorization: Bearer YOUR_GATEWAY_KEY"
```

---

## 📊 Các mô hình AI được hỗ trợ

| # | Model | Provider | Context Window |
|---|-------|----------|---------------|
| 1 | GPT-4o | OpenAI | 128K |
| 2 | GPT-4o-mini | OpenAI | 128K |
| 3 | o1-mini | OpenAI | 128K |
| 4 | Claude 3.5 Sonnet | Anthropic | 200K |
| 5 | Claude 3 Opus | Anthropic | 200K |
| 6 | Claude 3 Haiku | Anthropic | 200K |
| 7 | Gemini 2.0 Flash | Google | 1M |
| 8 | Gemini 1.5 Pro | Google | 2M |
| 9 | DeepSeek V3 | DeepSeek | 64K |
| 10 | DeepSeek R1 | DeepSeek | 64K |
| 11 | Llama 3.1 70B | Meta (Groq) | 128K |
| 12 | Mixtral 8x7B | Mistral | 32K |

---

## 📁 Cấu trúc thư mục

```
~/ai-gateway/
├── docker-compose.yml      # Docker orchestration (3 services)
├── Dockerfile              # Python FastAPI image
├── Dockerfile.php          # PHP 8.2 + Apache image
├── .env                    # Environment variables
├── .env.example            # Template .env
├── .dockerignore           # Files excluded from Docker build
├── .gitignore
├── README.md               # File này
├── database.sql            # MySQL schema + seed data (12 models)
├── requirements.txt        # Python dependencies
├── main.py                 # Python entry point (FastAPI)
├── start.bat               # Windows quick start script
├── app/                    # Python Backend
│   ├── config.py           # Settings (reads from ENV)
│   ├── api/                # Route handlers (auth, proxy, dashboard, management)
│   ├── auth/               # JWT authentication & security
│   ├── database/           # SQLAlchemy connection + session management
│   ├── middleware/          # Security headers, logging, exception handling
│   ├── models/             # SQLAlchemy ORM models
│   ├── schemas/            # Pydantic schemas
│   └── services/           # Business logic
├── public/                 # PHP Frontend (Apache DocumentRoot)
│   ├── .htaccess           # Apache mod_rewrite rules
│   ├── index.php           # Landing page
│   ├── login.php           # User login
│   ├── register.php        # User registration
│   ├── home.php            # User dashboard
│   ├── dashboard.php       # Chat interface
│   ├── config/             # PHP configuration
│   │   ├── database.php    # MySQL connection (Docker-aware)
│   │   └── functions.php   # Helper functions
│   ├── admin/              # Admin panel
│   │   ├── index.php       # Admin login
│   │   ├── dashboard.php   # Admin dashboard
│   │   ├── models.php      # Model management
│   │   ├── settings.php    # System settings
│   │   ├── history.php     # Chat history
│   │   └── usage.php       # Usage statistics
│   ├── api/                # PHP API endpoints
│   │   ├── chat.php        # Chat processing
│   │   ├── keys.php        # API key management
│   │   ├── models.php      # Model listing
│   │   ├── history.php     # Chat history API
│   │   └── upload.php      # File upload
│   └── uploads/            # Uploaded files (Docker volume)
├── templates/              # Jinja2 HTML templates (Gateway dashboard)
│   ├── base.html
│   ├── login.html
│   ├── dashboard.html
│   ├── api_keys.html
│   ├── providers.html
│   └── logs.html
└── logs/                   # Application logs directory
    └── apache/             # Apache logs
```

---

## 🛑 Quản lý Docker

```bash
# Xem logs tất cả services
docker compose logs -f

# Xem logs 1 service cụ thể
docker compose logs -f php-apache
docker compose logs -f ai-gateway
docker compose logs -f mysql

# Restart 1 service
docker compose restart ai-gateway

# Dừng tất cả
docker compose down

# Dừng + xóa database (reset hoàn toàn)
docker compose down -v

# Build lại sau khi sửa code
docker compose up -d --build

# Xem tài nguyên đang dùng
docker stats
```

---

## 🔄 Cập nhật code

```bash
cd ~/ai-gateway

# Pull code mới (nếu dùng git)
git pull

# Hoặc upload file mới từ máy local
# Rồi rebuild:
docker compose up -d --build
```

---

## 💻 Chạy không dùng Docker (local dev)

### Yêu cầu
- Ubuntu 20.04+
- PHP 8.0+ + extensions: `pdo`, `pdo_mysql`, `mbstring`, `json`, `curl`
- MySQL 8.0+
- Python 3.11+

### Cài PHP & Apache

```bash
sudo apt install -y apache2 php php-cli php-mysql php-mbstring php-curl php-json php-xml
sudo a2enmod rewrite

# Sửa DocumentRoot trong /etc/apache2/sites-available/000-default.conf
# DocumentRoot /home/user/ai-gateway/public
# <Directory /home/user/ai-gateway/public>
#     AllowOverride All
#     Require all granted
# </Directory>

sudo systemctl restart apache2
```

### Cài MySQL & Import database

```bash
sudo apt install -y mysql-server
sudo systemctl enable mysql
sudo systemctl start mysql

# Tạo database
sudo mysql -e "CREATE DATABASE IF NOT EXISTS locnguyen_ai CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
sudo mysql -e "ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'root';"
sudo mysql -e "FLUSH PRIVILEGES;"

# Import schema
mysql -u root -proot < ~/ai-gateway/database.sql
```

### Cài Python & dependencies

```bash
sudo apt install -y python3 python3-pip python3-venv

cd ~/ai-gateway

# Tạo virtual environment
python3 -m venv venv
source venv/bin/activate

# Cài dependencies
pip install -r requirements.txt
```

### Chạy AI Gateway

```bash
cd ~/ai-gateway
source venv/bin/activate

# Copy .env từ template
cp .env.example .env

# Chỉnh sửa .env nếu cần (database URL, port...)
nano .env

# Chạy
python main.py
```

---

## 🐛 Xử lý lỗi thường gặp

### Port 80 bị chiếm

```bash
# Kiểm tra process đang dùng port 80
sudo lsof -i :80

# Nếu là Apache2/XAMPP:
sudo systemctl stop apache2
sudo systemctl disable apache2

# Hoặc đổi port PHP trong docker-compose.yml:
# ports:
#   - "8080:80"
```

### Port 8000 bị chiếm

```bash
sudo lsof -i :8000
# Kill process nếu cần:
# sudo kill -9 <PID>
```

### Port 3306 đã có MySQL local

```bash
sudo systemctl stop mysql
sudo systemctl disable mysql

# Hoặc đổi port trong docker-compose.yml:
# ports:
#   - "3307:3306"
```

### MySQL container không khởi động

```bash
# Xem logs
docker compose logs mysql

# Lỗi thường gặp: port 3306 đã dùng → tắt MySQL local
sudo systemctl stop mysql

# Hoặc xóa volume cũ:
docker compose down -v
docker compose up -d
```

### Permission denied khi chạy Docker

```bash
# Nếu chưa thêm user vào group docker:
sudo usermod -aG docker $USER
# Rồi LOGOUT & LOGIN LẠI (bắt buộc!)

# Kiểm tra:
groups
# Phải thấy "docker" trong danh sách
```

### Lỗi kết nối database từ PHP

```bash
# Vào container PHP kiểm tra:
docker exec -it ai-gateway-php bash

# Test kết nối MySQL:
apt-get update && apt-get install -y mysql-client
mysql -h mysql -u root -proot locnguyen_ai -e "SHOW TABLES;"

# Nếu không kết nối được → MySQL chưa sẵn sàng, đợi thêm vài giây
```

### Lỗi kết nối database từ Python

```bash
# Xem logs Python container:
docker compose logs ai-gateway

# Vào container Python kiểm tra:
docker exec -it ai-gateway-python bash
python -c "from app.config import settings; print(settings.database_url)"
# Phải ra: mysql+aiomysql://root:root@mysql:3306/locnguyen_ai
```

---

## ⚙️ Biến môi trường (.env)

| Biến | Mặc định | Mô tả |
|------|----------|-------|
| `SECRET_KEY` | `ai-gateway-locnguyen-2025-...` | Khóa bí mật cho ứng dụng |
| `JWT_SECRET` | `jwt-locnguyen-ai-gateway-...` | Khóa ký JWT token |
| `DATABASE_URL` | `mysql+aiomysql://root:...` | Database URL cho Python async |
| `SYNC_DATABASE_URL` | `mysql+pymysql://root:...` | Database URL cho Python sync |
| `HOST` | `0.0.0.0` | IP bind |
| `PORT` | `8000` | Cổng Gateway |
| `ADMIN_USERNAME` | `locnguyen` | Username admin mặc định |
| `ADMIN_PASSWORD` | `Loc12345s@` | Password admin mặc định |
| `ENVIRONMENT` | `production` | Môi trường (development/production) |
| `DEBUG` | `false` | Debug mode |
| `LOG_LEVEL` | `info` | Log level (debug/info/warning/error) |

---

## 🔒 Bảo mật

### Đổi mật khẩu admin mặc định

```bash
# Đổi mật khẩu admin PHP:
# 1. Đăng nhập vào http://IP/admin/
# 2. Vào Settings → Đổi mật khẩu

# Đổi mật khẩu admin Gateway:
# 1. Đăng nhập vào http://IP:8000/dashboard
# 2. Vào mục Account → Change Password
```

### Đổi SECRET_KEY & JWT_SECRET trước khi deploy production

```bash
# Tạo key ngẫu nhiên
openssl rand -hex 32

# Sửa trong file .env hoặc docker-compose.yml
nano .env
# Đổi SECRET_KEY và JWT_SECRET
```

---

## 📞 Hỗ trợ

Nếu gặp lỗi trong quá trình cài đặt, kiểm tra logs:

```bash
# Xem toàn bộ logs
docker compose logs -f

# Lưu logs ra file để debug
docker compose logs > debug-logs.txt 2>&1
```

---

**Made with ❤️ by LocNguyen**
