# Digital Ocean gRPC web proxy

This tutorial will allow you to use SSLHubRpcClient for hub-nodejs and HubRpcClient for hub-web. gRPC (nodejs) will be exposed on port 2284 and gRPC-web (hub-web) will be exposed on port 2285 with a gRPC web proxy on port 2286.

## Requirements

- SSL Secured Hub ([Tutorial Here](https://github.com/alexpaden/farcaster-resources/blob/main/Tutorials/DIGITAL_OCEAN_SSL.md))

## Steps

### 1. Open your Nginx configuration file:

```
sudo nano /etc/nginx/sites-available/your.domain.xyz
```

and add a new server block for port 2285:

```
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
        grpc_pass grpc://0.0.0.0:2283;
        grpc_set_header X-Real-IP $remote_addr;
        grpc_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        grpc_set_header Host $http_host;
    }
}

server {
    listen 2285 ssl http2;
    server_name your.domain.xyz;

    ssl_certificate /etc/letsencrypt/live/your.domain.xyz/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your.domain.xyz/privkey.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384';

    location / {
        proxy_pass http://0.0.0.0:2286;
        proxy_http_version 1.1;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

### 2. Test your configuration and restart Nginx:

```
sudo nginx -t
sudo systemctl restart nginx
```

### 3. Install grpc-web:

```
sudo apt-get update
sudo apt-get install -y build-essential autoconf libtool pkg-config
git clone https://github.com/grpc/grpc-web.git
cd grpc-web
sudo apt-get install libprotoc-dev
sudo make install-plugin
```

### 4. Install Go:

```
sudo apt-get update
sudo apt-get install golang-go
```

### 5. Install grpcwebproxy

```
go install github.com/improbable-eng/grpc-web/go/grpcwebproxy@latest
```

### 6. Start the grpcwebproxy on port 2286 using pm2:

```
cd ..

pm2 start "./go/bin/grpcwebproxy --backend_addr=0.0.0.0:2283 --run_tls_server=false --server_http_debug_port=2286" --name web-proxy --exp-backoff-restart-delay=100
```

### 7. Test your server by making requests to https://your.domain.xyz:2285.

```
That's it! You've successfully set up a gRPC web proxy on your Digital Ocean node.

Test your server using the chron-feed example for hub-web AND hub-nodejs!

For web:
HUB_URL = 'https://galaxy.ditti.xyz:2285'

For nodeJS:
HUB_URL = 'galaxy.ditti.xyz:2284'
```

## What is a gRPC web proxy?

A gRPC web proxy serves as a bridge between gRPC and HTTP/1.1 or HTTP/2. It allows a browser-based app to use gRPC, which is not natively supported by the browser. The gRPC web proxy translates gRPC calls into HTTP requests that can be sent over the web. This makes it possible for web apps to access gRPC services seamlessly.

In summary, a gRPC web proxy is a server that acts as an intermediary between a web browser or mobile app and a gRPC service, allowing these clients to communicate with the service over the web.
