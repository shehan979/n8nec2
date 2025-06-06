## 100% Tested n8n install on AWS EC2 Guide
## Instruction Video
Step by Step Guide
https://youtu.be/-rcjJbUnaFQ
## 1. Requirements Checklist

AWS Account ready

The domain name was purchased (Namecheap, GoDaddy, etc.).

PuTTY is installed on your computer.

The .ppk key downloaded for EC2 access

## 2. Setup EC2 Server
1. Go to AWS Console > EC2 > Launch Instance.

2. Give a name for the Server and choose Amazon Linux 2023 (or Ubuntu 22.04 LTS).

3. Choose Instance Type: t2.micro (free tier), t2.small (better), or a higher version.

4. Create a new Key Pair (or use an existing one) and download the .ppk file.

5. Launch the instance.

7. Go to AWS Console > EC2 > Instances

8. Select your instance and click on the "Security" tab.

9. Click on the security group linked to your instance.

10. Click "Inbound Rules" and "Edit Inbound Rules".

11. Configure Security Group (Firewall):

12. Click "Add Rule" and add below additional ports:

    Allow HTTP (80) from 0.0.0.0/0

    Allow HTTPS (443) from 0.0.0.0/0

    Allow Custom TCP (5678) from 0.0.0.0/0 if you want direct n8n test access (optional)

14. Go back to instance and check the Instance state on Running.

## 3. Point Your Domain to EC2

1. Go to your domain registrar (Namecheap, GoDaddy, etc.).

2. Find DNS settings for your domain.

3. Create an A Record:

    Type: A

    Host: n8n (or @ if root)

    Value: EC2 Public IP (e.g., 12.61.155.33)

    TTL: Auto or 5 minutes

✅ This connects your domain yourdomain.com to your EC2 server.

(Propagation may take 5 minutes to 1 hour.)

## 4. Connect n8n on EC2 using PUTTY
Open PuTTY (if not download it: https://www.putty.org/)

Enter your EC2 Public IP in "Host Name"

Under "SSH" -> "Auth" -> "Credentials (Private key file for authentication)", load your .ppk private key file

Click "Open" to connect.

Accept the PuTTY Security Alert

Open and login as:
```
ec2-user
```
Update and install Docker
```
sudo yum update -y
```
```
sudo yum install -y docker
```
Start Docker
```
sudo systemctl start docker
```
```
sudo systemctl enable docker
```
Add user to docker group
```
sudo usermod -aG docker ec2-user
```
```
newgrp docker
```
Install Docker Compose
```
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
```
sudo chmod +x /usr/local/bin/docker-compose
```
Confirm docker-compose
```
docker-compose version
```
Install NGINX and Certbot (SSL)
```
sudo dnf install -y nginx
```
```
sudo systemctl start nginx
```
```
sudo systemctl enable nginx
```
```
sudo dnf install -y certbot python3-certbot-nginx
```
Set up n8n with Docker Compose
```
mkdir ~/n8n
```
```
cd ~/n8n
```
```
nano docker-compose.yml
```
Set up this inside docker-compose.yml
```
version: '3'
services:
  n8n:
    image: n8nio/n8n
    restart: always
    ports:
      - "5678:5678"
    environment:
      - N8N_BASIC_AUTH_ACTIVE=true
      - N8N_BASIC_AUTH_USER=admin
      - N8N_BASIC_AUTH_PASSWORD=yourStrongPasswordHere
      - WEBHOOK_URL=https://yourwebsitedomain/
    volumes:
      - n8n_data:/home/node/.n8n
volumes:
  n8n_data:
```
Save and exit (Ctrl + O, Enter, Ctrl + X)

Start n8n
```
docker-compose up -d
```
Setup NGINX Reverse Proxy
```
sudo nano /etc/nginx/conf.d/n8n.conf
```
Paste the following
```
server {
    listen 80;
    server_name yourdomainaddress;
    location / {
        proxy_pass http://localhost:5678/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```
Save and exit (Ctrl + O, Enter, Ctrl + X)

Test NGINX config:
```
sudo nginx -t
```
Reload NGINX:
```
sudo systemctl restart nginx
```
Issue SSL Certificate with Certbot
```
sudo certbot --nginx -d yourwebsitedomain
```
At that time, it will ask:

Your email address (for urgent expiry notices)

Agree to Let's Encrypt Terms of Service (Yes/No)

Optionally ask if you want to share your email with EFF (Electronic Frontier Foundation)

✅ You must enter an email and agree to continue issuing the free SSL certificate.

✅ This is normal and only happens once when you first request SSL.

Final Restart to apply everything
```
cd ~/n8n
```
```
docker-compose down
```
```
docker-compose up -d
```
```
sudo systemctl restart nginx
```
Open in browser:

https://yourwebsitedomain

Congratulations!

You have now fully installed and secured n8n on AWS EC2, accessed by domain with SSL, fully production ready!

## Step-by-Step Guide to Update n8n

This will pull the latest image to your server.
```
docker pull n8nio/n8n
```
Update the Docker Compose File (optional), & You can also pin a specific version like n8nio/n8n:1.38.1 if needed.
```
image: n8nio/n8n:latest
```
Start n8n with the New Image
```
docker-compose up -d
```
