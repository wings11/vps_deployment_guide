# 🚀 Complete VPS Deployment Guide: From Zero to Production

## 📖 Overview
This guide shows how to migrate your backend API from Render to DigitalOcean VPS to eliminate cold starts and improve performance.

**🏗️ What we're building:**
- 🌐 **Frontend**: Netlify → `https://wingsportfolio.dev`
- ⚡ **Backend**: DigitalOcean VPS → `https://api.wingsportfolio.dev`

**🎯 Benefits:**
- ✅ No cold starts (24/7 running server)
- ⚡ Better performance 
- 💰 Cost-effective (~$6/month)
- 🔒 Professional setup with SSL

---

## 🌊 Step 1: Create DigitalOcean Account

### 1.1 📝 Sign Up
1. 🌐 Go to [digitalocean.com](https://digitalocean.com)
2. 🖱️ Click **"Sign up"**
3. ✍️ Create account with email and password
4. ✉️ Verify your email address

### 1.2 💳 Add Payment Method
1. 🧭 Navigate to **Account → Billing**
2. 💳 Add credit card or PayPal
3. 💵 DigitalOcean will charge $1 for verification (refunded)

### 1.3 🎁 Get Welcome Credit
- 🔍 Look for promotional codes for $100-200 free credit
- 🔗 Often available through referral links or current promotions

---

## 🔐 Step 2: Prepare SSH Keys

### 2.1 🔍 Check Existing SSH Keys
Open PowerShell and check if you have SSH keys:
```powershell
ls $env:USERPROFILE\.ssh\
```

### 2.2 🗝️ Use Existing Key
If you have `id_ed25519.pub`, use that. Otherwise, generate new ones:
```powershell
ssh-keygen -t ed25519 -C "your_email@example.com"
```

### 2.3 📋 Get Your Public Key
```powershell
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub
```
📝 Copy the output (starts with `ssh-ed25519`).

---

## 💧 Step 3: Create VPS Droplet

### 3.1 🔐 Add SSH Key to DigitalOcean
1. 🏠 In DigitalOcean dashboard, click your avatar → **Settings**
2. 🛡️ Click **Security** → **Add SSH Key**
3. 📋 Paste your public key
4. 🏷️ Name it (e.g., "wingsportfolio-key")
5. ➕ Click **Add SSH Key**

### 3.2 🖥️ Create Droplet
1. ➕ Click **Create** → **Droplets**
2. 🐧 **Image**: Ubuntu 24.04 (LTS) x64
3. 💰 **Plan**: 
   - 🌟 Basic $6/month (1GB RAM, 1 CPU, 25GB SSD) - **Recommended**
   - 📦 Or Basic $4/month (512MB RAM) - Minimum
4. 🌍 **Datacenter**: Choose closest to your users
5. 🔐 **Authentication**: Select **SSH Keys** (choose your key)
6. ⚙️ **Additional Options**: Enable monitoring
7. 🏷️ **Hostname**: `wingsportfolio-api`
8. 🚀 Click **Create Droplet**

### 3.3 📍 Note Your IP Address
⏰ After 1-2 minutes, note your droplet's IP address (e.g., `167.99.79.56`)

---

## 🔧 Step 4: Initial Server Setup

### 4.1 🔌 Connect to Server
In PowerShell:
```powershell
ssh -i "$env:USERPROFILE\.ssh\id_ed25519" root@YOUR_IP_ADDRESS
```

⚠️ **🖥️ You're now in the DigitalOcean Console!** All following commands run on your VPS.

### 4.2 🔄 Update System
```bash
apt update && apt upgrade -y
```

### 4.3 👥 Create Non-Root User
```bash
adduser --disabled-password --gecos '' wings
usermod -aG sudo wings
```

### 4.4 🔐 Setup SSH for New User
```bash
mkdir -p /home/wings/.ssh
cp /root/.ssh/authorized_keys /home/wings/.ssh/
chown -R wings:wings /home/wings/.ssh
chmod 700 /home/wings/.ssh
chmod 600 /home/wings/.ssh/authorized_keys
```

### 4.5 🔓 Configure Passwordless Sudo
```bash
echo 'wings ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers.d/wings
```

### 4.6 🛡️ Configure Firewall
```bash
ufw allow 22
ufw allow 80
ufw allow 443
ufw enable
```
✅ Type `y` when prompted.

### 4.7 🧪 Test New User Connection
Exit and reconnect as wings user:
```bash
exit
```

🔄 In PowerShell:
```powershell
ssh -i "$env:USERPROFILE\.ssh\id_ed25519" wings@YOUR_IP_ADDRESS
```

---

## 📦 Step 5: Install Node.js and Dependencies

⚠️ **🖥️ All commands now run as `wings` user on your VPS console:**

### 5.1 🔧 Install Essential Tools
```bash
sudo apt update
sudo apt install -y curl wget git unzip
```

### 5.2 📗 Install Node.js 20.x
```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### 5.3 ✅ Verify Installation
```bash
node --version
npm --version
```

### 5.4 🔄 Install PM2 Process Manager
```bash
sudo npm install -g pm2
```

### 5.5 🌐 Install Nginx
```bash
sudo apt install -y nginx
```

---

## 🚀 Step 6: Deploy Your Application

### 6.1 📁 Create App Directory
```bash
mkdir -p ~/wingsportfolio-api
cd ~/wingsportfolio-api
```

### 6.2 📤 Upload Server Files
⚠️ **💻 Run this in PowerShell (on your local machine):**
```powershell
scp -i "$env:USERPROFILE\.ssh\id_ed25519" -r server/* wings@YOUR_IP_ADDRESS:~/wingsportfolio-api/
```

### 6.3 📋 Create package.json
⚠️ **🖥️ Back on VPS console:**
```bash
cd ~/wingsportfolio-api
npm init -y
```

### 6.4 📦 Install Dependencies
```bash
npm install express express-rate-limit node-fetch@2.6.7 dotenv
```

### 6.5 ⚙️ Create Environment File
```bash
cat > .env << 'EOF'
AI_PROVIDER=groqcloud
AI_KEY=your_groqcloud_api_key_here
GROQCLOUD_URL=
CACHE_TTL_MS=3600000
PORT=5000
CORS_ORIGIN=https://wingsportfolio.dev
MB_USER_AGENT=wingsportfolio/1.0 (api@wingsportfolio.dev)
EOF
```

**⚠️ 🔑 Important: Replace `your_groqcloud_api_key_here` with your actual GroqCloud API key.**

### 6.6 🧪 Test Application
```bash
node index.js
```
✅ You should see: `Server listening on 5000`  
🛑 Press `Ctrl+C` to stop.

---

## 🔄 Step 7: Setup Process Management with PM2

### 7.1 ⚙️ Create PM2 Configuration
```bash
cat > ecosystem.config.js << 'EOF'
module.exports = {
  apps: [{
    name: 'wingsportfolio-api',
    script: 'index.js',
    instances: 1,
    autorestart: true,
    watch: false,
    max_memory_restart: '1G',
    env: {
      NODE_ENV: 'production',
      PORT: 5000
    },
    log_file: '~/logs/wingsportfolio-api.log',
    out_file: '~/logs/wingsportfolio-api-out.log',
    error_file: '~/logs/wingsportfolio-api-error.log',
    time: true
  }]
};
EOF
```

### 7.2 📁 Create Logs Directory and Start App
```bash
mkdir -p ~/logs
pm2 start ecosystem.config.js
```

### 7.3 🔄 Configure Auto-Start on Boot
```bash
pm2 startup
```

📋 Copy and run the command PM2 shows you (it will look like):
```bash
sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u wings --hp /home/wings
```

### 7.4 💾 Save PM2 Configuration
```bash
pm2 save
```

### 7.5 ✅ Verify Application is Running
```bash
pm2 status
curl http://localhost:5000/health
```

---

## 🌐 Step 8: Configure Nginx Reverse Proxy

### 8.1 🗑️ Remove Default Site
```bash
sudo rm /etc/nginx/sites-enabled/default
```

### 8.2 ⚙️ Create API Site Configuration
```bash
echo 'server {
    listen 80;
    server_name api.wingsportfolio.dev;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /health {
        proxy_pass http://127.0.0.1:5000/health;
        access_log off;
    }
}' | sudo tee /etc/nginx/sites-available/api.wingsportfolio.dev
```

### 8.3 🔗 Create Default IP Handler
```bash
echo 'server {
    listen 80 default_server;
    server_name _;

    location / {
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /health {
        proxy_pass http://127.0.0.1:5000/health;
        access_log off;
    }
}' | sudo tee /etc/nginx/sites-available/default-ip
```

### 8.4 🔌 Enable Sites
```bash
sudo ln -s /etc/nginx/sites-available/api.wingsportfolio.dev /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/default-ip /etc/nginx/sites-enabled/
```

### 8.5 ✅ Test and Reload Nginx
```bash
sudo nginx -t
sudo systemctl reload nginx
```

---

## 🌐 Step 9: Configure DNS

### 9.1 📝 Add DNS Record
Since you use Netlify nameservers, go to:
1. 🏠 **Netlify Dashboard** → **Domain Settings** → **DNS**
2. ➕ Add **A Record**:
   - 🏷️ **Name**: `api`
   - 📍 **Value**: `YOUR_VPS_IP_ADDRESS`
   - ⏰ **TTL**: `300` (5 minutes)

**🔄 Alternative**: If using Name.com nameservers:
1. 🌐 Go to Name.com → DNS Management for `wingsportfolio.dev`
2. ➕ Add same A record

### 9.2 ⏳ Wait for DNS Propagation
🕐 Wait 5-30 minutes for DNS to propagate globally.

### 9.3 🧪 Test DNS Resolution
⚠️ **💻 On your local machine (PowerShell):**
```powershell
nslookup api.wingsportfolio.dev 8.8.8.8
```
✅ Should show your VPS IP address.

---

## 🔒 Step 10: Install SSL Certificate

### 10.1 📦 Install Certbot
⚠️ **🖥️ On VPS console:**
```bash
sudo apt install -y certbot python3-certbot-nginx
```

### 10.2 🎫 Get SSL Certificate
```bash
sudo certbot --nginx -d api.wingsportfolio.dev --non-interactive --agree-tos --email your_email@example.com
```

**⚠️ 📧 Important: Replace `your_email@example.com` with your actual email.**

### 10.3 🧪 Test HTTPS
```bash
curl https://api.wingsportfolio.dev/health
```

---

## ⚛️ Step 11: Update Frontend Configuration

### 11.1 ⚙️ Update Environment Variables
⚠️ **📁 In your local project's `.env` file:**
```env
REACT_APP_API_URL=https://api.wingsportfolio.dev
```

### 11.2 🏗️ Build and Deploy
⚠️ **💻 In PowerShell (local machine):**
```powershell
npm run build
git add .
git commit -m "Update API URL to use VPS backend"
git push origin main
```

🚀 If using Netlify's GitHub integration, it will auto-deploy.

---

## 🧪 Step 12: Testing Your Setup

### 12.1 🔍 Test API Endpoints
⚠️ **🖥️ On VPS console:**
```bash
# 💚 Health check
curl https://api.wingsportfolio.dev/health

# 🎵 Song recommendation
curl -X POST -H "Content-Type: application/json" \
     -d '{"query":"happy, energetic, upbeat"}' \
     https://api.wingsportfolio.dev/api/recommend
```

### 12.2 🌐 Test Frontend
1. 🏠 Visit `https://wingsportfolio.dev`
2. 🎵 Go to your relax/music section
3. 🤖 Try the AI Song Recommender with keywords like:
   - 📚 "chill, relaxing, study"
   - 💪 "energetic, workout, pump up"
   - 🌧️ "sad, melancholy, rain"

---

## 🎯 Maintenance Commands

### 📊 Monitor Your Application
⚠️ **🖥️ On VPS console:**
```bash
# ✅ Check PM2 status
pm2 status

# 📜 View logs
pm2 logs wingsportfolio-api

# 🔄 Restart app
pm2 restart wingsportfolio-api

# 💻 Check system resources
htop

# 💾 Check disk space
df -h
```

### 🔄 Update Your Application
⚠️ **📤 Upload new files (PowerShell on local machine):**
```powershell
scp -i "$env:USERPROFILE\.ssh\id_ed25519" -r server/* wings@YOUR_IP:~/wingsportfolio-api/
```

⚠️ **🖥️ On VPS console:**
```bash
# 📦 Install new dependencies (if any)
cd ~/wingsportfolio-api
npm install

# 🔄 Restart application
pm2 restart wingsportfolio-api
```

---

## 🎉 Congratulations!

🌟 You now have a professional VPS setup with:

✅ **🚫 No Cold Starts** - Your API runs 24/7  
✅ **🔒 SSL Security** - HTTPS with auto-renewal  
✅ **🔄 Process Management** - PM2 handles crashes/restarts  
✅ **⚡ Reverse Proxy** - Nginx for better performance  
✅ **💰 Cost Effective** - Only ~$6/month  
✅ **🌐 Professional Domain** - `https://api.wingsportfolio.dev`

🎵 Your song recommendation feature will now work instantly without any delays! 🚀

---

## � Cost Breakdown

- 🌊 **DigitalOcean VPS**: $6/month (1GB RAM)
- 🌐 **Domain**: Already owned (`wingsportfolio.dev`)
- 🔒 **SSL Certificate**: Free (Let's Encrypt)
- 💵 **Total**: **$6/month** vs Render's higher pricing

🎯 **Savings**: Significant cost reduction + better performance + no cold starts!

---

## 🚨 Important Notes

- ⚠️ **💻 Console Context**: Pay attention to whether commands run on your local machine (PowerShell) or VPS console (bash)
- 🔑 **API Keys**: Don't forget to replace placeholder API keys with real ones
- 🌐 **DNS**: DNS propagation can take 5 minutes to 48 hours
- 💾 **Backups**: Consider setting up automated backups for your VPS
- 🔒 **Security**: Regularly update your system with `sudo apt update && sudo apt upgrade`

---

## 🆘 Support

🛠️ If you encounter issues:
1. 📜 Check PM2 logs: `pm2 logs`
2. 🌐 Check Nginx logs: `sudo tail -f /var/log/nginx/error.log`
3. ✅ Verify services: `sudo systemctl status nginx` and `pm2 status`
4. 🧪 Test API directly: `curl http://localhost:5000/health`

**🎯 Happy deploying!** 🚀✨