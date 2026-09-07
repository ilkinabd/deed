# Anadolu Yakası Evde Fizyoterapi & Manuel Terapi Web Site (`deed`)

Anadolu Yakası evde fizyoterapi ve manuel terapi randevu ve bilgilendirme web sitesi.

---

## 🚀 Nginx Installation & Virtual Hosts Setup Guide for Ubuntu 24 (24.04 LTS / 24.10)

This guide covers installing Nginx on **Ubuntu 24**, configuring permissions, deploying static files, creating single/multiple **Virtual Hosts (Server Blocks)**, and enabling HTTPS.

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

# Copy project files (run from the project folder)
sudo cp -r index.html images /var/www/deed/

# Set ownership and permissions
sudo chown -R www-data:www-data /var/www/deed
sudo chmod -R 755 /var/www/deed
```

---

### 4. Creating Virtual Hosts (Server Blocks)

Virtual hosts (Server Blocks) allow hosting **multiple websites or subdomains** (e.g. `site1.com` and `site2.com`) on a single Ubuntu server.

#### A. Directory Structure for Multiple Sites

Create separate root directories for each site:

```bash
sudo mkdir -p /var/www/site1.com/html
sudo mkdir -p /var/www/site2.com/html

sudo chown -R www-data:www-data /var/www/site1.com /var/www/site2.com
sudo chmod -R 755 /var/www/site1.com /var/www/site2.com
```

#### B. Virtual Host Configuration for Site 1 (`site1.com`)

Create a virtual host configuration file in `sites-available`:

```bash
sudo nano /etc/nginx/sites-available/site1.com
```

Add the server block:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name site1.com www.site1.com;

    root /var/www/site1.com/html;
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

#### C. Virtual Host Configuration for Site 2 (`site2.com`)

```bash
sudo nano /etc/nginx/sites-available/site2.com
```

Add the server block:

```nginx
server {
    listen 80;
    listen [::]:80;
    server_name site2.com www.site2.com;

    root /var/www/site2.com/html;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~* \.(png|jpg|jpeg|gif|ico|svg|css|js)$ {
        expires 30d;
        add_header Cache-Control "public, no-transform";
    }

    gzip on;
}
```

---

### 5. Enabling & Managing Virtual Hosts

Nginx uses symbolic links between `sites-available` and `sites-enabled` to activate virtual hosts.

#### Enable a Virtual Host:

```bash
sudo ln -s /etc/nginx/sites-available/site1.com /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/site2.com /etc/nginx/sites-enabled/
```

#### Disable a Virtual Host (Without deleting the config):

```bash
sudo rm /etc/nginx/sites-enabled/site1.com
```

#### Test Configuration & Reload Nginx:

Always test configuration syntax before reloading Nginx:

```bash
# Test Nginx syntax
sudo nginx -t

# Reload Nginx to apply changes without downtime
sudo systemctl reload nginx
```

---

### 6. (Optional) Enable Free SSL (HTTPS) for Virtual Hosts with Certbot

Secure each virtual host with Let's Encrypt SSL:

```bash
sudo apt install -y certbot python3-certbot-nginx

# Obtain SSL certificates for each domain
sudo certbot --nginx -d site1.com -d www.site1.com
sudo certbot --nginx -d site2.com -d www.site2.com
```

Certbot automatically modifies the corresponding virtual host configuration in `/etc/nginx/sites-available/` to enable HTTPS and HTTP-to-HTTPS redirection.

---

### 🔧 Useful Management Commands

| Action | Command |
| :--- | :--- |
| **Start Nginx** | `sudo systemctl start nginx` |
| **Stop Nginx** | `sudo systemctl stop nginx` |
| **Restart Nginx** | `sudo systemctl restart nginx` |
| **Reload Config** | `sudo systemctl reload nginx` |
| **Test Syntax** | `sudo nginx -t` |
| **List Active Virtual Hosts** | `ls -la /etc/nginx/sites-enabled/` |
| **Access Logs** | `sudo tail -f /var/log/nginx/access.log` |
| **Error Logs** | `sudo tail -f /var/log/nginx/error.log` |
