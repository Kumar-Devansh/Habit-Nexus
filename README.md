![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)

# 🚀 Habit Nexus

<div align="center">

![Python](https://img.shields.io/badge/Python-3.10+-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Flask](https://img.shields.io/badge/Flask-Web%20Framework-000000?style=for-the-badge&logo=flask&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-Database-4169E1?style=for-the-badge&logo=postgresql&logoColor=white)
![Gunicorn](https://img.shields.io/badge/Gunicorn-WSGI-499848?style=for-the-badge)
![Nginx](https://img.shields.io/badge/Nginx-Reverse%20Proxy-009639?style=for-the-badge&logo=nginx&logoColor=white)
![AWS EC2](https://img.shields.io/badge/AWS-EC2-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)

**A Production-Ready Habit Tracking Web Application**

Build habits • Track progress • Stay consistent

</div>

---

# 📖 Overview

Habit Nexus is a production-ready habit tracking platform developed using **Flask**, **PostgreSQL**, and **Gunicorn**, deployed on an **AWS EC2 Ubuntu Server** behind **Nginx**.

The application enables users to create habits, maintain daily routines, monitor consistency, and visualize progress through an intuitive interface. The deployment follows industry-standard practices including process management with **systemd**, reverse proxy configuration using **Nginx**, environment variable management, and PostgreSQL database integration.

---

# ✨ Features

- 🔐 Secure User Authentication
- 📅 Daily Habit Tracking
- 📈 Progress Analytics
- 🎯 Routine Management
- 🗂 PostgreSQL Database
- ⚡ Gunicorn Production Server
- 🌐 Nginx Reverse Proxy
- 🔄 Systemd Service Management
- 🔒 Environment Variable Configuration
- ☁ AWS EC2 Deployment
- 🚀 Production Ready Architecture

---

# 🛠 Tech Stack

| Category | Technology |
|----------|------------|
| Backend | Flask |
| Language | Python |
| Database | PostgreSQL |
| ORM | psycopg2 |
| WSGI Server | Gunicorn |
| Reverse Proxy | Nginx |
| Process Manager | systemd |
| Server | AWS EC2 Ubuntu |
| Version Control | Git |
| Environment | Python Virtual Environment |

---

# 🏗 Production Architecture

```
                    Internet
                        │
                        ▼
                  Nginx (Port 80)
                        │
                        ▼
           Gunicorn (127.0.0.1:8000)
                        │
                        ▼
                 Flask Application
                        │
                        ▼
                 PostgreSQL Database
```

---

# 📂 Project Structure

```
Habit_Nexus/
│
├── app.py
├── database.py
├── requirements.txt
├── .env
├── venv/
├── templates/
├── static/
├── instance/
├── migrations/
└── README.md
```

---

# ⚙ Deployment Workflow

## 1. Clone Repository

```bash
git clone https://github.com/Kumar-Devansh/Habit_Nexus.git
cd Habit_Nexus
```

---

## 2. Create Virtual Environment

```bash
python3 -m venv venv
source venv/bin/activate
```

---

## 3. Install Dependencies

```bash
pip install -r requirements.txt
pip install gunicorn psycopg2-binary
```

---

## 4. Configure Environment Variables

Create a `.env` file.

Example:

```env
SECRET_KEY=your_secret_key

DATABASE_URL=postgresql://username:password@localhost/database

SESSION_COOKIE_SECURE=False
```

---

## 5. Initialize Database

```bash
python3 -c "from database import init_db; init_db()"
```

---

## 6. Run Development Server

```bash
python app.py
```

---

## 7. Run Production Server

```bash
gunicorn -w 3 -b 127.0.0.1:8000 app:app
```

---

# 🚀 Production Deployment

The project is deployed using:

- Ubuntu Server
- AWS EC2
- PostgreSQL
- Gunicorn
- Nginx
- systemd

---

# ⚡ Systemd Service

Example service:

```ini
[Unit]
Description=Habit Nexus

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/Habit_Nexus
EnvironmentFile=/home/ubuntu/Habit_Nexus/.env
ExecStart=/home/ubuntu/Habit_Nexus/venv/bin/gunicorn -w 3 -b 127.0.0.1:8000 app:app
Restart=always

[Install]
WantedBy=multi-user.target
```

Enable service

```bash
sudo systemctl daemon-reload
sudo systemctl enable habitnexus
sudo systemctl start habitnexus
```

---

# 🌐 Nginx Configuration

```nginx
server {

    listen 80;

    server_name YOUR_PUBLIC_IP;

    location / {

        proxy_pass http://127.0.0.1:8000;

        proxy_set_header Host $host;

        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

        proxy_set_header X-Forwarded-Proto $scheme;

    }

}
```

Restart nginx

```bash
sudo nginx -t
sudo systemctl restart nginx
```

---

# 🗄 PostgreSQL Setup

Initialize PostgreSQL

```bash
sudo -u postgres psql
```

Verify Database Connection

```bash
echo $DATABASE_URL

psql "$DATABASE_URL"
```

Initialize tables

```bash
python3 -c "from database import init_db; init_db()"
```

---

# 🔍 Monitoring & Debugging

## Gunicorn

```bash
ps -ef | grep gunicorn

sudo pkill gunicorn
```

---

## systemd Logs

```bash
sudo journalctl -u habitnexus -f
```

---

## Check Service Status

```bash
sudo systemctl status habitnexus
```

---

## Check Listening Ports

```bash
sudo ss -tulpn | grep 8000
```

---

## Verify Nginx

```bash
sudo nginx -t
```

---

## Test Backend

```bash
curl http://127.0.0.1:8000
```

---

## Test Reverse Proxy

```bash
curl http://127.0.0.1
```

---

# 💻 Useful Commands

Activate Virtual Environment

```bash
source venv/bin/activate
```

Pull Latest Changes

```bash
git pull
```

Restart Application

```bash
sudo systemctl restart habitnexus
```

Restart Nginx

```bash
sudo systemctl restart nginx
```

View Logs

```bash
sudo journalctl -u habitnexus -f
```

Check Disk

```bash
df -h
```

Check RAM

```bash
free -h
```

---

# 🔐 Security Practices

- Environment variables stored in `.env`
- PostgreSQL authentication
- Gunicorn bound to localhost only
- Nginx reverse proxy
- systemd process isolation
- Secret key management
- Session security configuration

---

# 📚 Skills Demonstrated

- Flask Development
- PostgreSQL Integration
- Production Deployment
- Linux Administration
- AWS EC2
- Gunicorn Configuration
- Nginx Reverse Proxy
- systemd Services
- Environment Management
- Debugging Production Servers
- Linux Networking
- Git Workflow
- Secure Deployment Practices

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

# 👨‍💻 Author

** Kumar Devansh**

GitHub

https://github.com/Kumar-Devansh

---

# ⭐ Support

If you found this project helpful, consider giving it a ⭐ on GitHub.

It helps others discover the project and motivates future improvements.

---
