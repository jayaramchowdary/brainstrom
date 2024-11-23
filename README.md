# Automated WordPress Deployment with Nginx (LEMP Stack) and GitHub Actions

This project automates the deployment of a WordPress website on a Virtual Private Server (VPS) using the LEMP (Linux, Nginx, MySQL, PHP) stack and GitHub Actions for CI/CD automation. It ensures the website is secured, optimized for performance, and easy to deploy.

## Prerequisites

- **Cloud Server**: AWS EC2 instance with Ubuntu 22.04.
- **GitHub Account**: To manage the repository and GitHub Actions workflow.
- **Domain Name**: A registered domain name pointing to the EC2 instance IP. You can use a service like [NoIP](https://www.noip.com/) for a free temporary hostname.
- **SSH Access**: Ensure SSH access to your EC2 instance is configured.

## Server Provisioning

### 1. Provision a VPS with AWS EC2

- Sign in to [AWS](https://aws.amazon.com/).
- Launch an EC2 instance with Ubuntu 22.04.
- Assign an Elastic IP to the instance to ensure a static IP.
- Set up security groups to allow HTTP (80), HTTPS (443), and SSH (22) traffic.
- SSH into the instance:
  ```bash
  ssh -i wordpress.pem ubuntu@65.2.175.160
  ```

### 2. Install Dependencies

On your EC2 instance, update the package list and install the necessary software for the LEMP stack.

```bash
sudo apt update
sudo apt install php-fpm php-mysql mysql-server nginx unzip wget curl
```

### 3. Configure Nginx to Serve WordPress

Replace the default Nginx configuration:

```bash
cd /etc/nginx/sites-available
sudo rm default
sudo nano default
```

Add the following configuration:

```nginx
server {
    listen 80;
    server_name 65.2.175.160;

    root /var/www/html;
    index index.php index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

Restart Nginx:

```bash
sudo systemctl restart nginx.service
```

## Website Configuration

### 1. Download and Set Up WordPress

- Navigate to `/var/www/html` and download WordPress:
  ```bash
  cd /var/www/html
  sudo wget https://wordpress.org/latest.zip
  sudo unzip latest.zip
  sudo rm latest.zip
  sudo mv wordpress/* .
  sudo chown -R www-data:www-data *
  ```

### 2. Secure MySQL Installation

Run the MySQL secure installation script:

```bash
sudo mysql_secure_installation
```

### 3. Set Up Database and User

Access MySQL and create the WordPress database and user:

```bash
sudo mysql
CREATE DATABASE wordpress_db;
CREATE USER 'wordpress_user'@'localhost' IDENTIFIED BY 'P@55word';
GRANT ALL PRIVILEGES ON wordpress_db.* TO 'wordpress_user'@'localhost';
QUIT;
```

## SSL/TLS Certificate Implementation

### 1. Install Certbot for SSL

Install Certbot and the Nginx plugin:

```bash
sudo apt install certbot python3-certbot-nginx
```

Obtain and install the SSL certificate:

```bash
sudo certbot --nginx -d lempwordpress.ddns.net
```

This will automatically configure SSL for Nginx, redirect HTTP to HTTPS, and configure the SSL certificate for your domain.

### 2. Verify SSL Installation

After completing the Certbot process, visit `lempwordpress.ddns.net` to verify the SSL certificate is installed and working.

## GitHub Actions CI/CD Setup

### 1. Create a GitHub Repository

- Create a new repository on GitHub and push your local code to it.

### 2. Set Up GitHub Actions Workflow

Create the `.github/workflows/deploy.yml` file for automated deployment.

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
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Set up SSH keys
        uses: webfactory/ssh-agent@v0.5.3
        with:
          ssh-private-key: ${{ secrets.SSH_PRIVATE_KEY }}

      - name: Deploy to server
        run: |
          ssh -o StrictHostKeyChecking=no ubuntu@65.2.175.160 "cd /var/www/html && git pull origin main && sudo systemctl restart nginx"
```

### 3. Trigger Deployment

Push changes to the `main` branch to trigger the GitHub Actions workflow, automatically deploying updates to your WordPress site.

## Conclusion

This setup provides a fully automated WordPress deployment process using the LEMP stack, GitHub Actions, and best practices for security and performance optimization. Follow the steps to set up your VPS, configure the LEMP stack, automate deployments using GitHub Actions, and secure your website with SSL.