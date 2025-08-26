# Django App Deployment on Domain with Nginx & SSL

This guide explains how to deploy your Django project (`Corevai`) on a domain using **Nginx** as a reverse proxy, and secure it with **Let's Encrypt SSL**.

---

## 1. Point Domain to Server

1. Log in to your domain DNS manager (where `brewyourbrandofficial.com` is registered).
2. Add an A record:

```
Host: myhr
Type: A
Value: 34.67.91.119
TTL: Auto / 3600
```

- This points: `myhr.brewyourbrandofficial.com` → `34.67.91.119`.

3. Test the DNS resolution:

```bash
ping myhr.brewyourbrandofficial.com
```

---

## 2. Install & Configure Nginx

Nginx will act as a reverse proxy to forward traffic to Gunicorn.

```bash
sudo apt update
sudo apt install nginx -y
```

### Create a site config for your project

```bash
sudo nano /etc/nginx/sites-available/corevai
```

Add the following configuration:

```nginx
server {
    server_name myhr.brewyourbrandofficial.com;

    location / {
        proxy_pass http://127.0.0.1:8000;   # Gunicorn/Django running on port 8000
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Enable the configuration

```bash
sudo ln -s /etc/nginx/sites-available/corevai /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

- Visit: [http://myhr.brewyourbrandofficial.com](http://myhr.brewyourbrandofficial.com) to verify.

---

## 3. Secure with SSL (HTTPS)

Install Certbot for Let’s Encrypt:

```bash
sudo apt install certbot python3-certbot-nginx -y
```

Run Certbot to generate SSL:

```bash
sudo certbot --nginx -d myhr.brewyourbrandofficial.com
```

This will:
- Verify your domain.
- Get SSL certificate.
- Automatically update Nginx config for HTTPS.

After success, your site will be live at:

```
https://myhr.brewyourbrandofficial.com
```

---

## 4. Auto Renew SSL

Let’s Encrypt certificates expire every 90 days. Enable auto-renewal:

```bash
sudo systemctl enable certbot.timer
```

---

## ✅ Summary

- Domain points to your server.
- Nginx reverse proxies requests to Gunicorn.
- SSL is enabled with HTTPS.
- Auto-renewal is set for Let’s Encrypt certificate.

> Optional: You can also manually configure Nginx for SSL to have full control instead of letting Certbot edit the file automatically.

