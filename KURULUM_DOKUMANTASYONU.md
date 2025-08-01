# Kurumsal Prosedür Asistanı (KPA) - Hosting Kurulum Rehberi

## 📋 Proje Özeti

**Kurumsal Prosedür Asistanı (KPA)**, Word dokümanlarından AI destekli soru-cevap sistemi sunan hibrit web uygulamasıdır.

### Teknoloji Yığını
- **Frontend**: React 18 + Tailwind CSS
- **Backend**: FastAPI (Python 3.11+)
- **Veritabanı**: MongoDB
- **AI/LLM**: Google Gemini 2.0 Flash
- **Vector Search**: FAISS + SentenceTransformer
- **Document Processing**: python-docx

---

## 🚀 Hızlı Kurulum (Üretim)

### 1. Sistem Gereksinimleri

```bash
# Minimum Sistem Gereksinimleri:
- CPU: 2 vCPU
- RAM: 4GB (önerilen: 8GB)
- Disk: 20GB SSD
- OS: Ubuntu 20.04+ / CentOS 8+ / Docker destekli sistem
```

### 2. Gerekli Yazılımlar

#### Ubuntu 24.04 LTS için Özel Kurulum

```bash
# Sistem güncellemesi
sudo apt update && sudo apt upgrade -y

# Temel geliştirme araçları
sudo apt install -y build-essential curl wget git vim nano htop

# Python 3.11+ (Ubuntu 24.04'te varsayılan Python 3.12)
sudo apt install -y python3 python3-pip python3-venv python3-dev

# Python versiyonunu kontrol edin
python3 --version  # Python 3.12.x çıktısı beklenir

# Node.js 20.x LTS kurulumu (Ubuntu 24.04 için önerilen)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# Node.js versiyonunu kontrol edin
node --version    # v20.x.x
npm --version     # 10.x.x

# Yarn package manager
npm install -g yarn

# PM2 process manager (production için)
npm install -g pm2

# Git konfigürasyonu (opsiyonel)
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

#### Diğer Dağıtımlar için

```bash
# Ubuntu/Debian (Eski sürümler) için:
sudo apt update
sudo apt install -y python3.11 python3.11-pip nodejs npm mongodb git

# CentOS/RHEL için:
sudo yum install -y python3.11 python3.11-pip nodejs npm mongodb-org git

# Arch Linux için:
sudo pacman -S python python-pip nodejs npm mongodb git

# macOS için (Homebrew):
brew install python@3.11 node mongodb/brew/mongodb-community git
```

### 3. MongoDB Kurulumu

#### Ubuntu 24.04 LTS için MongoDB 7.0 Kurulumu

```bash
# MongoDB GPG anahtarını import edin
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg \
   --dearmor

# MongoDB repository ekleyin (Ubuntu 24.04 "noble" için)
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

# Paket listesini güncelleyin
sudo apt update

# MongoDB Community Edition'ı kurun
sudo apt install -y mongodb-org

# MongoDB servisini başlatın ve enable edin
sudo systemctl start mongod
sudo systemctl enable mongod

# MongoDB servis durumunu kontrol edin
sudo systemctl status mongod

# MongoDB bağlantısını test edin
mongosh --eval "db.adminCommand('ismaster')"

# MongoDB versiyonunu kontrol edin
mongosh --eval "db.version()"  # 7.0.x çıktısı beklenir
```

#### Firewall Yapılandırması (Ubuntu 24.04)

```bash
# UFW firewall'ı aktifleştirin
sudo ufw enable

# SSH, HTTP, HTTPS portlarını açın
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# MongoDB ve backend portlarını yerel ağla sınırlayın (güvenlik için)
sudo ufw allow from 127.0.0.1 to any port 27017
sudo ufw allow from 127.0.0.1 to any port 8001

# UFW durumunu kontrol edin
sudo ufw status verbose
```

#### MongoDB Yapılandırma Dosyası (Ubuntu 24.04)

```bash
# MongoDB konfigürasyon dosyasını düzenleyin
sudo nano /etc/mongod.conf
```

```yaml
# /etc/mongod.conf - Ubuntu 24.04 için optimize edilmiş
storage:
  dbPath: /var/lib/mongodb
  journal:
    enabled: true

systemLog:
  destination: file
  logAppend: true
  path: /var/log/mongodb/mongod.log
  logRotate: reopen

net:
  port: 27017
  bindIp: 127.0.0.1

processManagement:
  timeZoneInfo: /usr/share/zoneinfo

# Güvenlik ayarları (production için)
security:
  authorization: enabled

# Memory ve performans ayarları
storage:
  wiredTiger:
    engineConfig:
      cacheSizeGB: 2  # RAM'inizin yarısı kadar
    collectionConfig:
      blockCompressor: snappy
    indexConfig:
      prefixCompression: true
```

#### Diğer Dağıtımlar için MongoDB

```bash
# Ubuntu 22.04 LTS için:
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo apt-key add -
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
sudo apt-get update
sudo apt-get install -y mongodb-org

# CentOS 9 / RHEL 9 için:
sudo tee /etc/yum.repos.d/mongodb-org-7.0.repo << 'EOF'
[mongodb-org-7.0]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/7.0/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-7.0.asc
EOF

sudo yum install -y mongodb-org
sudo systemctl start mongod
sudo systemctl enable mongod

# Docker ile MongoDB (platform bağımsız):
docker run -d --name mongodb \
  -p 27017:27017 \
  -v mongodb_data:/data/db \
  -e MONGO_INITDB_ROOT_USERNAME=admin \
  -e MONGO_INITDB_ROOT_PASSWORD=password123 \
  mongo:7.0
```

---

## 📦 Proje Kurulumu

### 1. Projeyi İndirin

#### Ubuntu 24.04 LTS için

```bash
# Proje dizini oluşturun
sudo mkdir -p /opt/kpa
sudo chown $USER:$USER /opt/kpa
cd /opt/kpa

# GitHub'dan clone edin (eğer reponuz varsa):
git clone https://github.com/[kullanici-adi]/kurumsal-prosedur-asistani.git .

# Veya zip dosyasını indirin ve çıkarın:
wget https://github.com/[kullanici-adi]/kurumsal-prosedur-asistani/archive/main.zip
unzip main.zip
mv kurumsal-prosedur-asistani-main/* .
rm -rf kurumsal-prosedur-asistani-main main.zip

# Veya yerel zip dosyasını çıkarın:
unzip kpa-project-final.zip
mv kpa-project/* .
rm -rf kpa-project

# Dizin izinlerini ayarlayın
sudo chown -R $USER:$USER /opt/kpa
chmod +x deploy.sh
```

#### Dosya Yapısını Kontrol Edin

```bash
# Proje yapısını görüntüleyin
tree /opt/kpa || ls -la /opt/kpa

# Beklenen yapı:
# /opt/kpa/
# ├── backend/
# ├── frontend/
# ├── docker-compose.yml
# ├── deploy.sh
# └── README.md
```

### 2. Backend Kurulumu (Ubuntu 24.04 LTS)

```bash
cd /opt/kpa/backend

# Python sanal ortam oluşturun (Ubuntu 24.04'te Python 3.12)
python3 -m venv venv

# Sanal ortamı aktifleştirin
source venv/bin/activate

# Python ve pip versiyonlarını kontrol edin
python --version  # Python 3.12.x
pip --version     # pip 24.x

# Pip'i güncelleyin
pip install --upgrade pip

# Sistem bağımlılıklarını kurun (gerekli C kütüphaneleri)
sudo apt install -y python3-dev python3-setuptools python3-wheel
sudo apt install -y build-essential libssl-dev libffi-dev libblas3 liblapack3
sudo apt install -y libatlas-base-dev gfortran  # NumPy/SciPy için

# Python bağımlılıklarını yükleyin
pip install -r requirements.txt

# Özel kütüphane kurulumu
pip install emergentintegrations --extra-index-url https://d33sy5i8bnduwe.cloudfront.net/simple/

# Kurulumu doğrulayın
python -c "import fastapi; print('FastAPI:', fastapi.__version__)"
python -c "import sentence_transformers; print('SentenceTransformers: OK')"
python -c "import faiss; print('FAISS: OK')"
python -c "from emergentintegrations.llm.chat import LlmChat; print('EmergentIntegrations: OK')"

# Environment variables ayarlayın
cp .env.example .env 2>/dev/null || true
nano .env
```

#### Backend Test ve Geliştirme

```bash
# Backend'i development modunda çalıştırın
cd /opt/kpa/backend
source venv/bin/activate

# Development server başlatın
uvicorn server:app --host 0.0.0.0 --port 8001 --reload

# Başka bir terminalde test edin
curl http://localhost:8001/api/status
```

### 3. Environment Variables (.env)

```bash
# backend/.env dosyası:
MONGO_URL="mongodb://localhost:27017"
DB_NAME="kpa_production"
GEMINI_API_KEY="[GOOGLE-GEMINI-API-ANAHTARINIZ]"

# Güvenlik için:
SECRET_KEY="[GUVENLI-RASTGELE-ANAHTAR-32-KARAKTER]"
ENVIRONMENT="production"
```

### 4. Frontend Kurulumu

```bash
cd ../frontend

# Node.js bağımlılıklarını yükleyin:
yarn install
# veya: npm install

# Production build oluşturun:
yarn build
# veya: npm run build

# Environment variables:
cp .env.example .env
nano .env
```

### 5. Frontend Environment Variables

```bash
# frontend/.env dosyası:
REACT_APP_BACKEND_URL=https://[DOMAIN-ADINIZ]/api
REACT_APP_ENV=production
```

---

## 🌐 Web Sunucusu Konfigürasyonu

### Nginx ile Reverse Proxy (Önerilen)

```bash
# Nginx kurulumu:
sudo apt install nginx

# Site konfigürasyonu:
sudo nano /etc/nginx/sites-available/kpa
```

```nginx
server {
    listen 80;
    server_name [DOMAIN-ADINIZ];
    
    # Frontend static files
    location / {
        root /path/to/kpa-project/frontend/build;
        try_files $uri $uri/ /index.html;
        
        # Cache static assets
        location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
    }
    
    # Backend API
    location /api {
        proxy_pass http://127.0.0.1:8001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # File upload settings
        client_max_body_size 50M;
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }
}
```

```bash
# Site'ı aktifleştir:
sudo ln -s /etc/nginx/sites-available/kpa /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

### SSL Sertifikası (Let's Encrypt)

```bash
# Certbot kurulumu:
sudo apt install certbot python3-certbot-nginx

# SSL sertifikası al:
sudo certbot --nginx -d [DOMAIN-ADINIZ]

# Otomatik yenileme:
sudo crontab -e
# Şu satırı ekleyin:
0 12 * * * /usr/bin/certbot renew --quiet
```

---

## 🔧 Servis Konfigürasyonu

### Systemd ile Backend Servisi

```bash
# Servis dosyası oluşturun:
sudo nano /etc/systemd/system/kpa-backend.service
```

```ini
[Unit]
Description=KPA Backend Service  
After=network.target mongodb.service

[Service]
Type=simple
User=www-data
WorkingDirectory=/path/to/kpa-project/backend
Environment=PATH=/path/to/kpa-project/backend/venv/bin
ExecStart=/path/to/kpa-project/backend/venv/bin/uvicorn server:app --host 0.0.0.0 --port 8001 --workers 4
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

```bash
# Servisi başlatın:
sudo systemctl daemon-reload
sudo systemctl enable kpa-backend
sudo systemctl start kpa-backend

# Servis durumunu kontrol edin:
sudo systemctl status kpa-backend
```

### PM2 ile Alternatif (Önerilen)

```bash
# PM2 kurulumu:
sudo npm install -g pm2

# Backend için PM2 konfigürasyonu:
cd /path/to/kpa-project/backend
```

```bash
# ecosystem.config.js oluşturun:
cat > ecosystem.config.js << 'EOF'
module.exports = {
  apps: [{
    name: 'kpa-backend',
    script: 'venv/bin/uvicorn',
    args: 'server:app --host 0.0.0.0 --port 8001 --workers 4',
    cwd: '/path/to/kpa-project/backend',
    instances: 1,
    autorestart: true,
    watch: false,
    max_memory_restart: '1G',
    env: {
      NODE_ENV: 'production'
    }
  }]
};
EOF

# PM2 ile başlatın:
pm2 start ecosystem.config.js
pm2 save
pm2 startup
```

---

## 🗃️ Veritabanı Konfigürasyonu

### MongoDB Güvenlik Ayarları

```bash
# MongoDB admin kullanıcısı oluşturun:
mongosh
```

```javascript
use admin
db.createUser({
  user: "kpa_admin",
  pwd: "GUVENLI_SIFRE_BURAYA",
  roles: ["userAdminAnyDatabase", "dbAdminAnyDatabase", "readWriteAnyDatabase"]
})

use kpa_production
db.createUser({
  user: "kpa_user", 
  pwd: "UYGULAMA_SIFRESI_BURAYA",
  roles: ["readWrite"]
})
```

```bash
# MongoDB authentication aktifleştirin:
sudo nano /etc/mongod.conf
```

```yaml
# /etc/mongod.conf
security:
  authorization: enabled
```

```bash
# MongoDB'yi yeniden başlatın:
sudo systemctl restart mongod

# .env dosyasını güncelleyin:
MONGO_URL="mongodb://kpa_user:UYGULAMA_SIFRESI_BURAYA@localhost:27017/kpa_production"
```

---

## 🔍 Monitoring ve Loglama

### Log Konfigürasyonu

```bash
# Log dizinleri oluşturun:
sudo mkdir -p /var/log/kpa
sudo chown www-data:www-data /var/log/kpa

# Logrotate konfigürasyonu:
sudo nano /etc/logrotate.d/kpa
```

```bash
/var/log/kpa/*.log {
    daily
    rotate 30
    compress
    delaycompress
    missingok
    notifempty
    create 644 www-data www-data
}
```

### Health Check Endpoint'i

```bash
# Backend health check:
curl http://localhost:8001/api/status

# Frontend check:
curl http://localhost/

# MongoDB check:
mongosh --eval "db.adminCommand('ping')"
```

---

## 🐳 Docker ile Kurulum (Alternatif)

### Dockerfile Oluşturma

```dockerfile
# backend/Dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
RUN pip install emergentintegrations --extra-index-url https://d33sy5i8bnduwe.cloudfront.net/simple/

COPY . .

EXPOSE 8001

CMD ["uvicorn", "server:app", "--host", "0.0.0.0", "--port", "8001"]
```

```dockerfile
# frontend/Dockerfile
FROM node:18-alpine as build

WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80
```

### Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  mongodb:
    image: mongo:6.0
    restart: always
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: password123
    volumes:
      - mongodb_data:/data/db

  backend:
    build: ./backend
    restart: always
    ports:
      - "8001:8001"
    environment:
      - MONGO_URL=mongodb://admin:password123@mongodb:27017/kpa_production?authSource=admin
      - GEMINI_API_KEY=${GEMINI_API_KEY}
    depends_on:
      - mongodb
    volumes:
      - ./backend:/app

  frontend:
    build: ./frontend
    restart: always
    ports:
      - "80:80"
    depends_on:
      - backend

volumes:
  mongodb_data:
```

```bash
# Docker ile çalıştırma:
docker-compose up -d

# Logları izleme:
docker-compose logs -f backend
```

---

## 📊 Performans Optimizasyonu

### Backend Optimizasyonu

```python
# server.py içinde eklenebilecek optimizasyonlar:

# Redis cache (isteğe bağlı):
import redis
redis_client = redis.Redis(host='localhost', port=6379, db=0)

# Connection pooling:
from motor.motor_asyncio import AsyncIOMotorClient
client = AsyncIOMotorClient(
    mongo_url, 
    maxPoolSize=20,
    minPoolSize=5
)

# Async processing:
from fastapi import BackgroundTasks
```

### Frontend Optimizasyonu

```json
// package.json build optimizasyonu:
{
  "scripts": {
    "build": "NODE_ENV=production npm run build:analyze",
    "build:analyze": "npm run build && npx webpack-bundle-analyzer build/static/js/*.js"
  }
}
```

### Database İndeksleme

```javascript
// MongoDB'de indeks oluşturma:
use kpa_production

// Documents collection indeksleri:
db.documents.createIndex({ "filename": 1 })
db.documents.createIndex({ "created_at": -1 })
db.documents.createIndex({ "embeddings_created": 1 })

// Chat sessions indeksleri:
db.chat_sessions.createIndex({ "session_id": 1, "created_at": -1 })
db.chat_sessions.createIndex({ "created_at": -1 })

// TTL indeks (chat geçmişi için):
db.chat_sessions.createIndex(
  { "created_at": 1 }, 
  { expireAfterSeconds: 2592000 } // 30 gün
)
```

---

## 🔧 Bakım ve Güncelleme

### Düzenli Bakım Görevleri

```bash
#!/bin/bash
# /opt/kpa/maintenance.sh

# Log temizleme:
find /var/log/kpa -name "*.log" -mtime +30 -delete

# MongoDB backup:
mongodump --uri="mongodb://kpa_user:SIFRE@localhost:27017/kpa_production" --out=/backup/mongodb/$(date +%Y%m%d)

# Disk kullanımı kontrolü:
df -h / | awk 'FNR==2 {print $5}' | sed 's/%//' | xargs -I {} sh -c 'if [ {} -gt 80 ]; then echo "Disk usage high: {}%"; fi'

# Servis health check:
curl -f http://localhost:8001/api/status || systemctl restart kpa-backend
```

```bash
# Crontab'a ekleme:
sudo crontab -e
# Şu satırları ekleyin:
0 2 * * * /opt/kpa/maintenance.sh
0 3 * * 0 /opt/kpa/backup.sh
```

### Güncelleme Süreci

```bash
#!/bin/bash
# /opt/kpa/update.sh

# Backup oluştur:
./backup.sh

# Git pull (eğer git kullanıyorsanız):
git pull origin main

# Backend güncelleme:
cd backend
source venv/bin/activate
pip install -r requirements.txt

# Frontend güncelleme:
cd ../frontend
yarn install
yarn build

# Servisleri yeniden başlat:
pm2 restart kpa-backend
sudo systemctl reload nginx

echo "Güncelleme tamamlandı!"
```

---

## 🚨 Sorun Giderme

### Yaygın Sorunlar

**1. Backend Başlamıyor:**
```bash
# Log kontrolü:
sudo journalctl -u kpa-backend -f

# Port kontrolü:
sudo netstat -tlnp | grep :8001

# Environment variables:
source backend/venv/bin/activate
python -c "import os; print(os.environ.get('GEMINI_API_KEY', 'NOT SET'))"
```

**2. MongoDB Bağlantı Sorunu:**
```bash
# MongoDB status:
sudo systemctl status mongod

# Connection test:
mongosh --eval "db.adminCommand('ping')"

# Log kontrolü:
sudo tail -f /var/log/mongodb/mongod.log
```

**3. Frontend Build Hatası:**
```bash
# Node.js versiyonu:
node --version  # 18+ olmalı

# Cache temizleme:
yarn cache clean
rm -rf node_modules
yarn install

# Memory artırma:
NODE_OPTIONS="--max-old-space-size=4096" yarn build
```

**4. AI Model Yükleme Hatası:**
```bash
# Python packages:
pip list | grep sentence-transformers
pip list | grep faiss

# Model indirme:
python -c "from sentence_transformers import SentenceTransformer; SentenceTransformer('all-MiniLM-L6-v2')"
```

### Debug Modu

```bash
# Backend debug:
cd backend
source venv/bin/activate
export DEBUG=True
uvicorn server:app --host 0.0.0.0 --port 8001 --reload --log-level debug

# Frontend debug:
cd frontend
yarn start
```

---

## 📞 Destek ve İletişim

### Log Dosyaları Konumları

- **Backend Logs**: `/var/log/kpa/backend.log`
- **Nginx Logs**: `/var/log/nginx/access.log`, `/var/log/nginx/error.log`
- **MongoDB Logs**: `/var/log/mongodb/mongod.log`
- **PM2 Logs**: `~/.pm2/logs/`

### Performans Metrikleri

```bash
# Sistem kaynakları:
htop
iotop
df -h

# PM2 monitoring:
pm2 monit

# Nginx status:
curl http://localhost/nginx_status
```

---

## 📝 Sürüm Notları

### v1.0.0 (İlk Sürüm)
- ✅ Word doküman işleme (.docx)
- ✅ Google Gemini 2.0 Flash entegrasyonu
- ✅ RAG sistemi (FAISS + SentenceTransformer)
- ✅ Modern React arayüzü
- ✅ MongoDB veritabanı
- ✅ Session tabanlı chat sistemi
- ✅ Responsive tasarım
- ✅ Türkçe dil desteği

### Gelecek Sürümler İçin Planlar
- 📄 PDF doküman desteği
- 📊 Excel dosya işleme
- 🔍 Gelişmiş arama filtreleri
- 📈 Kullanım analytics
- 🔐 Kullanıcı yetkilendirme sistemi
- 🌍 Çoklu dil desteği

---

## ⚖️ Lisans ve Güvenlik

### Güvenlik Önlemleri

```bash
# Firewall konfigürasyonu:
sudo ufw enable
sudo ufw allow ssh
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw deny 8001/tcp  # Backend portunu dışarıya kapatın
sudo ufw deny 27017/tcp # MongoDB portunu dışarıya kapatın

# SSL/TLS ayarları (nginx):
ssl_protocols TLSv1.2 TLSv1.3;
ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512;
ssl_prefer_server_ciphers off;
```

### API Key Güvenliği

```bash
# Environment variables güvenliği:
chmod 600 backend/.env
chown www-data:www-data backend/.env

# API key rotation:
# Gemini API anahtarınızı düzenli olarak yenileyin
```

Bu dokümantasyon Kurumsal Prosedür Asistanı (KPA) projesinin production ortamına kurulumu için hazırlanmıştır. Herhangi bir sorun yaşarsanız log dosyalarını kontrol edin ve gerekirse destek isteyin.