Here's a **visually appealing, step-by-step guide** to deploy NGINX on AWS EC2 Ubuntu (24.04 LTS). 🚀 Each step includes emojis, code blocks, and tips for clarity.
## 🟢 Step 1: Launch EC2 Instance

Go to AWS Console → EC2 → **Launch instance**.

- **AMI**: Ubuntu Server 24.04 LTS (free tier).  
- **Type**: t2.micro.  
- **Key pair**: Create/download .pem (for SSH).  
- **Security Group** (critical!):  
  ✅ SSH: TCP 22 (your IP only).  
  ✅ HTTP: TCP 80 (0.0.0.0/0). 

**Copy Public IP** after launch. 📋

## 🔵 Step 2: SSH Connect

Open terminal:

```bash
ssh -i ~/Downloads/your-key.pem ubuntu@EC2_PUBLIC_IP
```

✅ You're in as `ubuntu` user.

## 🟡 Step 3: Update Packages

Keep system fresh:

```bash
sudo apt update && sudo apt upgrade -y
```

⏳ Takes 1-2 mins. [cherryservers]
## 🟠 Step 4: Install NGINX

```bash
sudo apt install nginx -y
```

Auto-starts! Enable on reboot:

```bash
sudo systemctl enable --now nginx
```

✅ Check: `sudo systemctl status nginx` (shows "active"). 

## 🟣 Step 5: Allow Through Firewall

```bash
sudo ufw allow 'Nginx HTTP'
sudo ufw reload
sudo ufw status
```

✅ Port 80 open.

## 🔴 Step 6: Test Default Site

**Browser**: `http://EC2_PUBLIC_IP`  
✅ See "Welcome to nginx!" page.

**Local test**:

```bash
curl http://localhost
```

## 🟤 Step 7: Customize Your Page

Edit HTML:

```bash
sudo nano /var/www/html/index.html
```

Add sample content:
```html
<h1>🚀 My NGINX Site Live on AWS!</h1>
<p>EC2 Ubuntu + NGINX deployed successfully. 📈</p>
```

Reload (zero downtime):

```bash
sudo nginx -t  # ✅ Syntax OK?
sudo systemctl reload nginx
```

Refresh browser → new page! ✨

## ⚫ Quick Commands Table

| Emoji | Action | Command |
|-------|--------|---------|
| ▶️ | Start | `sudo systemctl start nginx`  |
| ⏹️ | Stop | `sudo systemctl stop nginx` |
| 🔄 | Restart | `sudo systemctl restart nginx` |
| 🔄 | Reload | `sudo systemctl reload nginx` |
| 📋 | Status | `sudo systemctl status nginx` |
| 🔍 | Test config | `sudo nginx -t` |
| 📜 | Logs | `sudo tail -f /var/log/nginx/access.log` |

## ✅ Troubleshooting Tips

- **No access?** Check Security Group (port 80 inbound).62462230/cannot-reach-nginx-on-ubuntu-aws-ec2-server)
- **NGINX down?** `sudo journalctl -u nginx -f`.
- **Next?** Add HTTPS: `sudo apt install certbot python3-certbot-nginx`. 
