# Anadolu Yakası Evde Fizyoterapi & Manuel Terapi Web Site (`deed`)

Anadolu Yakası evde fizyoterapi ve manuel terapi randevu ve bilgilendirme web sitesi.

---

## 🚀 Nginx & SSL Setup Guide for Ubuntu 24 (24.04 LTS / 24.10)

This guide covers installing Nginx on **Ubuntu 24**, deploying static web assets, setting up **Virtual Hosts (Server Blocks)**, installing **Certbot**, connecting SSL certificates to custom domains, and managing automatic certificate renewals.

---

### 1. Update Packages & Install Nginx

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y nginx
```

Verify Nginx is enabled and running:

```bash
sudo systemctl enable --now nginx
sudo systemctl status nginx
```

---

### 2. Configure Firewall (UFW)

Allow HTTP and HTTPS traffic through the Ubuntu firewall:

```bash
sudo ufw allow 'Nginx Full'
sudo ufw reload
sudo ufw status
```

---

### 3. Deploy Project Files to Web Root

Create the web directory for the project, copy all project files, and grant permissions to Nginx (`www-data` user):

```bash
# Create directory in /var/www
sudo mkdir -p /var/www/deed

# Copy project folders (run from the project folder)
sudo cp -r tr ru images /var/www/deed/

# Set ownership and permissions
sudo chown -R www-data:www-data /var/www/deed
sudo chmod -R 755 /var/www/deed
```

---

### 4. Creating Virtual Hosts (Server Blocks)

Virtual hosts (Server Blocks) allow hosting **multiple websites or subdomains** (e.g. `istanadoludeed.com` and `site2.com`) on a single Ubuntu server.

#### A. Create Virtual Host Configuration

```bash
sudo nano /etc/nginx/sites-available/deed
```

Add the server block configuration:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name istanadoludeed.com www.istanadoludeed.com;

    root /var/www/deed;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    # Static asset caching
    location ~* \.(png|jpg|jpeg|gif|ico|svg|css|js)$ {
        expires 30d;
        add_header Cache-Control "public, no-transform";
    }

    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml image/svg+xml;
}
```

#### B. Enable Virtual Host & Reload Nginx

```bash
# Enable the site configuration
sudo ln -s /etc/nginx/sites-available/deed /etc/nginx/sites-enabled/

# Disable default site (optional)
sudo rm -f /etc/nginx/sites-enabled/default

# Test Nginx configuration syntax
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx
```

---

### 🔒 5. Certbot Installation & SSL Domain Configuration

Certbot is an automated tool that fetches free, auto-renewing SSL certificates from **Let's Encrypt** and configures HTTPS on Nginx.

#### Step 5.1: Install Certbot & Nginx Plugin

```bash
sudo apt update
sudo apt install -y certbot python3-certbot-nginx
```

#### Step 5.2: Connect SSL Certificate to Your Domain

Make sure your domain's DNS A-record (e.g. `istanadoludeed.com`) points to your server's IP address (`207.154.247.234`).

- **Obtain & Configure SSL for Single Domain**:

  ```bash
  sudo certbot --nginx -d istanadoludeed.com
  ```

- **Obtain & Configure SSL for Domain + Subdomains (e.g., `www`)**:

  ```bash
  sudo certbot --nginx -d istanadoludeed.com -d www.istanadoludeed.com
  ```

- **Non-Interactive / Automated Setup**:

  ```bash
  sudo certbot --nginx -d istanadoludeed.com --non-interactive --agree-tos -m admin@istanadoludeed.com
  ```

*Certbot will automatically update your `/etc/nginx/sites-available/deed` file to enable SSL on port 443 and add automatic HTTP-to-HTTPS redirection.*

---

### 🔄 6. Updating & Renewing SSL Certificates

Let's Encrypt certificates are valid for **90 days**. Certbot handles automatic renewals via a systemd timer.

#### Test Automatic Renewal (Dry Run)

Test that the renewal process works properly without modifying certificates:

```bash
sudo certbot renew --dry-run
```

#### Manual Renewal

To force renewal of all certificates near expiration:

```bash
sudo certbot renew
```

#### Expand / Add New Subdomains to Existing Certificate

To add a new subdomain (e.g., `www.istanadoludeed.com`) to an existing certificate:

```bash
sudo certbot --nginx --expand -d istanadoludeed.com -d www.istanadoludeed.com
```

#### Check Active Certificates Status

View installed certificates, domain coverage, and expiration dates:

```bash
sudo certbot certificates
```

#### Revoke or Delete a Certificate

If you no longer need a certificate for a domain:

```bash
sudo certbot delete --cert-name istanadoludeed.com
```

---

### ☁️ 7. Cloudflare Integration & Error 521 Prevention

If using **Cloudflare** as a DNS / CDN proxy:

1. **SSL/TLS Mode in Cloudflare**:
   - Set to **Full** or **Full (Strict)** in the Cloudflare Dashboard (`SSL/TLS` -> `Overview`).
   - If set to *Full*, Cloudflare communicates securely with Port 443 on your origin server (which is active once Certbot is configured).
2. **Error 521 Fix**:
   - **Error 521 ("Web server is down")** occurs when Cloudflare SSL is set to *Full*, but Port 443 is not enabled on your server. Running `sudo certbot --nginx -d yourdomain.com` opens Port 443 and resolves Error 521 instantly.

---

### 🔧 Useful Management Commands Cheat Sheet

| Task | Command |
| :--- | :--- |
| **Start Nginx** | `sudo systemctl start nginx` |
| **Reload Nginx Config** | `sudo systemctl reload nginx` |
| **Test Nginx Syntax** | `sudo nginx -t` |
| **Obtain SSL Certificate** | `sudo certbot --nginx -d domain.com` |
| **Test SSL Auto-Renewal** | `sudo certbot renew --dry-run` |
| **List Installed SSL Certs** | `sudo certbot certificates` |
| **View Nginx Access Log** | `sudo tail -f /var/log/nginx/access.log` |
| **View Nginx Error Log** | `sudo tail -f /var/log/nginx/error.log` |
| **View Certbot Debug Log** | `sudo tail -f /var/log/letsencrypt/letsencrypt.log` |
