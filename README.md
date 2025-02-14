# 🚀 Django DRF Deployment on Ubuntu EC2 Server

This guide provides step-by-step instructions to deploy a Django REST Framework (DRF) application on an Ubuntu EC2 server using **Gunicorn, Supervisor, and Nginx** with **HTTPS support** and **GitHub integration**.

---
## ✅ 1. Update and Install Required Packages
### Update System Packages
```bash
sudo apt-get update && sudo apt-get upgrade -y
```

### Install Dependencies
```bash
sudo apt update && sudo apt install -y build-essential libpq-dev python3-dev python3-venv nginx supervisor
```

### Verify Python Installation
```bash
python3 --version
```

---
## 📂 2. Clone Repository & Setup Virtual Environment
### Clone the Project
```bash
git clone https://github.com/misbahulamin/erp-backend.git
cd erp-backend
```

### Create and Activate Virtual Environment
```bash
python3 -m venv venv
source venv/bin/activate
```

To deactivate the virtual environment:
```bash
deactivate
```

### Install Project Dependencies
```bash
pip install -r requirements.txt
pip install gunicorn
```

---
## ⚙️ 3. Configure Supervisor for Gunicorn
### Install Supervisor
```bash
sudo apt-get install supervisor -y
```

### Create Supervisor Configuration
```bash
cd /etc/supervisor/conf.d/
sudo touch gunicorn.conf
sudo nano gunicorn.conf
```

### Add the following content:
```ini
[program:gunicorn]
directory=/home/ubuntu/erp-backend
command=/home/ubuntu/venv/bin/gunicorn --workers 3 --bind unix:/home/ubuntu/erp-backend/app.sock core.wsgi:application
autostart=true
autorestart=true
stderr_logfile=/var/log/gunicorn/gunicorn.err.log
stdout_logfile=/var/log/gunicorn/gunicorn.out.log

[group:guni]
programs:gunicorn
```

### Create a Log Directory
```bash
sudo mkdir /var/log/gunicorn
```

### Restart Supervisor
```bash
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl status
```

---
## 🌐 4. Configure Nginx
### Create Nginx Configuration File
```bash
cd /etc/nginx/sites-available
sudo touch django.conf
sudo nano django.conf
```

### Add the following content:
```nginx
server {
    listen 80;
    server_name 3.107.200.148;

    location / {
        include proxy_params;
        proxy_pass http://unix:/home/ubuntu/erp-backend/app.sock;
    }
}
```

### Test & Enable Nginx Configuration
```bash
sudo nginx -t
sudo ln -s /etc/nginx/sites-available/django.conf /etc/nginx/sites-enabled/
sudo service nginx restart
```

### Modify the Domain Name (if needed)
```bash
sudo nano /etc/nginx/sites-available/django.conf
# Change server_name to salarybook.panamach.com
```

### Reload Nginx
```bash
sudo systemctl reload nginx
```

---
## 🔒 5. Enable HTTPS with Certbot
```bash
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d salarybook.panamach.com
```

---
## 🔗 6. Connect GitHub with Server
### Generate SSH Key
```bash
ssh-keygen -t rsa -b 4096 -C "panaceatechltd@gmail.com"
```
Press `Enter` until the key is created.

### Display the Public Key
```bash
cat /home/ubuntu/.ssh/id_rsa.pub
```
Add this key to GitHub under **Settings → Deploy Keys**.

### Verify SSH Key
```bash
ls -l ~/.ssh/
cat ~/.ssh/id_rsa
```

---
## 🔄 7. Setup GitHub Actions for Deployment
### Create GitHub Actions Workflow
Create a file `.github/workflows/deploy.yml` in your project repository and add the following content:

```yaml
name: Deploy to Ubuntu Server

on:
  push:
    branches:
      - main  # Change this to your deployment branch if it's not 'main'

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup SSH Connection
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.EC2_PRIVATE_KEY }}" > ~/.ssh/id_rsa
          chmod 600 ~/.ssh/id_rsa
          ssh-keyscan -H 52.63.110.224 >> ~/.ssh/known_hosts

      - name: Deploy to Server
        run: |
          ssh ubuntu@52.63.110.224 << 'EOF'
            set -e  # Exit if any command fails
            echo "Updating system packages..."
            sudo apt update && sudo apt upgrade -y
            
            echo "Pulling latest code from GitHub..."
            cd ~/erp-backend
            git pull origin main
            
            echo "Running system update again..."
            sudo apt update && sudo apt upgrade -y
            
            echo "Restarting server..."
            sudo reboot
          EOF
```

### How to Add GitHub Secrets
1. Navigate to your GitHub repository.
2. Go to **Settings → Secrets and Variables → Actions**.
3. Click **New repository secret**.
4. Add a secret named `EC2_PRIVATE_KEY` and paste the private key of your EC2 instance.

Now, every time you push to the `main` branch, the workflow will deploy your latest changes to the EC2 server.

---
## 🔄 8. Restart Services
```bash
sudo supervisorctl restart gunicorn
sudo systemctl restart nginx
```

Now your **Django DRF application** should be live on your **domain or IP address**. 🚀

---
## 🛠️ Troubleshooting
- **Check Gunicorn logs:**
  ```bash
  cat /var/log/gunicorn/gunicorn.err.log
  ```
- **Check Nginx logs:**
  ```bash
  sudo journalctl -u nginx --since today
  ```
- **Restart all services:**
  ```bash
  sudo supervisorctl restart gunicorn
  sudo systemctl restart nginx
  ```




# django-ec2-deployment-guide
Step-by-step guide to deploying a Django REST Framework (DRF) application on an Ubuntu EC2 server with Gunicorn, Supervisor, and Nginx. Includes HTTPS setup with Certbot and GitHub integration for seamless deployment. Ideal for beginners and professionals looking for an efficient deployment solution.
