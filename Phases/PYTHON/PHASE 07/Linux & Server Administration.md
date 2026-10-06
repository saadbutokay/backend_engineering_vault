
```
Every backend you build eventually runs on a Linux server.

Not your laptop. A server.
Somewhere in a data center (AWS, Google Cloud, Azure, DigitalOcean).
Running Linux.
No GUI. Just a terminal.

You connect via SSH, type commands, and that's it.

Why Linux?
  ✅ Free and open source
  ✅ Runs 90%+ of web servers
  ✅ Stable, secure, powerful
  ✅ AWS, Google Cloud, Azure all use Linux
  ✅ Docker runs on Linux
  ✅ Your production server IS Linux

This phase teaches you:
  - Navigate and manage a Linux system
  - Manage processes (start, stop, monitor)
  - Configure Nginx (reverse proxy)
  - Manage services with systemd
  - SSH access and security
  - Monitor server health
  - Everything a backend engineer does daily
```

---

## Setup — Get a Linux Environment

Bash

```
# Option 1: WSL2 on Windows (Windows Subsystem for Linux)
# In PowerShell (admin):
wsl --install
wsl --install -d Ubuntu

# Option 2: Mac (already Unix-based, most commands work)
# Install Homebrew if not done:
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Option 3: Ubuntu VM or container
docker run -it ubuntu:22.04 bash

# Option 4: Real cloud server (best for practice)
# Create a free tier EC2 instance on AWS or
# $5/month droplet on DigitalOcean

# Verify you have bash:
bash --version
uname -a    # shows OS info
```

---

## 1. 🗂️ File System Navigation

### Linux File System Structure

text

```
/                       ← root of everything
├── bin/                ← essential user commands (ls, cp, mv...)
├── sbin/               ← system admin commands (fdisk, ifconfig...)
├── etc/                ← configuration files
│   ├── nginx/          ← Nginx config
│   ├── systemd/        ← systemd service configs
│   └── ssh/            ← SSH configuration
├── home/               ← user home directories
│   └── ubuntu/         ← your home directory
├── var/                ← variable data (logs, databases)
│   ├── log/            ← system and app logs
│   └── www/            ← web files (sometimes)
├── usr/                ← user utilities and applications
│   ├── bin/            ← user binaries
│   ├── local/          ← locally installed software
│   └── lib/            ← libraries
├── tmp/                ← temporary files (cleared on reboot)
├── opt/                ← optional software packages
├── proc/               ← virtual filesystem for process info
├── sys/                ← virtual filesystem for kernel info
└── root/               ← root user's home directory
```

### Essential Navigation Commands

Bash

```
# ─────────────────────────────────────────
# WHERE AM I?
# ─────────────────────────────────────────
pwd
# /home/ubuntu/projects

# ─────────────────────────────────────────
# LISTING FILES
# ─────────────────────────────────────────
ls                      # basic list
ls -l                   # long format (permissions, size, date)
ls -la                  # include hidden files (starting with .)
ls -lh                  # human-readable sizes (KB, MB, GB)
ls -lt                  # sort by modification time (newest first)
ls -lS                  # sort by file size (largest first)
ls -lR                  # recursive (show all subdirectories)
ls /var/log/            # list specific directory

# Long format explained:
# -rw-r--r-- 1 ubuntu ubuntu 4096 Jan 15 14:32 main.py
#  │││││││││ │ │      │      │    │              └─ filename
#  │││││││││ │ │      │      │    └─ modification date
#  │││││││││ │ │      │      └─ size in bytes
#  │││││││││ │ │      └─ group owner
#  │││││││││ │ └─ user owner
#  │││││││││ └─ number of hard links
#  ││││││││└─ other permissions (r=read, w=write, x=execute)
#  │││││└───── group permissions
#  ││└───────── user permissions
#  └─────────── file type (- = file, d = directory, l = symlink)

# ─────────────────────────────────────────
# MOVING AROUND
# ─────────────────────────────────────────
cd /var/log             # go to absolute path
cd projects             # go to relative path
cd ..                   # up one level
cd ../..                # up two levels
cd ~                    # go home
cd -                    # go to previous directory
cd /                    # go to root

# ─────────────────────────────────────────
# FIND FILES
# ─────────────────────────────────────────
find / -name "nginx.conf"           # find file by name (from root)
find /var/log -name "*.log"         # find all .log files
find /home -user ubuntu             # files owned by ubuntu
find /tmp -mtime +7                 # files older than 7 days
find . -size +100M                  # files larger than 100MB
find . -type f -name "*.py"         # only files (not dirs)
find . -type d -name "venv"         # only directories named venv

# locate (faster but uses a database)
locate nginx.conf
sudo updatedb                       # update locate database

# which — where is a command?
which python3           # /usr/bin/python3
which nginx             # /usr/sbin/nginx
which pip               # shows which pip is active

# whereis — all locations
whereis python3
```

---

## 2. 📁 File Operations

Bash

```
# ─────────────────────────────────────────
# CREATE
# ─────────────────────────────────────────
touch myfile.txt                # create empty file
touch -t 202401150000 file.txt  # create with specific timestamp
mkdir mydir                     # create directory
mkdir -p /opt/myapp/logs        # create with parents
mkdir -m 755 mydir              # create with permissions

# ─────────────────────────────────────────
# READ/VIEW
# ─────────────────────────────────────────
cat file.txt                    # print entire file
cat file1.txt file2.txt         # concatenate files
less file.txt                   # view with pagination (q to quit)
                                # j/k = down/up, /pattern = search
more file.txt                   # older viewer
head file.txt                   # first 10 lines
head -n 20 file.txt             # first 20 lines
tail file.txt                   # last 10 lines
tail -n 50 file.txt             # last 50 lines
tail -f /var/log/syslog         # FOLLOW log file live (Ctrl+C to stop)
tail -f -n 100 app.log          # last 100 lines then follow

# ─────────────────────────────────────────
# COPY
# ─────────────────────────────────────────
cp file.txt backup.txt          # copy file
cp -r src/ dest/                # copy directory recursively
cp -p file.txt backup.txt       # preserve permissions and timestamps
cp -i file.txt dest.txt         # interactive (ask before overwrite)

# ─────────────────────────────────────────
# MOVE / RENAME
# ─────────────────────────────────────────
mv old.txt new.txt              # rename
mv file.txt /tmp/               # move to directory
mv dir1/ dir2/                  # rename directory
mv *.log /var/log/archive/      # move multiple files

# ─────────────────────────────────────────
# DELETE
# ─────────────────────────────────────────
rm file.txt                     # delete file
rm -i file.txt                  # interactive (confirm each)
rm -f file.txt                  # force (no prompt)
rm -r mydir/                    # delete directory recursively
rm -rf mydir/                   # force delete directory (CAREFUL!)

# ⚠️  CRITICAL SAFETY:
# rm -rf / = DELETES EVERYTHING
# rm -rf /* = DELETES EVERYTHING
# NEVER run these. Ever.

# ─────────────────────────────────────────
# VIEW / EDIT
# ─────────────────────────────────────────
nano file.txt                   # simple editor (beginners)
                                # Ctrl+O = save, Ctrl+X = exit

vim file.txt                    # powerful editor
                                # i = insert mode, Esc = command mode
                                # :w = save, :q = quit, :wq = save+quit
                                # :q! = quit without saving
                                # dd = delete line, yy = copy line, p = paste

# ─────────────────────────────────────────
# LINKS
# ─────────────────────────────────────────
ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/myapp
# Create symlink (like a shortcut)
# When you modify the original, the link reflects changes

ls -la /etc/nginx/sites-enabled/
# lrwxrwxrwx myapp -> /etc/nginx/sites-available/myapp
# l = symlink

# ─────────────────────────────────────────
# DISK USAGE
# ─────────────────────────────────────────
df -h                           # disk space (human readable)
# Filesystem      Size  Used Avail Use% Mounted on
# /dev/sda1        40G   12G   26G  32% /

du -sh /var/log/                # size of directory
du -sh *                        # size of each item in current dir
du -sh * | sort -rh | head -10  # top 10 largest items

# ─────────────────────────────────────────
# SEARCHING IN FILES
# ─────────────────────────────────────────
grep "error" /var/log/syslog    # find lines containing "error"
grep -i "error" /var/log/syslog # case insensitive
grep -n "error" /var/log/syslog # show line numbers
grep -r "TODO" ./src/           # recursive search in directory
grep -l "error" /var/log/*.log  # list files containing pattern
grep -v "DEBUG" app.log         # exclude lines with DEBUG
grep -c "error" app.log         # count matching lines
grep -A 3 "ERROR" app.log       # 3 lines after match
grep -B 3 "ERROR" app.log       # 3 lines before match
grep -E "error|warning" app.log # regex (or pattern)

# pipelines with grep
cat access.log | grep "POST" | grep "500"    # chain greps
ps aux | grep nginx                           # find nginx processes
history | grep "docker"                       # find docker commands in history
```

---

## 3. 🔐 Permissions

Bash

```
# ─────────────────────────────────────────
# UNDERSTANDING PERMISSIONS
# ─────────────────────────────────────────
# -rwxr-xr--
#  │││││││││
#  ││││││││└─ other: read only
#  │││││││└── other: no write
#  ││││││└─── other: no execute
#  │││││└──── group: read
#  ││││└───── group: no write
#  │││└────── group: execute
#  ││└─────── owner: read
#  │└──────── owner: write
#  └───────── owner: execute

# r = 4, w = 2, x = 1
# 7 = rwx (4+2+1)
# 6 = rw- (4+2+0)
# 5 = r-x (4+0+1)
# 4 = r-- (4+0+0)
# 0 = ---

# ─────────────────────────────────────────
# CHMOD — change permissions
# ─────────────────────────────────────────
chmod 755 script.sh             # rwxr-xr-x (owner=all, group/other=read+exec)
chmod 644 config.py             # rw-r--r-- (owner=read+write, others=read)
chmod 600 private_key           # rw------- (owner only)
chmod 777 file.txt              # rwxrwxrwx (everyone everything — AVOID!)
chmod +x script.sh              # add execute for everyone
chmod +x script.sh              # make executable
chmod -w file.txt               # remove write for everyone
chmod u+x file.txt              # add execute for user (owner)
chmod g-w file.txt              # remove write for group
chmod o-r file.txt              # remove read for others
chmod -R 755 /opt/myapp/        # recursive

# Common permission patterns:
# 755 = directories, executables
# 644 = regular files (config, code)
# 600 = private keys, secrets
# 700 = private directories
# 400 = read-only (backups)

# ─────────────────────────────────────────
# CHOWN — change ownership
# ─────────────────────────────────────────
chown ubuntu file.txt           # change owner to ubuntu
chown ubuntu:ubuntu file.txt    # change owner AND group
chown -R ubuntu:ubuntu /opt/myapp/  # recursive
sudo chown root:root /etc/nginx.conf  # owned by root

# ─────────────────────────────────────────
# SUDO — run as root
# ─────────────────────────────────────────
sudo command                    # run command as root
sudo -i                         # become root (stay as root)
sudo -u www-data command        # run as specific user
sudo !!                         # re-run last command with sudo

# Add user to sudoers (on Ubuntu):
sudo usermod -aG sudo username
```

---

## 4. 👤 User Management

Bash

```
# ─────────────────────────────────────────
# USERS
# ─────────────────────────────────────────
whoami                          # current user
id                              # current user ID, groups
id ubuntu                       # another user's info
w                               # who is logged in + what they're doing
last                            # login history

# Create user
sudo useradd -m -s /bin/bash -G sudo username
# -m = create home directory
# -s = shell
# -G = groups

# Set password
sudo passwd username

# Modify user
sudo usermod -aG docker ubuntu  # add ubuntu user to docker group
sudo usermod -s /bin/bash ubuntu # change shell
sudo usermod -L username        # lock account
sudo usermod -U username        # unlock account

# Delete user
sudo userdel -r username        # -r = remove home directory

# Switch user
su - username                   # switch to user
su -                            # switch to root

# ─────────────────────────────────────────
# GROUPS
# ─────────────────────────────────────────
groups                          # list current user's groups
groups ubuntu                   # list another user's groups
sudo groupadd devteam           # create group
sudo groupdel devteam           # delete group
cat /etc/group                  # all groups

# ─────────────────────────────────────────
# KEY FILES
# ─────────────────────────────────────────
cat /etc/passwd                 # all users (username, uid, home, shell)
cat /etc/shadow                 # password hashes (root only)
cat /etc/group                  # all groups
```

---

## 5. 📦 Package Management

Bash

```
# ─────────────────────────────────────────
# APT — Debian/Ubuntu package manager
# ─────────────────────────────────────────

# Always update before installing
sudo apt update                 # refresh package lists
sudo apt upgrade                # upgrade all installed packages
sudo apt dist-upgrade           # upgrade with dependency changes

# Install packages
sudo apt install nginx
sudo apt install python3.12 python3.12-venv python3-pip
sudo apt install postgresql postgresql-contrib
sudo apt install redis-server
sudo apt install git curl wget build-essential

# Install without confirmation prompt
sudo apt install -y nginx

# Search for packages
apt search nginx
apt show nginx                  # detailed package info

# Remove packages
sudo apt remove nginx           # remove package (keep config)
sudo apt purge nginx            # remove package AND config
sudo apt autoremove             # remove unused dependencies

# Clean cache
sudo apt clean                  # remove downloaded .deb files
sudo apt autoclean              # remove old downloaded .deb files

# List installed
apt list --installed
dpkg -l | grep nginx            # check if nginx is installed

# ─────────────────────────────────────────
# SNAP — newer package manager
# ─────────────────────────────────────────
sudo snap install code          # VS Code
sudo snap install docker        # Docker
sudo snap list                  # list snap packages
sudo snap remove code           # remove
```

---

## 6. ⚙️ Process Management

Bash

```
# ─────────────────────────────────────────
# VIEWING PROCESSES
# ─────────────────────────────────────────
ps                              # your processes in current session
ps aux                          # ALL processes, detailed
ps aux | grep nginx             # find nginx processes
ps -p 1234                      # show specific process
ps -f -u ubuntu                 # processes for user ubuntu

# Interactive process viewers
top                             # real-time process list
                                # q=quit, k=kill, h=help, 1=per-CPU
htop                            # better top (if installed)
                                # sudo apt install htop

# ─────────────────────────────────────────
# UNDERSTANDING ps aux OUTPUT
# ─────────────────────────────────────────
# USER    PID  %CPU %MEM    VSZ   RSS TTY   STAT  START  TIME  COMMAND
# root      1   0.0  0.1  16568  2048 ?     Ss   Jan15  0:04  /sbin/init
# ubuntu 1234   2.3  5.6 487292 56789 ?     Sl   10:32  0:45  python3 main.py
#
# PID  = Process ID (unique identifier)
# %CPU = CPU usage
# %MEM = Memory usage
# VSZ  = Virtual memory size
# RSS  = Resident set size (physical RAM used)
# STAT = Process state (R=running, S=sleeping, Z=zombie, T=stopped)
# TIME = Total CPU time used

# ─────────────────────────────────────────
# SIGNALS / KILLING PROCESSES
# ─────────────────────────────────────────
kill 1234                       # send SIGTERM (graceful shutdown)
kill -15 1234                   # same as above (15 = SIGTERM)
kill -9 1234                    # SIGKILL (force kill, no cleanup)
kill -SIGTERM 1234              # named signal
pkill nginx                     # kill by name
pkill -9 nginx                  # force kill by name
killall python3                 # kill all python3 processes

# ─────────────────────────────────────────
# BACKGROUND PROCESSES
# ─────────────────────────────────────────
python3 server.py &             # run in background
                                # [1] 1234 — job number and PID

jobs                            # list background jobs
fg                              # bring last job to foreground
fg %1                           # bring job #1 to foreground
bg                              # continue paused job in background
Ctrl+Z                          # pause current process (sends to background)
Ctrl+C                          # interrupt/kill current process

# ─────────────────────────────────────────
# NOHUP — run after logout
# ─────────────────────────────────────────
nohup python3 server.py &       # continues running after you log out
nohup python3 server.py > output.log 2>&1 &
# > output.log = redirect stdout
# 2>&1 = redirect stderr to stdout
# & = background

# ─────────────────────────────────────────
# PROCESS MONITORING
# ─────────────────────────────────────────

# Find process using a port
sudo lsof -i :8000              # what's using port 8000?
sudo ss -tulpn | grep :8000     # socket statistics
sudo netstat -tulpn | grep :8000 # (older alternative)

# Find process by port
sudo fuser 8000/tcp             # show PID using port 8000
sudo fuser -k 8000/tcp          # kill process using port 8000

# Memory info
free -h                         # RAM usage
cat /proc/meminfo               # detailed memory info

# CPU info
lscpu                           # CPU details
cat /proc/cpuinfo               # detailed CPU info
nproc                           # number of CPU cores

# Uptime and load
uptime
# 14:32:01 up 42 days, 3:15, 2 users, load average: 0.15, 0.22, 0.18
#                                                     1min 5min 15min
# Load average: number of processes waiting for CPU
# Should be <= number of CPU cores
```

---

## 7. 🌐 Nginx — The Reverse Proxy

### What is a Reverse Proxy?

text

```
Without Nginx:
  Browser → Port 8000 (FastAPI/uvicorn directly)
  
  Problems:
  ❌ Uvicorn shouldn't be exposed to internet
  ❌ No SSL/TLS termination
  ❌ One app per server
  ❌ No load balancing
  ❌ No static file serving

With Nginx:
  Browser → Port 80/443 (Nginx)
              ↓
          Nginx handles:
            - SSL/TLS (HTTPS)
            - Static files
            - Multiple apps on one server
            - Load balancing
            - Rate limiting
            - Compression
              ↓
          Port 8000 (your FastAPI/uvicorn)

Nginx is the gatekeeper. Your app is protected behind it.
```

### Installing and Configuring Nginx

Bash

```
# Install
sudo apt update
sudo apt install -y nginx

# Start and enable (auto-start on boot)
sudo systemctl start nginx
sudo systemctl enable nginx

# Verify it's running
sudo systemctl status nginx
curl http://localhost           # should see Nginx welcome page

# Nginx config structure:
# /etc/nginx/
#   nginx.conf              ← main config
#   sites-available/        ← all site configs (disabled)
#   sites-enabled/          ← active sites (symlinks to sites-available)
#   conf.d/                 ← additional config files
#   snippets/               ← reusable config snippets
```

### Nginx Configuration Files

nginx

```
# /etc/nginx/nginx.conf — main configuration

user www-data;                  # nginx runs as this user
worker_processes auto;          # one worker per CPU core

error_log /var/log/nginx/error.log warn;
pid /run/nginx.pid;

events {
    worker_connections 1024;    # max connections per worker
    multi_accept on;            # accept multiple connections at once
}

http {
    # ─── Basic settings ───────────────────────────────────────
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # ─── Logging ──────────────────────────────────────────────
    log_format main '$remote_addr - $remote_user [$time_local] '
                    '"$request" $status $body_bytes_sent '
                    '"$http_referer" "$http_user_agent"';

    access_log /var/log/nginx/access.log main;
    error_log /var/log/nginx/error.log;

    # ─── Performance ──────────────────────────────────────────
    sendfile on;                # efficient file transfer
    tcp_nopush on;              # optimize packet sending
    tcp_nodelay on;             # don't buffer data
    keepalive_timeout 65;       # keep connections alive 65s
    types_hash_max_size 2048;

    # ─── Gzip compression ─────────────────────────────────────
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;          # compression level (1-9)
    gzip_types text/plain text/css text/xml
               application/json application/javascript
               application/xml+rss application/atom+xml
               image/svg+xml;

    # ─── Security headers ─────────────────────────────────────
    server_tokens off;          # don't reveal nginx version

    # ─── Include site configs ─────────────────────────────────
    include /etc/nginx/conf.d/*.conf;
    include /etc/nginx/sites-enabled/*;
}
```

nginx

```
# /etc/nginx/sites-available/myapp
# Your FastAPI application

# ─── Upstream: your FastAPI app ───────────────────────────────
upstream fastapi_app {
    server 127.0.0.1:8000;          # single instance

    # Load balancing (multiple instances):
    # server 127.0.0.1:8000;
    # server 127.0.0.1:8001;
    # server 127.0.0.1:8002;

    # With weights:
    # server 127.0.0.1:8000 weight=3;
    # server 127.0.0.1:8001 weight=1;

    keepalive 32;                    # keep connections to upstream
}

# ─── Redirect HTTP to HTTPS ───────────────────────────────────
server {
    listen 80;
    listen [::]:80;
    server_name myapp.com www.myapp.com;

    # Redirect all HTTP to HTTPS
    return 301 https://$server_name$request_uri;
}

# ─── Main HTTPS server ────────────────────────────────────────
server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name myapp.com www.myapp.com;

    # ─── SSL Configuration ────────────────────────────────────
    ssl_certificate /etc/letsencrypt/live/myapp.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/myapp.com/privkey.pem;

    # Mozilla SSL config (modern)
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:...;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;

    # HSTS (tell browsers to always use HTTPS)
    add_header Strict-Transport-Security "max-age=63072000" always;

    # ─── Security Headers ─────────────────────────────────────
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;

    # ─── Logging ──────────────────────────────────────────────
    access_log /var/log/nginx/myapp_access.log;
    error_log /var/log/nginx/myapp_error.log;

    # ─── Client settings ──────────────────────────────────────
    client_max_body_size 20M;        # max upload size
    client_body_timeout 30s;
    client_header_timeout 30s;

    # ─── Rate limiting ────────────────────────────────────────
    # Define in http block:
    # limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;

    # Apply rate limit:
    # limit_req zone=api burst=20 nodelay;

    # ─── Static files ─────────────────────────────────────────
    location /static/ {
        alias /var/www/myapp/static/;
        expires 1y;                  # cache for 1 year
        add_header Cache-Control "public, immutable";
        gzip_static on;
    }

    location /media/ {
        alias /var/www/myapp/media/;
        expires 1d;
    }

    # ─── API proxy ────────────────────────────────────────────
    location / {
        proxy_pass http://fastapi_app;
        proxy_http_version 1.1;

        # Required headers
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # WebSocket support
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;

        # Buffering
        proxy_buffering on;
        proxy_buffer_size 4k;
        proxy_buffers 8 16k;
    }

    # ─── Health check (bypass rate limiting) ──────────────────
    location /health {
        proxy_pass http://fastapi_app;
        proxy_set_header Host $http_host;
        access_log off;              # don't log health checks
    }

    # ─── Block common attack patterns ─────────────────────────
    location ~ /\. {
        deny all;                    # block .htaccess, .git, etc.
    }

    location ~* \.(php|asp|aspx|cgi)$ {
        deny all;                    # block PHP/ASP files
    }
}
```

Bash

```
# ─────────────────────────────────────────
# ENABLING THE SITE
# ─────────────────────────────────────────

# Create symlink to enable
sudo ln -s /etc/nginx/sites-available/myapp \
           /etc/nginx/sites-enabled/myapp

# Test configuration (ALWAYS do this before reload!)
sudo nginx -t
# nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
# nginx: configuration file /etc/nginx/nginx.conf test is successful

# Reload nginx (no downtime)
sudo nginx -s reload
# OR:
sudo systemctl reload nginx

# Restart nginx (brief downtime)
sudo systemctl restart nginx

# View nginx logs
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log

# Common nginx commands
sudo nginx -T                   # dump full configuration
sudo nginx -v                   # nginx version
sudo nginx -V                   # version + build options
```

### Nginx Rate Limiting

nginx

```
# /etc/nginx/nginx.conf — add to http block

# Define rate limit zones
# zone=api: zone name
# 10m: 10MB memory for storing client IPs
# rate=10r/s: 10 requests per second per IP
limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
limit_req_zone $binary_remote_addr zone=login:10m rate=5r/m;  # 5/minute

# In server block:
location /api/ {
    limit_req zone=api burst=20 nodelay;
    # burst=20: allow bursts of 20 requests
    # nodelay: don't queue, reject immediately if over burst

    proxy_pass http://fastapi_app;
}

location /auth/login {
    limit_req zone=login burst=3 nodelay;
    proxy_pass http://fastapi_app;
}
```

---

## 8. 🔧 systemd — Service Management

### What is systemd?

text

```
systemd = service manager for Linux.

Without systemd:
  You start your app manually.
  Server reboots → app is gone.
  App crashes → nobody restarts it.

With systemd:
  ✅ App starts automatically on boot
  ✅ App restarts automatically on crash
  ✅ Logs go to journald (centralized logging)
  ✅ Manages dependencies between services
  ✅ Proper start/stop/status commands
```

### Creating a systemd Service

ini

```
# /etc/systemd/system/myapp.service
# Your FastAPI application as a systemd service

[Unit]
Description=My FastAPI Application
Documentation=https://github.com/myorg/myapp
# Start after these services are running:
After=network.target postgresql.service redis.service
# We REQUIRE these (fail if not running):
Requires=postgresql.service

[Service]
# ─── Process settings ─────────────────────────────────────────
Type=exec                       # exec: direct process, not forked
User=ubuntu                     # run as this user (NOT root!)
Group=ubuntu
WorkingDirectory=/home/ubuntu/myapp

# ─── Environment ──────────────────────────────────────────────
# Option 1: Environment variables directly
Environment="APP_ENV=production"
Environment="PORT=8000"

# Option 2: Environment file (better for secrets)
EnvironmentFile=/home/ubuntu/myapp/.env

# ─── The command to run ───────────────────────────────────────
ExecStart=/home/ubuntu/myapp/venv/bin/uvicorn \
    app.main:app \
    --host 0.0.0.0 \
    --port 8000 \
    --workers 4 \
    --log-level info \
    --access-log

# Pre-start: run migrations (optional)
ExecStartPre=/home/ubuntu/myapp/venv/bin/alembic upgrade head

# Graceful reload (no downtime)
ExecReload=/bin/kill -HUP $MAINPID

# ─── Restart policy ───────────────────────────────────────────
Restart=always                  # always restart on failure
RestartSec=5                    # wait 5 seconds before restart
StartLimitInterval=60           # in 60 seconds...
StartLimitBurst=5               # ...allow max 5 restart attempts

# ─── Security hardening ───────────────────────────────────────
NoNewPrivileges=yes             # can't gain new privileges
ProtectSystem=strict            # read-only filesystem (except allowed)
ProtectHome=read-only           # home is read-only
ReadWritePaths=/home/ubuntu/myapp/logs
ReadWritePaths=/tmp

# ─── Resource limits ──────────────────────────────────────────
LimitNOFILE=65536               # max open files (important for high traffic!)
LimitNPROC=4096                 # max processes
MemoryLimit=1G                  # max memory (OOM killer if exceeded)

# ─── Output ───────────────────────────────────────────────────
StandardOutput=journal          # logs go to journald
StandardError=journal
SyslogIdentifier=myapp          # identifies logs from this service

[Install]
WantedBy=multi-user.target      # start in normal multi-user mode
```

Bash

```
# ─────────────────────────────────────────
# MANAGING YOUR SERVICE
# ─────────────────────────────────────────

# After creating/modifying the service file:
sudo systemctl daemon-reload    # reload systemd config

# Enable service (start on boot)
sudo systemctl enable myapp

# Start service
sudo systemctl start myapp

# Check status
sudo systemctl status myapp

# Output:
# ● myapp.service - My FastAPI Application
#      Loaded: loaded (/etc/systemd/system/myapp.service; enabled)
#      Active: active (running) since Mon 2024-01-15 14:32:01 UTC; 1h ago
#    Main PID: 1234 (uvicorn)
#     CGroup: /system.slice/myapp.service
#             ├─1234 uvicorn app.main:app --host 0.0.0.0 ...
#             ├─1235 uvicorn worker
#             └─1236 uvicorn worker

# Stop service
sudo systemctl stop myapp

# Restart service
sudo systemctl restart myapp

# Reload (graceful — sends SIGHUP)
sudo systemctl reload myapp

# Disable (don't start on boot)
sudo systemctl disable myapp

# View logs
sudo journalctl -u myapp                    # all logs
sudo journalctl -u myapp -f                 # follow live
sudo journalctl -u myapp --since "1 hour ago"
sudo journalctl -u myapp --since "2024-01-15" --until "2024-01-16"
sudo journalctl -u myapp -n 100            # last 100 lines
sudo journalctl -u myapp -p err            # only errors
sudo journalctl -u myapp --no-pager       # no pagination

# List all services
systemctl list-units --type=service
systemctl list-units --type=service --state=running
systemctl list-units --type=service --state=failed

# ─────────────────────────────────────────
# MULTIPLE WORKERS WITH GUNICORN
# ─────────────────────────────────────────

# If using gunicorn instead of uvicorn directly:
# ExecStart=/home/ubuntu/myapp/venv/bin/gunicorn \
#     --bind 0.0.0.0:8000 \
#     --workers 4 \
#     --worker-class uvicorn.workers.UvicornWorker \
#     --timeout 120 \
#     --access-logfile /var/log/myapp/access.log \
#     --error-logfile /var/log/myapp/error.log \
#     app.main:app
```

---

## 9. 🔑 SSH — Secure Remote Access

Bash

```
# ─────────────────────────────────────────
# GENERATING SSH KEYS
# ─────────────────────────────────────────
# On your LOCAL machine:
ssh-keygen -t ed25519 -C "your-email@example.com"
# Enter file location: ~/.ssh/id_ed25519 (press Enter for default)
# Enter passphrase: (recommended — adds security layer)

# This creates TWO files:
# ~/.ssh/id_ed25519      ← PRIVATE KEY (never share this!)
# ~/.ssh/id_ed25519.pub  ← PUBLIC KEY (safe to share)

# ─────────────────────────────────────────
# CONNECTING TO A SERVER
# ─────────────────────────────────────────
ssh ubuntu@203.0.113.1          # connect with default key
ssh -i ~/.ssh/my_key ubuntu@server.com   # specific key
ssh -p 2222 ubuntu@server.com   # non-default port

# ─────────────────────────────────────────
# SSH CONFIG FILE — shortcuts
# ─────────────────────────────────────────
# ~/.ssh/config
Host myserver
    HostName 203.0.113.1        # server IP or hostname
    User ubuntu                 # username
    IdentityFile ~/.ssh/id_ed25519  # which key to use
    Port 22                     # SSH port
    ServerAliveInterval 60      # keep connection alive

Host staging
    HostName staging.myapp.com
    User ubuntu
    IdentityFile ~/.ssh/staging_key

# Now you can just:
ssh myserver                    # instead of: ssh -i key ubuntu@IP
ssh staging

# ─────────────────────────────────────────
# COPYING SSH KEY TO SERVER
# ─────────────────────────────────────────
# From your local machine:
ssh-copy-id -i ~/.ssh/id_ed25519.pub ubuntu@203.0.113.1
# OR manually:
cat ~/.ssh/id_ed25519.pub | ssh ubuntu@server.com \
    "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"

# ─────────────────────────────────────────
# SERVER SSH CONFIGURATION
# ─────────────────────────────────────────
# /etc/ssh/sshd_config — server-side SSH config

# Harden SSH (important for production!):
sudo nano /etc/ssh/sshd_config

# Key settings:
Port 2222                       # change from default 22 (security through obscurity)
PasswordAuthentication no       # DISABLE password login! Keys only.
PermitRootLogin no              # never allow root SSH login
MaxAuthTries 3                  # 3 failed attempts = disconnect
PubkeyAuthentication yes        # enable key-based auth
AuthorizedKeysFile .ssh/authorized_keys

# After changes:
sudo sshd -t                    # test config
sudo systemctl restart sshd     # apply changes

# ─────────────────────────────────────────
# SSH TUNNELING — advanced
# ─────────────────────────────────────────

# Forward local port to remote (access DB through SSH)
ssh -L 5433:localhost:5432 ubuntu@server.com
# Now: psql -h localhost -p 5433  ← connects to remote PostgreSQL!

# Forward remote port to local
ssh -R 8080:localhost:8000 ubuntu@server.com
# Now: requests to server:8080 reach your local:8000

# SOCKS proxy through SSH
ssh -D 1080 ubuntu@server.com
# Now: configure browser to use localhost:1080 as SOCKS5 proxy

# ─────────────────────────────────────────
# SCP — copy files over SSH
# ─────────────────────────────────────────
scp file.txt ubuntu@server.com:/home/ubuntu/  # local → remote
scp ubuntu@server.com:/var/log/app.log ./     # remote → local
scp -r ./myapp ubuntu@server.com:/home/ubuntu/  # directory
scp -i ~/.ssh/mykey file.txt ubuntu@server.com:/tmp/  # specific key

# RSYNC — better than scp for sync
rsync -avz ./myapp ubuntu@server.com:/home/ubuntu/
rsync -avz --delete ./myapp ubuntu@server.com:/home/ubuntu/
# -a = archive mode
# -v = verbose
# -z = compress
# --delete = delete files not in source
```

---

## 10. 🌐 Firewall Configuration

Bash

```
# ─────────────────────────────────────────
# UFW — Uncomplicated Firewall
# ─────────────────────────────────────────
# UFW is the beginner-friendly frontend to iptables

# Check status
sudo ufw status
sudo ufw status verbose

# Enable UFW
sudo ufw enable

# ─────────────────────────────────────────
# RULES — what to allow
# ─────────────────────────────────────────

# Allow SSH (CRITICAL — do this BEFORE enabling UFW)
sudo ufw allow ssh              # = allow 22/tcp
sudo ufw allow 2222/tcp         # custom SSH port

# Allow HTTP and HTTPS
sudo ufw allow http             # = allow 80/tcp
sudo ufw allow https            # = allow 443/tcp
sudo ufw allow 'Nginx Full'     # allow both 80 and 443 for Nginx

# Allow specific ports
sudo ufw allow 8000/tcp         # development server

# Allow from specific IP
sudo ufw allow from 203.0.113.1 to any port 22    # only this IP can SSH
sudo ufw allow from 10.0.0.0/8                    # entire subnet

# Deny rules
sudo ufw deny 8000              # block port 8000 from internet
sudo ufw deny from 192.168.1.1  # block specific IP

# Delete rules
sudo ufw delete allow 8000
sudo ufw delete allow http
sudo ufw status numbered         # show numbered rules
sudo ufw delete 3               # delete rule number 3

# Reset to defaults
sudo ufw reset

# ─────────────────────────────────────────
# PRODUCTION FIREWALL SETUP
# ─────────────────────────────────────────
sudo ufw default deny incoming   # deny all incoming by default
sudo ufw default allow outgoing  # allow all outgoing
sudo ufw allow 2222/tcp         # SSH (custom port)
sudo ufw allow 80/tcp           # HTTP
sudo ufw allow 443/tcp          # HTTPS
# Only allow direct DB access from app server, not internet:
# sudo ufw allow from app_server_ip to any port 5432
sudo ufw enable
sudo ufw status verbose
```

---

## 11. 📊 System Monitoring

Bash

```
# ─────────────────────────────────────────
# REAL-TIME MONITORING
# ─────────────────────────────────────────

top                             # real-time system stats
htop                            # better top (color, mouse support)
vmstat 1                        # virtual memory stats every 1 second
iostat 1                        # I/O statistics
iotop                           # disk I/O per process
nethogs                         # network usage per process
iftop                           # network bandwidth

# ─────────────────────────────────────────
# MEMORY
# ─────────────────────────────────────────
free -h
# Output:
#               total        used        free      shared  buff/cache   available
# Mem:           7.7G        2.1G        1.2G        256M        4.4G        5.1G
# Swap:          2.0G        0.0G        2.0G
#
# Mem: total = 7.7GB
# used = 2.1GB actively used
# buff/cache = 4.4GB used for disk cache (can be freed)
# available = 5.1GB actually available for new processes
# Swap: virtual memory on disk (bad if heavily used = system too slow)

# Check which processes use most memory
ps aux --sort=-%mem | head -10

# ─────────────────────────────────────────
# CPU
# ─────────────────────────────────────────
uptime
# 14:32:01 up 42 days, load average: 0.15, 0.22, 0.18
# load average: 1min, 5min, 15min
# Rule: load average should be <= number of CPU cores
# More cores = can handle higher load averages

mpstat 1 5                      # CPU stats for 5 seconds
mpstat -P ALL 1                 # per-core CPU stats

# ─────────────────────────────────────────
# DISK I/O
# ─────────────────────────────────────────
df -h                           # disk space
iostat -x 1                     # extended I/O stats
iotop                           # real-time I/O per process

# Check if disk is bottleneck
iostat -dx 1
# %util = disk utilization (if > 80%, disk is bottleneck)

# ─────────────────────────────────────────
# NETWORK
# ─────────────────────────────────────────
ss -tulpn                       # listening sockets
ss -tulpn | grep :80           # who's using port 80
netstat -tulpn                  # older alternative
iftop                           # real-time bandwidth per connection
ip addr                         # network interfaces and IPs
ip route                        # routing table
curl https://api.myapp.com/health  # quick API health check

# ─────────────────────────────────────────
# LOGS
# ─────────────────────────────────────────
# System logs
sudo journalctl -f              # follow all system logs
sudo journalctl -p err          # only errors
sudo journalctl --since today   # today's logs
sudo journalctl -u nginx        # nginx service logs
sudo journalctl -u myapp -f     # follow your app

# Log files
sudo tail -f /var/log/syslog
sudo tail -f /var/log/auth.log      # SSH/sudo attempts
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log

# ─────────────────────────────────────────
# CRON JOBS — scheduled tasks
# ─────────────────────────────────────────
crontab -e                      # edit current user's cron jobs
sudo crontab -u ubuntu -e       # edit specific user's cron

# Cron format:
# minute hour day-of-month month day-of-week command
# *      *    *             *     *            command (every minute)

# Examples:
# Run every day at 2am:
# 0 2 * * * /home/ubuntu/myapp/scripts/backup.sh

# Run every hour:
# 0 * * * * /home/ubuntu/myapp/scripts/cleanup.sh

# Run every 5 minutes:
# */5 * * * * /home/ubuntu/myapp/scripts/healthcheck.sh

# Run Monday-Friday at 9am:
# 0 9 * * 1-5 /home/ubuntu/myapp/scripts/report.sh

# View cron jobs
crontab -l                      # current user's jobs
sudo cat /etc/crontab           # system-wide jobs
ls /etc/cron.d/                 # additional job files

# ─────────────────────────────────────────
# SERVER SECURITY MONITORING
# ─────────────────────────────────────────
# Watch for failed SSH attempts
sudo grep "Failed password" /var/log/auth.log | tail -20

# Watch for successful logins
sudo grep "Accepted" /var/log/auth.log | tail -20

# Who's logged in right now
who
w

# Last logins
last | head -20

# Check for suspicious processes
ps aux | grep -v "ubuntu\|root\|systemd\|nginx"
```

---

## 12. 🚀 Complete Deployment Script

Bash

```
#!/bin/bash
# deploy.sh — complete deployment script
# Run: ./deploy.sh

set -e              # exit on any error
set -u              # exit on undefined variable
set -o pipefail     # exit on pipe failure

# ─────────────────────────────────────────
# CONFIGURATION
# ─────────────────────────────────────────
APP_NAME="myapp"
APP_DIR="/home/ubuntu/myapp"
VENV_DIR="$APP_DIR/venv"
SERVICE_NAME="myapp"
NGINX_SITE="myapp"
REPO_URL="https://github.com/myorg/myapp.git"
BRANCH="main"

log() { echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1"; }
error() { echo "[ERROR] $1" >&2; exit 1; }

# ─────────────────────────────────────────
# PREFLIGHT CHECKS
# ─────────────────────────────────────────
log "Starting deployment of $APP_NAME..."

# Check we're on the right server
[[ $USER == "ubuntu" ]] || error "Must run as ubuntu user"

# Check required tools
command -v python3 >/dev/null || error "python3 not installed"
command -v nginx >/dev/null || error "nginx not installed"
command -v git >/dev/null || error "git not installed"

# ─────────────────────────────────────────
# UPDATE CODE
# ─────────────────────────────────────────
log "Pulling latest code..."

if [ -d "$APP_DIR" ]; then
    cd "$APP_DIR"
    git fetch origin
    git reset --hard "origin/$BRANCH"
else
    git clone "$REPO_URL" "$APP_DIR"
    cd "$APP_DIR"
fi

log "Current commit: $(git rev-parse --short HEAD)"

# ─────────────────────────────────────────
# SETUP VIRTUAL ENVIRONMENT
# ─────────────────────────────────────────
log "Setting up virtual environment..."

if [ ! -d "$VENV_DIR" ]; then
    python3 -m venv "$VENV_DIR"
fi

source "$VENV_DIR/bin/activate"
pip install --upgrade pip --quiet
pip install -r requirements.txt --quiet

# ─────────────────────────────────────────
# DATABASE MIGRATIONS
# ─────────────────────────────────────────
log "Running database migrations..."
alembic upgrade head

# ─────────────────────────────────────────
# COLLECT STATIC FILES (if Django)
# ─────────────────────────────────────────
# python manage.py collectstatic --noinput

# ─────────────────────────────────────────
# RELOAD APPLICATION
# ─────────────────────────────────────────
log "Reloading application..."
sudo systemctl reload "$SERVICE_NAME" 2>/dev/null || \
    sudo systemctl restart "$SERVICE_NAME"

# Wait for service to start
sleep 3

# ─────────────────────────────────────────
# VERIFY DEPLOYMENT
# ─────────────────────────────────────────
log "Verifying deployment..."

# Check service is running
if ! sudo systemctl is-active --quiet "$SERVICE_NAME"; then
    error "Service $SERVICE_NAME failed to start!"
fi

# Check health endpoint
HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" \
    http://localhost:8000/health)

if [ "$HTTP_CODE" != "200" ]; then
    error "Health check failed! HTTP $HTTP_CODE"
fi

log "✅ Deployment successful!"
log "Commit: $(git rev-parse --short HEAD)"
log "Time: $(date)"

# ─────────────────────────────────────────
# CLEANUP
# ─────────────────────────────────────────
# Remove old .pyc files
find . -name "*.pyc" -delete
find . -name "__pycache__" -type d -exec rm -rf {} + 2>/dev/null || true

log "Deployment complete!"
```

---

## 📊 Visual Summary

text

```
┌────────────────────────────────────────────────────────────────┐
│              LINUX & SERVER ADMINISTRATION                     │
│                                                                │
│  FILE SYSTEM:                                                  │
│  /etc = config  /var/log = logs  /opt = apps  /tmp = temp    │
│  ls -la | cd | find | grep | tail -f                         │
│  chmod 755 | chown user:group | sudo                         │
│                                                                │
│  PROCESSES:                                                    │
│  ps aux | top | htop | kill -9 PID | pkill nginx             │
│  lsof -i :8000 → what's using port 8000                      │
│  nohup cmd & → run after logout                               │
│                                                                │
│  NGINX:                                                        │
│  Reverse proxy → protects app, handles HTTPS, load balance   │
│  /etc/nginx/sites-available/ → create config                 │
│  ln -s available/myapp enabled/myapp → enable site           │
│  nginx -t → test config (always before reload!)              │
│  systemctl reload nginx → apply changes, no downtime         │
│                                                                │
│  SYSTEMD:                                                      │
│  /etc/systemd/system/myapp.service → service definition      │
│  systemctl enable/start/stop/restart/status myapp            │
│  journalctl -u myapp -f → follow service logs                │
│  Restart=always → auto-restart on crash                       │
│                                                                │
│  SSH:                                                          │
│  ssh-keygen -t ed25519 → generate key pair                   │
│  ~/.ssh/config → connection shortcuts                         │
│  PasswordAuthentication no → keys only (production!)         │
│  ssh -L 5433:localhost:5432 server → port forwarding         │
│                                                                │
│  MONITORING:                                                   │
│  free -h → RAM | df -h → disk | uptime → load               │
│  journalctl -f → live logs | tail -f /var/log/nginx/access   │
│  ss -tulpn → listening ports                                  │
└────────────────────────────────────────────────────────────────┘
```

---

## ✅ Knowledge Check

text

```
1.  What is the Linux file system hierarchy?
    Where do config files live? Log files?
2.  What do the three sets of rwx mean in ls -la output?
3.  What does chmod 755 give and what does it mean?
4.  What is the difference between kill and kill -9?
5.  How do you find what process is using port 8000?
6.  What is Nginx and why do you put it in front of your app?
7.  What is the difference between nginx -s reload
    and systemctl restart nginx?
8.  What are the three sections of a systemd service file?
9.  What does Restart=always do in systemd?
10. How do you follow your application logs in real-time?
11. What is PasswordAuthentication no in SSH config?
    Why is it important?
12. What does ufw default deny incoming do?
13. How do you check disk space on a server?
14. What does load average mean and what's a bad value?
```

---

## 🛠️ Practice Exercises

Bash

```
# Exercise 1: Basic Server Setup
# On a fresh Ubuntu 22.04 server (VM or container):
# 1. Update all packages
# 2. Create a new user 'appuser' with sudo access
# 3. Set up SSH key authentication for appuser
# 4. Disable password authentication
# 5. Configure UFW: allow SSH + HTTP + HTTPS only
# 6. Install Python 3.12, Nginx, PostgreSQL
# Verify each step before moving to the next.

# Exercise 2: Deploy a FastAPI App with Nginx
# 1. Copy your Phase 4.2 FastAPI app to the server
# 2. Create a virtual environment and install deps
# 3. Create a systemd service file for it
# 4. Start it with systemd
# 5. Configure Nginx as reverse proxy
# 6. Test: curl http://localhost/health
# 7. Make the service restart automatically
#    (kill -9 the process, verify it restarts)

# Exercise 3: Nginx Rate Limiting
# Add rate limiting to your Nginx config:
# - Max 100 requests/second per IP globally
# - Max 5 requests/minute for /auth/login
# Test with: for i in {1..20}; do curl http://localhost/auth/login; done
# Verify 429 responses after limit exceeded

# Exercise 4: Log Analysis
# Generate some traffic to your app, then:
# 1. Count total requests in access.log
# 2. Find top 5 most requested endpoints
# 3. Find all 404 errors
# 4. Find all requests from a specific IP
# 5. Calculate average response time
# Use: grep, awk, sort, uniq -c, head

# Exercise 5: Systemd and Process Management
# 1. Create a simple Python HTTP server as a systemd service
# 2. Make it start on boot
# 3. Configure it to restart after 5 seconds if it crashes
# 4. Set a memory limit of 256MB
# 5. Test crash recovery: kill the process, verify it restarts
# 6. View the restart logs in journalctl
```

---

## ✅ Phase 7.1 Complete!

**You now know:**

text

```
✅ Linux file system structure and navigation
✅ File permissions (chmod, chown) and their meanings
✅ Package management with apt
✅ Process management (ps, top, kill, background jobs)
✅ Finding processes by port (lsof, ss, fuser)
✅ Nginx installation and configuration
✅ Nginx as reverse proxy for FastAPI
✅ Nginx SSL, rate limiting, static files
✅ systemd service creation and management
✅ Service auto-restart and lifecycle management
✅ journalctl for log viewing
✅ SSH key setup and hardening
✅ SSH config file and tunneling
✅ UFW firewall configuration
✅ System monitoring (memory, CPU, disk, network)
✅ Cron jobs for scheduled tasks
✅ Complete deployment script
```

---