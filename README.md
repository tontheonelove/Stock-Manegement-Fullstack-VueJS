# 🗂️Stock-Manegement-Fullstack-VueJS🗂️

## 🗄️Server Requirement 🗄️
1. ubuntu server 20++
2. mysql server
3. node js 20+
4. nginx


โครงสร้าง app

`````
stockmanagement---| frontend
                  | backend
`````
                  
### 🛠️start installation guide🛠️

🛠️ 1. เตรียมระบบพื้นฐาน
`````
# อัปเดตระบบ
sudo apt update && sudo apt upgrade -y

# ติดตั้งเครื่องมือพื้นฐาน
sudo apt install -y curl git build-essential ufw
`````

🌐 2. ติดตั้ง Node.js + Vue.js (Frontend)
`````
# ติดตั้ง Node.js LTS (20.x)
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt install -y nodejs

# ตรวจสอบ
node -v
npm -v

# สร้างโปรเจกต์ Vue + TS  (Create frontend folder)
npm create vue@latest frontend
# ✔ Add TypeScript? Yes
# ✔ Add Router? Yes
# ✔ Add Pinia? Yes
cd frontend
npm install
npm run dev   # ทดสอบ
`````

🐍 3. ติดตั้ง Python + Virtualenv (Backend)

`````
# ติดตั้ง Python
sudo apt install -y python3 python3-venv python3-pip

# สร้าง backend project
mkdir backend && cd backend
python3 -m venv venv
source venv/bin/activate

# ติดตั้ง FastAPI + Uvicorn
pip install fastapi uvicorn mysql-connector-python

test
uvicorn main:app --reload --host 0.0.0.0 --port 8000

สร้าง service
nano /etc/systemd/system/netstock-backend.service
`````
ปรับตาม path จริง

`````
[Unit]
Description=NetStock Backend Service (FastAPI)
After=network.target

[Service]
# รันด้วย root (เพราะ path อยู่ใน /root)
User=root
Group=root

# โฟลเดอร์ backend
WorkingDirectory=/root/stock-management/backend

# เรียกใช้ uvicorn จาก virtualenv (ถ้ามี venv)
ExecStart=/root/stock-management/backend/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000

# ถ้าไม่มี venv ใช้ python3 ตรง ๆ ได้ เช่น:
# ExecStart=/usr/bin/python3 -m uvicorn main:app --host 0.0.0.0 --port 8000

Restart=always
RestartSec=5
Environment="PATH=/root/stock-management/backend/venv/bin"

[Install]
WantedBy=multi-user.target
`````
▶️ คำสั่งจัดการ service

`````
# โหลด service ใหม่
sudo systemctl daemon-reload

# เริ่มรัน
sudo systemctl start netstock-backend

# ให้รันอัตโนมัติหลังบูต
sudo systemctl enable netstock-backend

# ตรวจสอบสถานะ
sudo systemctl status netstock-backend

# ดู log แบบ real-time
journalctl -u netstock-backend -f
`````


🗄️ 4. ติดตั้ง MySQL
`````
sudo apt install -y mysql-server
sudo systemctl enable mysql
sudo systemctl start mysql

# ตั้งรหัส root
sudo mysql_secure_installation
`````
### สร้าง DB + User
`````
CREATE DATABASE netstock;
CREATE USER 'netstock'@'localhost' IDENTIFIED BY 'StrongPassword123';
GRANT ALL PRIVILEGES ON netstock.* TO 'netstock'@'localhost';
FLUSH PRIVILEGES;

USE netstock;
CREATE TABLE inventory (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100),
  category VARCHAR(50),
  status VARCHAR(20)
);

`````

🌍 5. ติดตั้ง Nginx (Reverse Proxy)

`````
sudo apt install -y nginx
sudo systemctl enable nginx
sudo systemctl start nginx

`````

### สร้าง config /etc/nginx/sites-available/netstock
`````
server {
    listen 80;
    server_name yourdomain.com;

    location /api/ {
        proxy_pass http://127.0.0.1:8000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location / {
        root /var/www/frontend/dist;
        index index.html;
        try_files $uri /index.html;
    }
}
`````
`````
sudo ln -s /etc/nginx/sites-available/netstock /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
`````












