# 📚 Panduan Instalasi MobSF - Lengkap dari Awal sampai Akhir

Dokumentasi lengkap instalasi Mobile Security Framework (MobSF) dengan berbagai metode.

---

## 📋 Daftar Isi

- [Prasyarat Sistem](#-prasyarat-sistem)
- [Method 1: Docker Run (Tercepat)](#method-1-docker-run-tercepat-)
- [Method 2: Docker Compose](#method-2-docker-compose)
- [Method 3: Instalasi Manual](#method-3-instalasi-manual)
- [Verifikasi Instalasi](#-verifikasi-instalasi)
- [Troubleshooting](#-troubleshooting)
- [Command Reference](#-command-reference)

---

## 💻 Prasyarat Sistem

### Hardware Requirements

| Component | Minimum | Recommended |
|-----------|---------|-------------|
| CPU | 2 cores | 4+ cores |
| RAM | 4 GB | 8+ GB |
| Storage | 10 GB | 20+ GB |

### Software Requirements

- **OS:** Linux (Ubuntu 20.04+ / Debian 11+)
- **Docker:** Version 20.10+
- **Docker Compose:** V2 (included in Docker)

---

## Method 1: Docker Run (Tercepat) ⭐

**Waktu setup: ~5 menit**

Metode ini paling mudah dan direkomendasikan untuk pemula.

### Step 1: Install Docker

```bash
# Update package list
sudo apt update

# Install Docker
sudo apt install docker.io -y

# Start dan enable Docker service
sudo systemctl start docker
sudo systemctl enable docker

# Verifikasi instalasi
docker --version
# Output: Docker version 27.x.x, build xxxxx
```

### Step 2: Tambahkan User ke Docker Group (Optional tapi Recommended)

```bash
# Tambahkan user ke docker group
sudo usermod -aG docker $USER

# Verifikasi
groups $USER
# Output akan menampilkan: ... docker ...

# Aktifkan group di session saat ini
newgrp docker

# ATAU logout dan login lagi untuk efek permanent
```

**Setelah step ini, Anda bisa menjalankan Docker tanpa `sudo`**

### Step 3: Run MobSF Container

#### **Option A: Basic Setup (Data Tidak Tersimpan)**

Container akan otomatis dihapus saat di-stop. Cocok untuk testing.

```bash
sudo docker run -it --rm -d \
  -p 8000:8000 \
  --name mobsf \
  opensecurity/mobile-security-framework-mobsf:latest
```

#### **Option B: Persistent Data (Recommended)**

Data akan tetap tersimpan meskipun container di-stop atau dihapus.

```bash
# Buat Docker volume
sudo docker volume create mobsf_data

# Run dengan volume mount
sudo docker run -it -d \
  -p 8000:8000 \
  -v mobsf_data:/home/mobsf/.MobSF \
  --name mobsf \
  opensecurity/mobile-security-framework-mobsf:latest
```

**Penjelasan Parameters:**

| Parameter | Fungsi |
|-----------|--------|
| `-it` | Interactive terminal mode |
| `--rm` | Auto remove container saat stop (JANGAN pakai untuk data persistent) |
| `-d` | Detached mode (run in background) |
| `-p 8000:8000` | Port mapping: localhost:8000 → container:8000 |
| `-v mobsf_data:/home/mobsf/.MobSF` | Mount volume untuk persistent storage |
| `--name mobsf` | Nama container (untuk management) |
| `opensecurity/...` | Official MobSF Docker image |

### Step 4: Tunggu Download & Initialization

```bash
# Container akan download image (~2-3 GB) di first run
# Tunggu sekitar 2-5 menit tergantung koneksi internet

# Cek status download
sudo docker ps

# Output yang diharapkan:
# CONTAINER ID   IMAGE                                                  STATUS
# abc123def456   opensecurity/mobile-security-framework-mobsf:latest   Up 2 minutes
```

### Step 5: Dapatkan Credentials

```bash
# Lihat logs untuk mendapatkan username, password, dan API key
sudo docker logs mobsf

# Output akan menampilkan:
# ==========================================
# MOBSF Static Analyzer listening on http://0.0.0.0:8000
# 
# User credentials:
#    Username: mobsf
#    Password: mobsf
# 
# REST API key:
#    b1465e304310292923ee7f27cc6d874bf776a078713c4710de927ccccad791b1
# ==========================================
```

### Step 6: Akses MobSF

1. Buka browser (Chrome/Firefox)
2. Akses URL: **http://localhost:8000**
3. Login dengan:
   - **Username:** `mobsf`
   - **Password:** `mobsf`

**🎉 Installation Complete!**

### Step 7: Upload & Analyze APK (Testing)

1. Download sample APK untuk testing (atau gunakan APK Anda sendiri)
2. Di dashboard MobSF, klik **"Upload & Analyze"**
3. Pilih file APK
4. Klik **"Analyze"**
5. Tunggu proses analisis selesai (2-10 menit tergantung ukuran APK)
6. Lihat hasil analisis dan security report

---

## Method 2: Docker Compose

**Waktu setup: ~15 menit**

Setup multi-container dengan PostgreSQL database dan Nginx reverse proxy.

### Step 1: Pastikan Docker dan Docker Compose Terinstall

```bash
# Cek Docker
docker --version

# Cek Docker Compose V2
docker compose version
# Output: Docker Compose version v2.30.3
```

### Step 2: Navigate ke Project Directory

```bash
cd /home/haninayy/MovingForward/security-mobile
```

### Step 3: Review File docker-compose.yml

File `docker/docker-compose.yml` berisi konfigurasi untuk:
- **PostgreSQL** - Database untuk menyimpan hasil analisis
- **MobSF** - Aplikasi utama
- **Django-Q** - Background worker untuk processing
- **Nginx** - Reverse proxy

### Step 4: Setup Environment Variables (Optional)

```bash
# Buat file .env untuk konfigurasi custom
cat > docker/.env << 'EOF'
# Database Configuration
POSTGRES_DB=mobsf
POSTGRES_USER=mobsf
POSTGRES_PASSWORD=SecurePasswordHere123!

# MobSF Configuration
MOBSF_DB_ENGINE=postgresql
MOBSF_DB_NAME=mobsf
MOBSF_DB_USER=mobsf
MOBSF_DB_PASSWORD=SecurePasswordHere123!
MOBSF_DB_HOST=postgres
EOF
```

### Step 5: Buat Folder untuk Data Persistence

```bash
# Buat folder dengan permission yang benar
mkdir -p $HOME/MobSF/mobsf_data
mkdir -p $HOME/MobSF/postgresql_data

# Set ownership (UID 9901 = mobsf user di container)
sudo chown -R 9901:9901 $HOME/MobSF/mobsf_data
```

### Step 6: Run Docker Compose

```bash
# Build dan start semua services
sudo docker compose -f docker/docker-compose.yml up -d

# Output yang diharapkan:
# [+] Running 5/5
#  ✔ Network docker_mobsf_network  Created
#  ✔ Container docker-postgres-1   Started
#  ✔ Container docker-djangoq-1    Started
#  ✔ Container docker-mobsf-1      Started
#  ✔ Container docker-nginx-1      Started
```

### Step 7: Verifikasi Semua Container Running

```bash
# Cek status
sudo docker compose -f docker/docker-compose.yml ps

# Output yang benar (semua STATUS: Up):
# NAME                  IMAGE           STATUS
# docker-postgres-1     postgres:14     Up 2 minutes
# docker-djangoq-1      mobsf:latest    Up 2 minutes
# docker-mobsf-1        mobsf:latest    Up 2 minutes
# docker-nginx-1        nginx:latest    Up 2 minutes
```

### Step 8: Lihat Logs

```bash
# Lihat logs semua services
sudo docker compose -f docker/docker-compose.yml logs

# Atau specific service
sudo docker compose -f docker/docker-compose.yml logs mobsf

# Follow logs real-time
sudo docker compose -f docker/docker-compose.yml logs -f
```

### Step 9: Akses MobSF

Buka browser dan akses: **http://localhost:8000**

Login dengan default credentials: `mobsf/mobsf`

---

## Method 3: Instalasi Manual

**Waktu setup: ~30-45 menit**

Instalasi langsung di sistem tanpa Docker. Cocok untuk development atau customization.

### Step 1: Install System Dependencies

```bash
# Update sistem
sudo apt update && sudo apt upgrade -y

# Install required packages
sudo apt install -y \
    android-tools-adb \
    build-essential \
    curl \
    git \
    libffi-dev \
    libssl-dev \
    libxml2-dev \
    libxslt1-dev \
    libjpeg-dev \
    python3.12 \
    python3.12-dev \
    python3-pip \
    sqlite3 \
    unzip \
    wget \
    zlib1g-dev
```

### Step 2: Install Java (OpenJDK 17+)

```bash
# Install OpenJDK 17
sudo apt install -y openjdk-17-jdk

# Verify installation
java -version
# Output: openjdk version "17.x.x"

# Set JAVA_HOME
echo 'export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64' >> ~/.bashrc
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

### Step 3: Install wkhtmltopdf (untuk PDF Reports)

```bash
# Download wkhtmltopdf
cd /tmp
wget https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-3/wkhtmltox_0.12.6.1-3.jammy_amd64.deb

# Install
sudo dpkg -i wkhtmltox_0.12.6.1-3.jammy_amd64.deb

# Install dependencies jika ada error
sudo apt-get install -f -y

# Verify
wkhtmltopdf --version
```

### Step 4: Clone MobSF Repository

```bash
# Clone dari GitHub
cd ~
git clone https://github.com/MobSF/Mobile-Security-Framework-MobSF.git
cd Mobile-Security-Framework-MobSF

# Atau jika sudah punya local copy
cd /home/haninayy/MovingForward/security-mobile
```

### Step 5: Install Poetry (Python Package Manager)

```bash
# Install Poetry
curl -sSL https://install.python-poetry.org | python3 -

# Add Poetry to PATH
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# Verify installation
poetry --version
# Output: Poetry (version 1.8.4)
```

### Step 6: Install Python Dependencies

```bash
# Install dependencies menggunakan Poetry
poetry config virtualenvs.create false
poetry install --only main --no-root --no-interaction --no-ansi

# Proses ini akan memakan waktu 5-10 menit
# Output akan menampilkan progress installation packages
```

### Step 7: Setup Database

```bash
# Setup environment variables untuk superuser
export DJANGO_SUPERUSER_USERNAME=mobsf
export DJANGO_SUPERUSER_PASSWORD=mobsf

# Run database migrations
poetry run python manage.py makemigrations
poetry run python manage.py makemigrations StaticAnalyzer
poetry run python manage.py migrate

# Create superuser
poetry run python manage.py createsuperuser --noinput --email ""

# Create roles (jika ada)
poetry run python manage.py create_roles
```

### Step 8: Run Setup Script

```bash
# Jalankan setup script untuk additional configuration
chmod +x setup.sh
./setup.sh

# Script akan:
# - Verify Python version
# - Install/upgrade pip
# - Install dependencies
# - Setup database
# - Create superuser
```

### Step 9: Start MobSF Server

```bash
# Run MobSF
chmod +x run.sh
./run.sh

# Atau manual:
poetry run python manage.py runserver 0.0.0.0:8000

# Output yang diharapkan:
# Performing system checks...
# System check identified no issues (0 silenced).
# October 23, 2025 - 10:00:00
# Django version 4.x.x, using settings 'mobsf.MobSF.settings'
# Starting development server at http://0.0.0.0:8000/
# Quit the server with CONTROL-C.
```

### Step 10: Akses MobSF

Buka browser: **http://localhost:8000**

Login dengan:
- **Username:** `mobsf`
- **Password:** `mobsf` (atau password yang Anda set)

---

## ✅ Verifikasi Instalasi

### 1. Cek Status Container (Docker)

```bash
# Untuk Docker Run
sudo docker ps

# Untuk Docker Compose
sudo docker compose -f docker/docker-compose.yml ps

# Output yang benar: STATUS = Up X minutes
```

### 2. Cek Logs

```bash
# Docker Run
sudo docker logs mobsf

# Docker Compose
sudo docker compose -f docker/docker-compose.yml logs mobsf

# Manual Installation
# Lihat output di terminal saat run ./run.sh
```

### 3. Test Web Access

```bash
# Test dengan curl
curl http://localhost:8000

# Atau buka di browser
# http://localhost:8000
```

### 4. Test API Endpoint

```bash
# Get API version
curl http://localhost:8000/api/v1/version

# Output:
# {"version": "4.x.x"}
```

### 5. Upload Test APK

1. Login ke web interface
2. Klik "Upload & Analyze"
3. Upload APK file
4. Verify proses analisis berjalan

---

## 🔧 Troubleshooting

### Issue 1: docker-compose not found

**Error:**
```
Command 'docker-compose' not found
```

**Root Cause:** Mencoba menggunakan Docker Compose V1 (deprecated)

**Solution:**
```bash
# Gunakan Docker Compose V2 (tanpa hyphen)
docker compose version

# Jika belum ada, install Docker yang lebih baru
sudo apt install docker.io -y
```

---

### Issue 2: Permission Denied - Docker Socket

**Error:**
```
permission denied while trying to connect to Docker daemon socket at unix:///var/run/docker.sock
```

**Root Cause:** User belum masuk dalam `docker` group

**Solution:**
```bash
# Tambahkan user ke docker group
sudo usermod -aG docker $USER

# Aktifkan group tanpa logout
newgrp docker

# Atau logout & login lagi

# Verify
groups $USER | grep docker
```

---

### Issue 3: Container Keeps Restarting

**Error:**
```
STATUS: Restarting (XX) X seconds ago
```

**Root Cause:** Ada error di dalam container

**Solution:**
```bash
# Lihat logs untuk detail error
sudo docker logs mobsf

# Common issues:

# 1. Permission issue pada volume
sudo chown -R 9901:9901 $HOME/MobSF/mobsf_data

# 2. Port sudah dipakai
sudo lsof -i :8000
sudo kill -9 <PID>

# 3. Environment variable missing
# Gunakan official image, jangan build sendiri
```

---

### Issue 4: Port Already in Use

**Error:**
```
Bind for 0.0.0.0:8000 failed: port is already allocated
```

**Solution:**
```bash
# Cek process yang menggunakan port 8000
sudo lsof -i :8000

# Output:
# COMMAND  PID   USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
# python  1234  user   3u  IPv4  12345      0t0  TCP *:8000 (LISTEN)

# Kill process
sudo kill -9 1234

# Atau gunakan port lain
docker run -p 9000:8000 ... # Akses di http://localhost:9000
```

---

### Issue 5: No module named 'distutils'

**Error:**
```
ModuleNotFoundError: No module named 'distutils'
```

**Root Cause:** Python 3.12+ tidak include module `distutils`, dan `docker-compose` V1 masih menggunakannya

**Solution:**
```bash
# Gunakan Docker Compose V2 (recommended)
docker compose version

# Atau install distutils
sudo apt install python3-distutils

# ATAU gunakan official Docker image (paling mudah)
docker run ... opensecurity/mobile-security-framework-mobsf:latest
```

---

### Issue 6: Out of Memory (OOM)

**Error:**
```
Container 'mobsf' killed (exit code 137)
```

**Root Cause:** Container kehabisan memory

**Solution:**
```bash
# Allocate lebih banyak memory
docker run --memory="4g" --memory-swap="4g" ...

# Atau tambah swap di sistem
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Verify
free -h
```

---

### Issue 7: Build Failed (Docker Compose)

**Error:**
```
ERROR: failed to solve: process "/bin/sh -c ..." did not complete successfully
```

**Solution:**
```bash
# Clean Docker cache
docker system prune -a

# Build dengan no-cache
docker compose -f docker/docker-compose.yml build --no-cache

# Cek disk space
docker system df
df -h

# Jika masih error, gunakan official image
# Edit docker-compose.yml:
# image: opensecurity/mobile-security-framework-mobsf:latest
# (comment out 'build' section)
```

---

### Issue 8: Database Connection Error

**Error:**
```
django.db.utils.OperationalError: could not connect to server: Connection refused
```

**Solution:**
```bash
# Cek PostgreSQL container running
docker compose -f docker/docker-compose.yml ps postgres

# Restart PostgreSQL
docker compose -f docker/docker-compose.yml restart postgres

# Cek logs
docker compose -f docker/docker-compose.yml logs postgres

# Test connection
docker exec -it docker-postgres-1 psql -U mobsf -d mobsf
```

---

## 📚 Command Reference

### Docker Commands

#### Container Management

```bash
# Start MobSF
docker run -it --rm -d -p 8000:8000 --name mobsf \
  opensecurity/mobile-security-framework-mobsf:latest

# Stop container
docker stop mobsf

# Start stopped container
docker start mobsf

# Restart container
docker restart mobsf

# Remove container
docker rm mobsf
docker rm -f mobsf  # Force remove (even if running)

# List running containers
docker ps

# List all containers (including stopped)
docker ps -a
```

#### Logs & Debugging

```bash
# View logs
docker logs mobsf

# Follow logs (real-time)
docker logs -f mobsf

# Last 100 lines
docker logs --tail 100 mobsf

# Logs since specific time
docker logs --since 10m mobsf

# Access container shell
docker exec -it mobsf bash
docker exec -it mobsf sh

# Run command in container
docker exec mobsf ls -la /home/mobsf/.MobSF

# Inspect container details
docker inspect mobsf

# View resource usage
docker stats mobsf
```

#### Image Management

```bash
# Pull latest image
docker pull opensecurity/mobile-security-framework-mobsf:latest

# List images
docker images

# Remove image
docker rmi opensecurity/mobile-security-framework-mobsf:latest

# Remove unused images
docker image prune
docker image prune -a  # Remove all unused
```

#### Volume Management

```bash
# Create volume
docker volume create mobsf_data

# List volumes
docker volume ls

# Inspect volume
docker volume inspect mobsf_data

# Remove volume
docker volume rm mobsf_data

# Remove all unused volumes
docker volume prune
```

#### System Cleanup

```bash
# Remove stopped containers
docker container prune

# Remove unused images
docker image prune -a

# Remove unused volumes
docker volume prune

# Remove everything unused
docker system prune -a --volumes

# Check disk usage
docker system df
```

---

### Docker Compose Commands

```bash
# Start all services (detached mode)
docker compose -f docker/docker-compose.yml up -d

# Start with logs visible
docker compose -f docker/docker-compose.yml up

# Stop services
docker compose -f docker/docker-compose.yml stop

# Stop and remove containers
docker compose -f docker/docker-compose.yml down

# Stop, remove, and delete volumes
docker compose -f docker/docker-compose.yml down -v

# Restart all services
docker compose -f docker/docker-compose.yml restart

# Restart specific service
docker compose -f docker/docker-compose.yml restart mobsf

# Build images
docker compose -f docker/docker-compose.yml build

# Build with no cache
docker compose -f docker/docker-compose.yml build --no-cache

# Build and start
docker compose -f docker/docker-compose.yml up -d --build

# View logs
docker compose -f docker/docker-compose.yml logs

# Follow logs
docker compose -f docker/docker-compose.yml logs -f

# Specific service logs
docker compose -f docker/docker-compose.yml logs mobsf
docker compose -f docker/docker-compose.yml logs postgres

# Check service status
docker compose -f docker/docker-compose.yml ps

# View configuration
docker compose -f docker/docker-compose.yml config

# Execute command in service
docker compose -f docker/docker-compose.yml exec mobsf bash
```

---

### Manual Installation Commands

```bash
# Setup
./setup.sh

# Run MobSF
./run.sh

# Run with specific host:port
poetry run python manage.py runserver 0.0.0.0:9000

# Database operations
poetry run python manage.py makemigrations
poetry run python manage.py migrate
poetry run python manage.py createsuperuser

# Django shell
poetry run python manage.py shell

# Database shell
poetry run python manage.py dbshell

# Collect static files
poetry run python manage.py collectstatic --noinput

# Run tests
poetry run python manage.py test

# Create database backup
poetry run python manage.py dumpdata > backup.json

# Restore database
poetry run python manage.py loaddata backup.json
```

---

## 🔒 Security Best Practices

### 1. Ganti Default Credentials

```bash
# Login pertama kali, langsung ganti password
# Web Interface: Settings > Change Password

# Atau via command line (Docker)
docker exec -it mobsf python manage.py changepassword mobsf

# Manual installation
poetry run python manage.py changepassword mobsf
```

### 2. Gunakan HTTPS di Production

```bash
# Setup Nginx dengan SSL
sudo apt install nginx certbot python3-certbot-nginx

# Get SSL certificate
sudo certbot --nginx -d mobsf.yourdomain.com

# Nginx akan auto-configure HTTPS
```

### 3. Restrict Network Access

```bash
# Bind hanya ke localhost (tidak exposed ke internet)
docker run -p 127.0.0.1:8000:8000 ...

# Atau gunakan firewall
sudo ufw allow from 192.168.1.0/24 to any port 8000
sudo ufw enable
```

### 4. Regular Backups

```bash
# Backup Docker volume
docker run --rm \
  -v mobsf_data:/data \
  -v $(pwd):/backup \
  ubuntu tar czf /backup/mobsf_backup_$(date +%Y%m%d).tar.gz /data

# Restore
docker run --rm \
  -v mobsf_data:/data \
  -v $(pwd):/backup \
  ubuntu tar xzf /backup/mobsf_backup.tar.gz -C /
```

### 5. Update Regularly

```bash
# Update Docker image
docker pull opensecurity/mobile-security-framework-mobsf:latest

# Recreate container dengan image baru
docker stop mobsf
docker rm mobsf
docker run -it -d -p 8000:8000 -v mobsf_data:/home/mobsf/.MobSF \
  --name mobsf opensecurity/mobile-security-framework-mobsf:latest
```

### 6. Secure API Keys

```bash
# Generate secure API key
openssl rand -hex 32

# Set via environment variable (jangan hardcode)
export MOBSF_API_KEY=$(openssl rand -hex 32)

# Never commit to git
echo "*.env" >> .gitignore
echo "MOBSF_API_KEY=*" >> .gitignore
```

---

## 📞 Support & Resources

- **Official Documentation:** https://mobsf.github.io/docs/
- **GitHub Repository:** https://github.com/MobSF/Mobile-Security-Framework-MobSF
- **GitHub Issues:** https://github.com/MobSF/Mobile-Security-Framework-MobSF/issues
- **Docker Hub:** https://hub.docker.com/r/opensecurity/mobile-security-framework-mobsf
- **Slack Community:** https://mobsf.slack.com/

---

## 📄 License

GNU General Public License v3.0 - see [LICENSE](LICENSE)

---

**Created:** October 23, 2025  
**Author:** Setup Documentation Team  
**Last Updated:** October 23, 2025
