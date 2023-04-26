# Setting up SSL for Hub RPC on Digital Ocean

In this tutorial, we'll walk you through setting up SSL for the Hub RPC on a Digital Ocean server.

## Prerequisites

- A Digital Ocean server
- A domain or subdomain pointing to your server's IP address

## Step 1: Create an A record in your domain manager

Create an A record to point your domain or subdomain to your server's public IP address.

For a domain without a subdomain (e.g., domain.xyz):

`A | @ | (your public IP)`

For a domain with a subdomain (e.g., your.domain.xyz):

`A | your | (your public IP)`

At this point, you can use an insecure Hub RPC client with the HUB_URL set to your domain.

## Step 2: Update your system

Make sure your system is up to date:

```bash
sudo apt update
sudo apt upgrade
```

## Step 3: Install certbot and nginx

Install certbot and nginx on your server:

```bash
sudo apt install certbot
sudo apt install nginx
```

## Step 4: Create an SSL certificate with certbot

Generate a new SSL certificate using certbot:

```bash
sudo certbot certonly --standalone -d your.domain.xyz --preferred-challenges http --agree-tos -m you@example.com
```

## Step 5: Create an nginx configuration

Create a new nginx configuration file:

```bash
sudo nano /etc/nginx/sites-available/your.domain.xyz
```

Add the following configuration:

```bash
server {
    listen 80;
    server_name your.domain.xyz;
    return 301 https://$host$request_uri;
}

server {
    listen 2284 ssl http2;
    server_name your.domain.xyz;

    ssl_certificate /etc/letsencrypt/live/your.domain.xyz/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your.domain.xyz/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384';

    location / {
        grpc_pass grpc://127.0.0.1:2283;
        grpc_set_header X-Real-IP $remote_addr;
        grpc_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        grpc_set_header Host $http_host;
    }
}
```

## Step 6: Enable the site

Create a symlink to enable the site:

```bash
sudo ln -s /etc/nginx/sites-available/your.domain.xyz /etc/nginx/sites-enabled/
```

## Step 7: Test your configuration

Test your nginx configuration:

```bash
sudo nginx -t
```

## Step 8: Restart nginx

Restart (or start) your nginx server:

```bash
sudo systemctl restart nginx
```

If you encounter an error when starting nginx, run the following command to view its status:

```bash
sudo systemctl status nginx.service
```

## Step 9: Connect to your SSL Hub

```bash
const client = getSSLHubRpcClient("your.domain.xyz:2284");
```

That's it! You've successfully set up SSL for the Hub RPC on a Digital Ocean server.
