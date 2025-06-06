
## Apache Setup for hosting a single site with HTTPS and Custom Logs

## **Steps**

## **Part1: Hosting a Website at port 80**

## Pre-req
Add A type of Records in the DNS in your Registrar's Dashboard

```bash
A @ <Public IP> 3600
A www @ 3600
```

### 1. Install Apache on Ubuntu

```bash
apt update
apt install apache2 -y
```
### 2. Create a Web Directory
This is where we copy the files and folders of the website given by the developer
```bash
mkdir -p /var/www/www.cicdnow.com
```

**You should keep the website code in:**

    /var/www/www.cicdnow.com/

This is the **clean and scalable** approach when you're hosting **multiple websites** (using Apache virtual hosts), because:

-   `/var/www/` is the standard root directory.
    
-   Each domain gets its own subfolder, e.g.:
    
    -   `/var/www/example.com/`
        
    -   `/var/www/www.cicdnow.com/`
        
    -   `/var/www/another-site.net/`

### 3. Download the website code from Github
You are currently in the user home directory. 

We shall download code here and unzip it, and then we shall copy the code to `/var/www/www.cicdnow.com`

**Downloading a specific file (like `website.zip`) from a GitHub repo, the correct raw format is:**
```bash
https://github.com/<user>/<repo>/raw/<branch>/<path-to-file>
```
**Our GitHub Repo is:**
```bash
https://github.com/devopsmatic/apache-single-site
```
**Now constructing the actual RAW URL for download**
```bash
https://github.com/devopsmatic/apache-single-site/raw/main/website.zip

```
#### ✅ Download Using `curl` or `wget`

#### **With `curl`:**

```bash
curl -L -o website.zip https://github.com/devopsmatic/apache-single-site/raw/main/website.zip
```

#### **With `wget`:**
```bash
wget https://github.com/devopsmatic/apache-single-site/raw/main/website.zip` 
```
-   `-L` in `curl` ensures it follows redirects.
    
-   `wget` does this automatically.

### 4. Unzip
```bash
sudo apt install unzip -y
unzip website.zip
sudo cp -r website/* /var/www/html/www.cicdnow.com/
```
### 5. Set ownership and permissions for Apache
```bash
sudo chown -R www-data:www-data /var/www/www.cicdnow.com/
sudo chmod -R 755 /var/www/www.cicdnow.com/
```

### 6. Create a Log Directory
This is where Apache will be put the website logs
```bash
mkdir -p /var/www/www.cicdnow.com/logs
```

### 7. Configuring Apache to host the website
**Virtual Host File**
This is the file where we configure a website configuration.

The file should be put in this directory: 

    /etc/apache2/sites-available/

Name of the Virtual Host File is usually the same name as your website address.
For example: **www.cicdnow.com.conf**

If you are hosting Multiple websites in the same server you find multiple Virtual Host Files as:
```bash
/etc/apache2/sites-available/site1.com.conf
/etc/apache2/sites-available/site2.com.conf
/etc/apache2/sites-available/site3.com.conf
/etc/apache2/sites-available/site4.com.conf
/etc/apache2/sites-available/site5.com.conf
```
**Virtual Host File contents**
**Apache Virtual Host configuration file** for the HTML site at `www.cicdnow.com`, assuming:

-   Website files are in: `/var/www/www.cicdnow.com/`
    
-   Logs go to: `/var/www/www.cicdnow.com/logs/`
    
-   Domain: `www.cicdnow.com` or `cicdnow.com`
    
-   Hosting on port 80 (HTTP)

File: `/etc/apache2/sites-available/www.cicdnow.com.conf`
```bash
<VirtualHost *:80>
    ServerName www.cicdnow.com
    ServerAlias cicdnow.com
    DocumentRoot /var/www/www.cicdnow.com

    # Directory settings
    <Directory /var/www/www.cicdnow.com>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    # Custom log files
    ErrorLog /var/www/www.cicdnow.com/logs/error.log
    CustomLog /var/www/www.cicdnow.com/logs/access.log combined
</VirtualHost>
```
### Summary Table

| Directive              | Meaning                                      |
|------------------------|----------------------------------------------|
| `Options Indexes`      | List files in browser if no index file       |
| `Options FollowSymLinks` | Allow symbolic links to be followed        |
| `AllowOverride All`    | Enable `.htaccess` to override settings      |
| `Require all granted`  | Allow access to all users (no restrictions)  |

## Cybersecurity Tip
### ❌ Should You Avoid `Options Indexes`?

Yes — in most cases, it's **better to avoid `Options Indexes`**, especially on a production server.

---

### 🚫 Why You Should Avoid `Options Indexes`

| Concern                | Explanation                                                                 |
|------------------------|-----------------------------------------------------------------------------|
| 🔓 **Security risk**    | If a directory doesn’t have an `index.html`, Apache will list **all files** — even sensitive ones (e.g., config, backups). |
| 🕵️‍♂️ **Information leakage** | Attackers can see file names, versions, structure — which can help them plan attacks. |
| 🧼 **Unprofessional appearance** | File listings look raw and unstyled; not user-friendly.                 |

---

### ✅ When to Use `Indexes` (If Ever)

| Use Case                       | Example                                     |
|--------------------------------|---------------------------------------------|
| You're intentionally sharing files | Like an open downloads folder or file repository |
| Internal/test environment only     | Temporary debugging or dev purpose         |
| You're using `.htaccess` to **limit access** | With authentication or IP filtering in place |

---

### 🔒 Safer Alternative

Instead of:

```apache
Options Indexes FollowSymLinks
```
Use
```apache
Options -Indexes +FollowSymLinks
```
or simply
```apache
Options FollowSymLinks
```
🛡️ Visitors see a **403 Forbidden** if there's no index file

### 🚫 Default Site vs. Named Virtual Host

-   Apache ships with a **default site**:  
    `/etc/apache2/sites-available/000-default.conf` with `DocumentRoot /var/www/html`

**Disable that site**
```bash
sudo a2dissite 000-default.conf
```
**Enable  our site**
```bash
sudo a2ensite www.cicdnow.com.conf
sudo systemctl reload apache2
```

## **Part2: Hosting a Website at port 443**
**Using a Free SSL Certificate by Let's Encrypt**
```bash
apt  install  -y  certbot  python3-certbot-apache
certbot  --apache
```
**Follow the onscreen instructions!**

Access the website `https://www.cicdnow.com`

