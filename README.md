# WordPress Automated Deployment with GitHub Actions and LEMP Stack

This project demonstrates an automated deployment process for a WordPress website using AWS, the LEMP stack (Linux, Nginx, MySQL, PHP), and GitHub Actions for CI/CD.


## Prerequisites

1. **AWS EC2 Instance**:
   - Ubuntu 22.04 LTS
   - Minimum 2 vCPUs, 4 GB RAM
2. **GitHub Repository**:
   - For version control and workflow automation.
3. **Domain Name**:
   - Register a domain or use a free service like [No-IP](https://www.noip.com/).
4. **GitHub Secrets**:
   - Store sensitive information like server IP, username, and password.

---

## Server Provisioning

1. **Provision an EC2 Instance**:
   - Launch an Ubuntu 22.04 instance.
   - Open the necessary ports (22, 80, 443) in the security group.
   - Access the server via SSH.

2. **Secure the Server**:
   ```bash
   sudo apt update && sudo apt upgrade -y
   sudo ufw allow OpenSSH
   ```

---

## LEMP Stack Installation

1. **Install Nginx**:
   ```bash
   sudo apt install nginx -y
   sudo systemctl start nginx
   sudo systemctl enable nginx
   ```

2. **Install MySQL**:
   ```bash
   sudo apt install mysql-server -y
   sudo mysql_secure_installation
   ```

3. **Install PHP**:
   ```bash
   sudo apt install php-fpm php-mysql -y
   ```

4. **Verify Setup**:
   Create a test PHP file:
   ```bash
   echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/info.php
   ```
   Visit `http://65.0.18.235/info.php` to confirm PHP is working.

---

## WordPress Configuration

1. **Download WordPress**:
   ```bash
   wget https://wordpress.org/latest.tar.gz
   tar -xvf latest.tar.gz
   sudo mv wordpress /var/www/html/
   ```

2. **Set Up the Database**:
   ```bash
   sudo mysql -u root -p
   ```

   Run the following SQL commands:
   ```sql
   CREATE DATABASE wordpress_db;
   CREATE USER 'wordpress_user'@'localhost' IDENTIFIED BY 'your_password';
   GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wordpress_user'@'localhost';
   FLUSH PRIVILEGES;
   EXIT;
   ```

3. **Configure WordPress**:
   ```bash
   sudo cp /var/www/html/wordpress/wp-config-sample.php /var/www/html/wordpress/wp-config.php
   sudo nano /var/www/html/wordpress/wp-config.php
   ```
   Update the database credentials:
   ```php
   define( 'DB_NAME', 'wordpress_db' );
   define( 'DB_USER', 'wordpress_user' );
   define( 'DB_PASSWORD', 'your_password' );
   ```

4. **Set Permissions**:
   ```bash
   sudo chown -R www-data:www-data /var/www/html/wordpress
   sudo chmod -R 755 /var/www/html/wordpress
   ```

---

## Securing the Website with SSL

1. **Install Certbot**:
   ```bash
   sudo apt install certbot python3-certbot-nginx -y
   ```

2. **Obtain SSL Certificate**:
   ```bash
   sudo certbot --nginx -d lempwordpress.ddns.net
   ```

3. **Verify Renewal**:
   ```bash
   sudo certbot renew --dry-run
   ```

---

## Nginx Optimization

Edit the Nginx configuration file for WordPress:
```bash
sudo nano /etc/nginx/sites-available/wordpress
```

Add the following:
```nginx
server {
    listen 80;
    server_name lempwordpress.ddns.net;

    root /var/www/html/wordpress;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ /index.php$is_args$args;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
    }

    location ~ /\.ht {
        deny all;
    }

    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
}
```

Enable the configuration and reload Nginx:
```bash
sudo ln -s /etc/nginx/sites-available/wordpress /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## GitHub Actions Workflow

1. **Create Workflow File**:
   Add `.github/workflows/deploy.yml`:
   ```yaml
   name: Deploy WordPress

    on:
    push:
        branches:
        - main

    jobs:
    deploy:
        runs-on: ubuntu-latest

        steps:
        # Checkout the code
        - name: Checkout code
            uses: actions/checkout@v3

        # Set up SSH
        - name: Setup SSH
            uses: webfactory/ssh-agent@v0.5.3
            with:
            ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

        # Deploy to server using SSH
        - name: Deploy to server
            env:
            SERVER_IP: ${{ secrets.SERVER_IP }}
            SERVER_USER: ${{ secrets.SERVER_USER }}
            run: |
            ssh -o StrictHostKeyChecking=no $SERVER_USER@$SERVER_IP << 'EOF'
            cd /var/www/html
            git pull origin main
            sudo systemctl reload nginx
            EOF
   ```

2. **Add Secrets**:
   - `SERVER_IP`: Public IP of the EC2 instance.
   - `SERVER_USER`: SSH username.
   - `SERVER_PASSWORD`: SSH password.

---

## Deployment Steps

1. Push changes to `main`:
   ```bash
   git add .
   git commit -m "Update WordPress"
   git push origin main
   ```

2. GitHub Actions will handle deployment.
---

### Deployment URL

Access the live website: [https://lempwordpress.ddns.net](lempwordpress.ddns.net)