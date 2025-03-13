# UNION Validator NGINX Proxy Setup

This guide documents the setup of NGINX as a reverse proxy for a UNION validator node (Cosmos-based) to securely expose RPC, API, and gRPC endpoints for monitoring. It includes TLS encryption via Let's Encrypt, HTTP/2 for gRPC, and optional Cloudflare proxying for added security and performance.

Date: March 12, 2025  
Environment: Ubuntu (NGINX 1.24.0)

---

## Prerequisites

- **UNION Node**: Running with RPC (`26657`), API (`1317`), and gRPC (`9090`) enabled in `~/.union/config/config.toml`.
- **NGINX**: Installed (`sudo apt install nginx`).
- **Certbot**: For Let's Encrypt certificates (`sudo apt install certbot python3-certbot-nginx`).
- **Domain**: `eastwoodnode.xyz` with subdomains (`rpc`, `api`, `grpc`) pointing to your server’s IP.
- **Root Access**: Required for configuration.

---

## NGINX Configuration

### Initial Setup
- **File**: `/etc/nginx/sites-available/union-validator.conf`
- **Symlink**: Enabled via `sudo ln -s /etc/nginx/sites-available/union-validator.conf /etc/nginx/sites-enabled/`
- **Content**:
  ```nginx
  # HTTP to HTTPS redirect
  server {
      listen 80;
      server_name rpc.eastwoodnode.xyz api.eastwoodnode.xyz grpc.eastwoodnode.xyz;
      return 301 https://$host$request_uri;
  }

  # RPC Server
  server {
      listen 443 ssl http2;
      server_name rpc.eastwoodnode.xyz;
      ssl_certificate /etc/letsencrypt/live/rpc.eastwoodnode.xyz/fullchain.pem;
      ssl_certificate_key /etc/letsencrypt/live/rpc.eastwoodnode.xyz/privkey.pem;
      location / {
          proxy_pass http://127.0.0.1:26657;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      }
  }

  # API Server
  server {
      listen 443 ssl http2;
      server_name api.eastwoodnode.xyz;
      ssl_certificate /etc/letsencrypt/live/api.eastwoodnode.xyz/fullchain.pem;
      ssl_certificate_key /etc/letsencrypt/live/api.eastwoodnode.xyz/privkey.pem;
      location / {
          proxy_pass http://127.0.0.1:1317;
          proxy_set_header Host $host;
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      }
  }

  # gRPC Server
  server {
      listen 443 ssl http2;
      server_name grpc.eastwoodnode.xyz;
      ssl_certificate /etc/letsencrypt/live/grpc.eastwoodnode.xyz/fullchain.pem;
      ssl_certificate_key /etc/letsencrypt/live/grpc.eastwoodnode.xyz/privkey.pem;
      location / {
          grpc_pass grpc://127.0.0.1:9090;
          grpc_set_header X-Real-IP $remote_addr;
          grpc_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      }
  }

## Test and Reload:

```
sudo nginx -t
sudo systemctl reload nginx
```
### TLS Certificates with Certbot
Issue: Initial attempts failed due to missing certificates and Cloudflare proxying.
Solution: Used Certbot standalone mode after stopping NGINX:
```
sudo systemctl stop nginx
sudo certbot certonly --standalone -d rpc.eastwoodnode.xyz -d api.eastwoodnode.xyz -d grpc.eastwoodnode.xyz
```
### Result: Certificates in /etc/letsencrypt/live/<subdomain>.eastwoodnode.xyz/.
Note: Separate certs per subdomain; can consolidate into one cert later.

### Troubleshooting
HTTP/2 Error: unknown directive "http2" fixed by verifying ngx_http_v2_module with nginx -V.
Certificate Errors: Resolved by generating certs after fixing NGINX config.
301 Redirect Loop: Caused by Cloudflare proxying and NGINX misconfiguration. Fixed by:
Disabling Cloudflare proxy (DNS Only) temporarily.
Ensuring no return 301 in 443 blocks.

Port Conflicts: NGINX failed to bind to 80/443 due to stray processes (PID 59583). Fixed with:
```
sudo pkill -9 nginx
sudo systemctl start nginx
```
### Cloudflare Proxying
Enable Proxy: In Cloudflare DNS, set rpc, api, and grpc.eastwoodnode.xyz to Proxied (orange cloud).
SSL/TLS: Set to Full (strict) in Cloudflare SSL/TLS > Overview.

Test:
```
curl https://rpc.eastwoodnode.xyz/status
curl https://api.eastwoodnode.xyz/cosmos/bank/v1beta1/balances/<address>
grpcurl -insecure grpc.eastwoodnode.xyz:443 list
```
### Final Verification
Endpoints:
RPC: https://rpc.eastwoodnode.xyz/status → JSON (HTTP 200).
API: https://api.eastwoodnode.xyz/cosmos/bank/v1beta1/balances/<address> → JSON balances.
gRPC: grpcurl -insecure grpc.eastwoodnode.xyz:443 list → Service list.
Systemd:
```
sudo systemctl status nginx  # Active: active (running)
sudo systemctl enable nginx  # Auto-start on boot
```
Renewal:
```
sudo certbot renew --dry-run
```
### Security Hardening
Firewall:
```
sudo ufw allow 80
sudo ufw allow 443
sudo ufw status
```
Optional: Restrict to Cloudflare IPs (see https://www.cloudflare.com/ips/).

NGINX Logs:
Access: /var/log/nginx/access.log
Error: /var/log/nginx/error.log
