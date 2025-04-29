Open PuTTY

Enter your EC2 Public IP in "Host Name"

Under "SSH" -> "Auth", load your .ppk private key file

Click "Open" to connect.

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
docker-compose version</code>
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
      - WEBHOOK_URL=https://yourwebsiteurl/
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




