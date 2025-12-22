# Nankoyo Team Website & Management System - Project Documentation

# Nankoyo Team Website & Management System - Project Documentation

## Document Overview

This document serves as the official guide for the closed-source team website and management system developed with PHP. It covers project architecture, environment requirements, installation & deployment, feature modules, maintenance protocols, and collaboration guidelines. Designed for developers, system administrators, and content managers, this Markdown document can be rendered in the website's "Documentation Center" via the integrated Parsedown parser.

---

## 1. Project Overview

### 1.1 Project Positioning

- **Public Website**: Serves clients, partners, and the public—showcasing team capabilities, products/services, case studies, news, and contact channels to enhance brand visibility.

- **Management System**: Empowers internal administrators to manage website content, control user permissions, and analyze data, enabling non-technical teams to operate efficiently.

- **Closed-Source Nature**: Source code is proprietary and restricted to authorized team members. Core business logic, API protocols, and database schemas are strictly confidential.

### 1.2 Core Technology Stack

|Category|Technology Selection|Description|
|---|---|---|
|Backend|PHP 8.2+, MySQL 8.0+|Native PHP development (framework-free) for lightweight deployment|
|Frontend|HTML5, Tailwind CSS v3, JavaScript|Responsive design supporting desktop/mobile devices|
|Markdown Parsing|Parsedown 1.7+|Converts Markdown to HTML for documentation center rendering|
|Storage|Local File System, MySQL|Static assets stored locally; business data in database|
|Server|Nginx 1.20+, PHP-FPM|High-performance web server configuration|
---

## 2. Environment Requirements

### 2.1 Server Environment

- Operating System: Linux (CentOS 7+/Ubuntu 20.04+)

- PHP Version: 8.2+ (Required extensions: PDO, MySQLi, GD, Fileinfo, mbstring)

- MySQL Version: 8.0+ (Supports utf8mb4 character set)

- Nginx Version: 1.20+

- Memory: ≥ 2GB

- Storage: ≥ 20GB (including logs and static asset storage)

- Network: Port 80/443 access; SSL certificate required

### 2.2 Local Development Environment

- Recommended Tools: XAMPP/WAMP (Windows), MAMP (Mac), Docker Containers

- Version Control: Git (follow Git Flow for team collaboration)

- Editor: VS Code (Recommended extensions: PHP Intelephense, Tailwind CSS IntelliSense)

---

## 3. Project Structure

### 3.1 Directory Layout

```Plain Text

project-root/
├── app/                  # Core business logic
│   ├── Admin/            # Management system module (permissions, content management)
│   │   ├── Controller/   # Request handlers
│   │   ├── Model/        # Data models
│   │   ├── View/         # Template files
│   │   └── docs/         # Module-specific documentation
│   ├── Web/              # Public website module (homepage, products, cases)
│   │   ├── Controller/
│   │   ├── Model/
│   │   └── View/
│   └── Common/           # Shared components (utils, constants, middleware)
├── config/               # Configuration files (database, cache, permissions)
│   ├── database.php      # Database connection settings
│   ├── app.php           # Application configuration
│   └── auth.php          # Authentication settings
├── docs/                 # Project documentation (this file's location)
│   ├── guide/            # Operational guides
│   │   ├── install.md    # Installation & deployment guide
│   │   ├── maintain.md   # Maintenance manual
│   │   └── faq.md        # Frequently asked questions
│   ├── api/              # Internal API documentation
│   │   ├── admin_api.md  # Management system APIs
│   │   └── error_codes.md# Error code reference
│   └── structure.md      # Project structure overview
├── public/               # Web root (publicly accessible)
│   ├── index.php         # Application entry point
│   ├── static/           # Static assets (CSS, JS, images)
│   │   ├── css/
│   │   ├── js/
│   │   └── uploads/      # Uploaded files (product images, case studies, etc.)
│   └── favicon.ico       # Website icon
├── vendor/               # Third-party dependencies (Composer-managed)
├── .gitignore            # Git ignore rules
├── composer.json         # Composer configuration
└── README.md             # Quick start guide
```

### 3.2 Key File Descriptions

- `public/index.php`: Application entry point handling routing and environment initialization

- `app/Common/Utils.php`: Shared utility class (string processing, file uploads, data validation)

- `config/database.php`: Database connection settings (encrypt sensitive data in production)

- `vendor/erusev/parsedown`: Markdown parsing dependency for documentation center rendering

---

## 4. Installation & Deployment Guide

### 4.1 Prerequisites

1. Install PHP 8.2+, MySQL 8.0+, Nginx 1.20+ on the server and enable required PHP extensions

2. Configure domain DNS (e.g., `www.nankoyo.com` for the website, `admin.nankoyo.com` for the management system)

3. Obtain an SSL certificate (recommended: Let's Encrypt free certificate)

### 4.2 Deployment Steps

#### 1. Clone Authorized Code Repository

```bash

# Clone private team repository (example URL)
git clone git@github.com:your-team/nankoyo-website.git
cd nankoyo-website
```

#### 2. Install Dependencies

```bash

# Install Composer dependencies (including Parsedown)
composer install --no-dev  # Disable dev dependencies in production
```

#### 3. Configure Environment Variables

Copy the example environment file and update with production settings:

```bash

cp config/.env.example config/.env
# Edit .env to configure database, domain, and security keys
vim config/.env
```

Core configuration example:

```php

DB_HOST=localhost
DB_NAME=nankoyo_website
DB_USER=root
DB_PASS=StrongPassword123!  # Use a strong password in production
SITE_DOMAIN=www.nankoyo.com
ADMIN_DOMAIN=admin.nankoyo.com
APP_KEY=RandomGeneratedString123  # For encryption purposes
```

#### 4. Import Database Schema

```bash

# Execute SQL script to initialize database structure and default data
mysql -u root -p nankoyo_website < sql/init.sql
```

#### 5. Configure Nginx

Create an Nginx configuration file (example path: `/etc/nginx/conf.d/nankoyo.conf`):

```nginx

# Public Website Configuration (www.nankoyo.com)
server {
    listen 80;
    server_name www.nankoyo.com;
    return 301 https://$host$request_uri;  # Redirect HTTP to HTTPS
}

server {
    listen 443 ssl;
    server_name www.nankoyo.com;
    root /var/www/nankoyo-website/public;
    index index.php;

    # SSL Configuration
    ssl_certificate /etc/nginx/ssl/www.nankoyo.com.crt;
    ssl_certificate_key /etc/nginx/ssl/www.nankoyo.com.key;

    # Static Asset Caching
    location ~* \.(css|js|jpg|png|gif)$ {
        expires 7d;
        add_header Cache-Control "public";
    }

    # PHP Processing
    location ~ \.php$ {
        fastcgi_pass unix:/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}

# Management System Configuration (admin.nankoyo.com)
server {
    listen 80;
    server_name admin.nankoyo.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    server_name admin.nankoyo.com;
    root /var/www/nankoyo-website/public;
    index index.php;

    ssl_certificate /etc/nginx/ssl/admin.nankoyo.com.crt;
    ssl_certificate_key /etc/nginx/ssl/admin.nankoyo.com.key;

    # Restrict Access by IP (Optional, Enhanced Security)
    allow 192.168.1.0/24;  # Team internal IP range
    deny all;

    location ~ \.php$ {
        fastcgi_pass unix:/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```

#### 6. Set Directory Permissions

```bash

# Configure permissions for file uploads and cache writing
chmod -R 755 storage/
chmod -R 755 public/static/uploads/
chown -R www-data:www-data /var/www/nankoyo-website
```

#### 7. Test Access

- Access the public website: `https://www.nankoyo.com` to verify homepage loads correctly

- Access the management system: `https://admin.nankoyo.com` and log in with default credentials (Username: admin, Password: 123456 — reset immediately after first login)

### 4.3 Deployment Validation

1. Verify database connection: Navigate to "System Settings" → "Database Check" in the management system

2. Test content publishing: Create a blog post in the management system and confirm it displays on the website

3. Test file upload: Upload a product cover image and verify it appears in `public/static/uploads/`

---

## 5. Feature Module Overview

### 5.1 Public Website Modules (For External Users)

|Module|Core Features|Data Source|
|---|---|---|
|Homepage|Hero section, service showcases, product lists, featured cases, news|Admin system configuration|
|About Us|Team introduction, company history, culture, contact information|Admin system "About Us" section|
|Products|Product listings, detailed pages (specs, screenshots, descriptions)|Admin system "Product Management"|
|Case Studies|Case listings, detailed pages (client, requirements, solutions)|Admin system "Case Management"|
|Blog/News|Article listings, detail pages, tags, search functionality|Admin system "Blog Management"|
|Careers|Job listings, detailed descriptions, resume submission form|Admin system "Career Management"|
|Contact Us|Contact form, map location, contact details|Admin system "Contact Settings"|
|Documentation|Markdown-rendered guides (installation, help docs)|MD files in `docs/` directory|
### 5.2 Management System Modules (For Internal Users)

|Module|Core Features|Permission Control|
|---|---|---|
|Authentication|Login, role assignment, permission management|Super Admin, Content Admin, etc.|
|Dashboard|Data overview (traffic, content count, resumes)|System analytics data|
|Content Management|Homepage configuration, About Us, single-page editing|Content
