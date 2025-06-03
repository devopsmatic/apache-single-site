# 🌐 Apache Setup Script for www.cicdnow.xyz with HTTPS and Custom Logs

This markdown contains a fully automated, step-by-step bash script to:

- Install Apache and Certbot on Ubuntu
- Configure a custom virtual host for `www.cicdnow.xyz`
- Create dedicated log and web directories
- Secure the site with a Let's Encrypt SSL certificate
- Enable auto-renewal and add security headers

> 🖥️ **Run as root or with `sudo` privileges on Ubuntu**

---

```bash
#!/bin/bash

# ---------------------------------------
# Step 1: Install Apache and Certbot
# ---------------------------------------

apt update
apt install apache2 certbot python3-certbot-apache -y

# ---------------------------------------
# Step 2: Create directories
# ---------------------------------------

mkdir -p /var/www/html/www.cicdnow.xyz
mkdir -p /var/www/www.cicdnow.xyz/logs

# ---------------------------------------
# Step 3: Add a default index.html to the website root
# ---------------------------------------

echo "<h1>Welcome to www.cicdnow.xyz</h1>" > /var/www/html/www.cicdnow.xyz/index.html

# ---------------------------------------
# Step 4: Set ownership for web and log directories
# ---------------------------------------

chown -R www-data:www-data /var/www/html/www.cicdnow.xyz
chown -R www-data:www-data /var/www/www.cicdnow.xyz

# ---------------------------------------
# Step 5: Create Apache VirtualHost configuration
# ---------------------------------------

cat <<EOF > /etc/apache2/sites-available/www.cicdnow.xyz.conf
<VirtualHost *:80>
    ServerName www.cicdnow.xyz
    DocumentRoot /var/www/html/www.cicdnow.xyz

    ErrorLog /var/www/www.cicdnow.xyz/logs/error.log
    CustomLog /var/www/www.cicdnow.xyz/logs/access.log combined

    <Directory /var/www/html/www.cicdnow.xyz>
        Options -Indexes +FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>
</VirtualHost>
EOF

# ---------------------------------------
# Step 6: Enable site, disable default, and reload Apache
# ---------------------------------------

a2ensite www.cicdnow.xyz.conf
a2dissite 000-default.conf
systemctl reload apache2

# ---------------------------------------
# Step 7: Obtain and install SSL certificate using Certbot
# ---------------------------------------

certbot --apache -d www.cicdnow.xyz

# ---------------------------------------
# Step 8: Test Certbot auto-renewal
# ---------------------------------------

certbot renew --dry-run

# ---------------------------------------
# Step 9: Add security headers (optional but recommended)
# ---------------------------------------

# Enable headers module
a2enmod headers

# Add security headers to SSL VirtualHost
SSL_CONF="/etc/apache2/sites-available/www.cicdnow.xyz-le-ssl.conf"
if grep -q "Header always set" "$SSL_CONF"; then
  echo "Security headers already exist in SSL config."
else
  sed -i '/<\/VirtualHost>/i \
Header always set Strict-Transport-Security "max-age=63072000; includeSubDomains; preload"\n\
Header always set X-Content-Type-Options "nosniff"\n\
Header always set X-Frame-Options "SAMEORIGIN"\n\
Header always set X-XSS-Protection "1; mode=block"\n' "$SSL_CONF"
fi

# Restart Apache to apply all changes
systemctl restart apache2

echo "✅ Setup complete: https://www.cicdnow.xyz"
