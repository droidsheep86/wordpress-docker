# WordPress Docker Development Environment

A complete Docker Compose setup for WordPress development with Apache, Nginx reverse proxy, configurable PHP/MariaDB versions, WP-CLI, Redis, and MailHog.

## Features

- ✅ WordPress on Apache with configurable PHP versions (8.1, 8.2, 8.3)
- ✅ Nginx reverse proxy serving `wordpress.local`
- ✅ MariaDB with configurable versions (10.6, 10.11, 11.0, 11.1)
- ✅ WP-CLI for command-line WordPress management
- ✅ Redis for object caching
- ✅ MailHog for email testing
- ✅ PHPMyAdmin for database management (optional)

## Prerequisites

- Docker and Docker Compose installed
- Ability to modify your hosts file

## Directory Structure

Create this folder structure:

```
wordpress-docker/
├── docker-compose.yml          # Main Docker Compose configuration
├── .env                        # Environment variables
├── INSTRUCTIONS.md                  # This file
├── CHANGELOG.md
├── nginx/
│   ├── nginx.conf             # Nginx main configuration
│   └── sites/
│       └── wordpress.conf     # WordPress site configuration
├── mysql/
│   └── custom.cnf             # MariaDB custom configuration
├── uploads.ini                # PHP upload settings
└── wordpress/                # WordPress directory
```

## Quick Setup

### 1. Add Local Domain

Add this line to your hosts file:

- **Linux/Mac:** `/etc/hosts`
```
127.0.0.1 wordpress.local
```
### 2. Change .env files
```
Adjust .env variables

Defaults are

# Database Configuration
DB_NAME=wordpress
DB_USER=wordpress
DB_PASSWORD=wordpress
DB_ROOT_PASSWORD=rootpassword

# WordPress Site Details
WP_URL=http://wordpress.local
WP_TITLE="Wordpress"
WP_ADMIN_USER=droid
WP_ADMIN_PASSWORD=@Popopoko0
WP_ADMIN_EMAIL=droidsheep86@gmail.com
```

### 3. Start Environment

```bash
# Start all services
docker-compose up -d

# Optional: Start with PHPMyAdmin
docker-compose --profile tools up -d
```

### 3. Install WordPress

```bash
# Wait for services to start (about 30 seconds), then install WordPress
docker-compose run --rm wpcli wp core install \
  --url="http://wordpress.local" \
  --title="My WordPress Site" \
  --admin_user="admin" \
  --admin_password="admin123" \
  --admin_email="admin@example.com"
```

## Access Your Services

- **WordPress:** http://wordpress.local
- **WordPress Admin:** http://wordpress.local/wp-admin (admin/admin123)
- **MailHog:** http://localhost:8025
- **PHPMyAdmin:** http://localhost:8080 (if using `--profile tools`)

## Configuration

### Change PHP or MariaDB Versions

Edit the `.env` file:

```bash
# Available PHP versions: php8.1, php8.2, php8.3
PHP_VERSION=php8.2

# Available MariaDB versions: 10.6, 10.11, 11.0, 11.1
MARIADB_VERSION=10.11
```

Apply changes:

```bash
docker-compose down
docker-compose pull
docker-compose up -d
```

## WP-CLI Usage

```bash
# List all plugins
docker-compose run --rm wpcli wp plugin list

# Install and activate a plugin
docker-compose run --rm wpcli wp plugin install redis-cache --activate

# Enable Redis cache
docker-compose run --rm wpcli wp redis enable

# Create a new user
docker-compose run --rm wpcli wp user create developer dev@example.com --role=administrator

# Update WordPress core
docker-compose run --rm wpcli wp core update

# Flush rewrite rules
docker-compose run --rm wpcli wp rewrite flush

# Export/Import database
docker-compose run --rm wpcli wp db export backup.sql
docker-compose run --rm wpcli wp db import backup.sql
```

## Email Testing with MailHog

**Install WP Mail SMTP Plugin:**

```bash
docker-compose run --rm wpcli wp plugin install wp-mail-smtp --activate
```

**Configure SMTP Settings in WordPress:**

- SMTP Host: `mailhog`
- SMTP Port: `1025`
- Authentication: None
- Encryption: None

View Emails: [http://localhost:8025](http://localhost:8025)

## Redis Cache Setup

**Install Redis Object Cache Plugin:**

```bash
docker-compose run --rm wpcli wp plugin install redis-cache --activate
```

**Enable Redis:**

```bash
docker-compose run --rm wpcli wp redis enable
```

**Check Status:**

```bash
docker-compose run --rm wpcli wp redis status
```

## Management Commands

### Start/Stop Environment

```bash
# Start all services
docker-compose up -d

# Stop all services
docker-compose down

# Stop and remove all data (volumes) (deletes everything except wordpress files which are bind!)
docker-compose down -v

# Remove data from wordpress folder
sudo rm -rf ./wordpress/*
```

### View Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f wordpress
docker-compose logs -f nginx
docker-compose logs -f mariadb
```

### Restart Services

```bash
# Restart specific service
docker-compose restart wordpress
docker-compose restart nginx

# Restart all services
docker-compose restart
```

## Database Management

### Backup Database

```bash
# Using WP-CLI
docker-compose run --rm wpcli wp db export backup.sql

# Using mysqldump
docker-compose exec mariadb mysqldump -u wordpress -p wordpress > backup.sql
```

### Restore Database

```bash
# Using WP-CLI
docker-compose run --rm wpcli wp db import backup.sql

# Using mysql
docker-compose exec -i mariadb mysql -u wordpress -p wordpress < backup.sql
```

### Access Database Directly

```bash
docker-compose exec mariadb mysql -u wordpress -p wordpress
```

## File Management

- **Themes:** `wp-content/themes/`
- **Plugins:** `wp-content/plugins/`
- **Uploads:** `wp-content/uploads/`

### Fix Permissions (if needed)

```bash
sudo chown -R $USER:$USER wp-content/
chmod -R 755 wp-content/
```

## Troubleshooting

### Common Issues

**WordPress not accessible:**

- Check hosts file entry: `127.0.0.1 wordpress.local`
- Verify services are running: `docker-compose ps`
- Check nginx logs: `docker-compose logs nginx`

**Database connection error:**

- Wait for MariaDB to fully start (can take 30-60 seconds)
- Check database logs: `docker-compose logs mariadb`
- Verify credentials in `.env` file

**Permission errors:**

```bash
sudo chown -R $USER:$USER wp-content/
chmod -R 755 wp-content/
```

**Email not working:**

- Ensure MailHog is running: `docker-compose ps mailhog`
- Check MailHog interface: [http://localhost:8025](http://localhost:8025)
- Verify SMTP settings in WordPress

### Reset Everything

```bash
# Stop and remove all containers and volumes
docker-compose down -v

# Remove all Docker images (optional)
docker-compose down --rmi all

# Start fresh
docker-compose up -d
```

## Development Tips

### Debugging

- WordPress debug logs: `wp-content/debug.log`
- PHP errors in Docker logs: `docker-compose logs wordpress`
- Enable `WP_DEBUG` in `wp-config.php` (already enabled in this setup)

### Performance

- Redis cache is configured and ready to use
- Nginx serves static files efficiently
- MariaDB is optimized for development

### Security

- Change default passwords in `.env` for production
- Disable debug mode for production
- Use strong passwords for WordPress admin

## Step-by-Step Setup Summary

1. Create folders:  
   `mkdir -p wordpress-docker/{nginx/sites,mysql,wp-content/{themes,plugins,uploads}}`
2. Copy files: Save all configuration files from the artifacts above
3. Add to hosts:  
   `echo "127.0.0.1 wordpress.local" >> /etc/hosts` (Linux/Mac)
4. Start Docker:  
   `docker-compose up -d`
5. Install WordPress: Use the WP-CLI command provided above
6. Access site: [http://wordpress.local](http://wordpress.local)

---

Happy WordPress development! 🚀
# Stop all services
docker-compose down

# Stop and remove all data (careful!)
docker-compose down -v
```

### View Logs

```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f wordpress
docker-compose logs -f nginx
docker-compose logs -f mariadb
```

### Restart Services

```bash
# Restart specific service
docker-compose restart wordpress
docker-compose restart nginx

# Restart all services
docker-compose restart
```

## Database Management

### Backup Database

```bash
# Using WP-CLI
docker-compose run --rm wpcli wp db export backup.sql

# Using mysqldump
docker-compose exec mariadb mysqldump -u wordpress -p wordpress > backup.sql
```

### Restore Database

```bash
# Using WP-CLI
docker-compose run --rm wpcli wp db import backup.sql

# Using mysql
docker-compose exec -i mariadb mysql -u wordpress -p wordpress < backup.sql
```

### Access Database Directly

```bash
docker-compose exec mariadb mysql -u wordpress -p wordpress
```

## File Management

- **Themes:** `wp-content/themes/`
- **Plugins:** `wp-content/plugins/`
- **Uploads:** `wp-content/uploads/`

### Fix Permissions (if needed)

```bash
sudo chown -R $USER:$USER wp-content/
chmod -R 755 wp-content/
```

## Troubleshooting

### Common Issues

**WordPress not accessible:**

- Check hosts file entry: `127.0.0.1 wordpress.local`
- Verify services are running: `docker-compose ps`
- Check nginx logs: `docker-compose logs nginx`

**Database connection error:**

- Wait for MariaDB to fully start (can take 30-60 seconds)
- Check database logs: `docker-compose logs mariadb`
- Verify credentials in `.env` file

**Permission errors:**

```bash
sudo chown -R $USER:$USER wp-content/
chmod -R 755 wp-content/
```

**Email not working:**

- Ensure MailHog is running: `docker-compose ps mailhog`
- Check MailHog interface: [http://localhost:8025](http://localhost:8025)
- Verify SMTP settings in WordPress

### Reset Everything

```bash
# Stop and remove all containers and volumes
docker-compose down -v

# Remove all Docker images (optional)
docker-compose down --rmi all

# Start fresh
docker-compose up -d
```

## Development Tips

### Debugging

- WordPress debug logs: `wp-content/debug.log`
- PHP errors in Docker logs: `docker-compose logs wordpress`
- Enable `WP_DEBUG` in `wp-config.php` (already enabled in this setup)

### Performance

- Redis cache is configured and ready to use
- Nginx serves static files efficiently
- MariaDB is optimized for development

### Security

- Change default passwords in `.env` for production
- Disable debug mode for production
- Use strong passwords for WordPress admin

## Step-by-Step Setup Summary

1. Create folders:  
   `mkdir -p wordpress-docker/{nginx/sites,mysql,wp-content/{themes,plugins,uploads}}`
2. Copy files: Save all configuration files from the artifacts above
3. Add to hosts:  
   `echo "127.0.0.1 wordpress.local" >> /etc/hosts` (Linux/Mac)
4. Start Docker:  
   `docker-compose up -d`
5. Install WordPress: Use the WP-CLI command provided above
6. Access site: [http://wordpress.local](http://wordpress.local)

---

Happy WordPress development! 🚀

