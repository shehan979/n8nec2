Open PuTTY

Enter your EC2 Public IP in "Host Name"

Under "SSH" -> "Auth", load your .ppk private key file

Click "Open" to connect.

Update and install Docker

##
<tab><tab>code/sudo yum update -y</tab></tab>

<code>sudo yum install -y docker</code>

Start Docker

<code>sudo systemctl start docker</code>

<code>sudo systemctl enable docker</code>

Add user to docker group

<code>sudo usermod -aG docker ec2-user</code>

<code>newgrp docker</code>

Install Docker Compose

<code>sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose</code>

<code>sudo chmod +x /usr/local/bin/docker-compose</code>

Confirm docker-compose

<code>docker-compose version</code>

Install NGINX and Certbot (SSL)

<code>sudo dnf install -y nginx</code>

<code>sudo systemctl start nginx</code>

<code>sudo systemctl enable nginx</code>

<code>sudo dnf install -y certbot python3-certbot-nginx</code>

Set up n8n with Docker Compose

<code>mkdir ~/n8n</code>

<code>cd ~/n8n</code>

<code>nano docker-compose.yml</code>

Set up this inside docker-compose.yml

<code>version: '3'
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
  n8n_data:</code>

Save and exit (Ctrl + O, Enter, Ctrl + X)

Start n8n

<code>docker-compose up -d</code>

Setup NGINX Reverse Proxy

<code>sudo nano /etc/nginx/conf.d/n8n.conf</code>

Paste the following

<code>server {
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
}</code>

Save and exit (Ctrl + O, Enter, Ctrl + X)




