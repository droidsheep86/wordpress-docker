# WordPress Docker Development Environment

A complete Docker Compose setup for WordPress development with Apache, Nginx reverse proxy, configurable PHP/MariaDB versions, WP-CLI, Redis, MailHog, and optional PHPMyAdmin.

---

## Features

* ✅ WordPress on Apache with configurable PHP versions (8.1, 8.2, 8.3)
* ✅ Nginx reverse proxy serving `wordpress.local`
* ✅ MariaDB with configurable versions (10.6, 10.11, 11.0, 11.1)
* ✅ WP-CLI for command-line WordPress management
* ✅ Redis for object caching
* ✅ MailHog for email testing
* ✅ PHPMyAdmin for database management (optional)

---

## Prerequisites

1. **Docker & Docker Compose** installed on your machine.
2. Ability to modify your OS hosts file (e.g., `/etc/hosts` on Linux/Mac or `C:\Windows\System32\drivers\etc\hosts` on Windows).
3. (Optional) Basic familiarity with WP-CLI, but we’ll provide examples.

---

## Directory Structure

From your project root (e.g., `wordpress-docker/`), create the following layout:

```
wordpress-docker/
├── docker-compose.yml          # Main Docker Compose configuration
├── .env                        # Environment variables
├── INSTRUCTIONS.md             # This file
├── CHANGELOG.md                # (Optional) Track changes over time
├── nginx/
│   ├── nginx.conf              # Nginx main configuration (global)
│   └── sites/
│       └── wordpress.conf      # WordPress site configuration
├── mysql/
│   └── custom.cnf              # MariaDB custom configuration
├── uploads.ini                 # PHP upload settings (e.g., upload_max_filesize)
└── wordpress/                  # WordPress files (bind-mounted)
    ├── wp-content/
    │   ├── plugins/
    │   ├── themes/
    │   └── uploads/
    └── (other WP core files)
```

> **Tip:** If you prefer, you can rename `wordpress/` to something else, but ensure your `docker-compose.yml` volume paths match.

---

## .env Configuration

Create a `.env` file in the root (`wordpress-docker/.env`). Example contents:

```dotenv
# PHP Version: choose one of php8.1, php8.2, php8.3
PHP_VERSION=php8.2

# MariaDB Version: choose one of 10.6, 10.11, 11.0, 11.1
MARIADB_VERSION=10.11

# Database Credentials
DB_NAME=wordpress
DB_USER=wordpress
DB_PASSWORD=wordpress
DB_ROOT_PASSWORD=rootpassword

# WordPress Site Details
WP_URL=http://wordpress.local
WP_TITLE="My WordPress Site"
WP_ADMIN_USER=admin
WP_ADMIN_PASSWORD=admin123
WP_ADMIN_EMAIL=admin@example.com
```

* Adjust values as you like.
* After changing versions (e.g., `PHP_VERSION` or `MARIADB_VERSION`), you’ll need to rebuild/recreate containers (see below).

---

## Hosts File Entry

Add `wordpress.local` to your hosts file so it resolves to localhost:

* **Linux/Mac**:

  ```bash
  sudo -- sh -c "echo '127.0.0.1 wordpress.local' >> /etc/hosts"
  ```
* **Windows**:
  Edit `C:\Windows\System32\drivers\etc\hosts` as Administrator and add:

  ```
  127.0.0.1 wordpress.local
  ```

> **Note:** If `wordpress.local` conflicts with another entry, pick another hostname (e.g., `wp.local`) and update both `.env` and Nginx config accordingly.

---

## docker-compose.yml Overview

The `docker-compose.yml` should define services such as:

* **nginx** (reverse proxy)
* **wordpress** (Apache + PHP-FPM container)
* **mariadb** (database)
* **wpcli** (for running WP-CLI commands)
* **redis** (for object caching)
* **mailhog** (for email testing)

---

## Start Environment

1. From project root:

   ```bash
   docker-compose up -d
   ```
2. (Optional) To start tools like PHPMyAdmin:

   ```bash
   docker-compose --profile tools up -d
   ```
3. Wait \~20–30 seconds for all containers (especially MariaDB) to be ready.

---

## Install WordPress via WP-CLI

Run once after containers are up:

```bash
docker-compose run --rm wpcli wp core install \
  --url="$WP_URL" \
  --title="$WP_TITLE" \
  --admin_user="$WP_ADMIN_USER" \
  --admin_password="$WP_ADMIN_PASSWORD" \
  --admin_email="$WP_ADMIN_EMAIL"
```

* If you change site details, rerun with updated flags.
* You can also import an existing database or export backup later.

---

## Accessing Services

* **WordPress Frontend**: [http://wordpress.local](http://wordpress.local)
* **WordPress Admin**: [http://wordpress.local/wp-admin](http://wordpress.local/wp-admin) (use your admin credentials)
* **MailHog UI**: [http://localhost:8025](http://localhost:8025)
* **PHPMyAdmin**: [http://localhost:8080](http://localhost:8080) (if started under `--profile tools`)

> If you pick a different hostname/port, adjust URLs accordingly.

---

## Changing PHP or MariaDB Versions

1. Edit `.env`, set `PHP_VERSION` or `MARIADB_VERSION` to desired value.
2. Recreate containers:

   ```bash
   docker-compose down
   docker-compose pull          # fetch updated images
   docker-compose up -d
   ```
3. Re-run WP-CLI install or migrations if needed.
4. Check functionality; sometimes PHP version changes require plugin/theme compatibility checks.

---

## WP-CLI Usage Examples

* **List all plugins**

  ```bash
  docker-compose run --rm wpcli wp plugin list
  ```
* **Install & activate a plugin**

  ```bash
  docker-compose run --rm wpcli wp plugin install redis-cache --activate
  ```
* **Enable Redis object cache**

  ```bash
  docker-compose run --rm wpcli wp redis enable
  ```
* **Check Redis status**

  ```bash
  docker-compose run --rm wpcli wp redis status
  ```
* **Create a new admin user**

  ```bash
  docker-compose run --rm wpcli wp user create developer dev@example.com --role=administrator
  ```
* **Update WordPress core**

  ```bash
  docker-compose run --rm wpcli wp core update
  ```
* **Flush rewrite rules**

  ```bash
  docker-compose run --rm wpcli wp rewrite flush
  ```
* **Database export/import**

  ```bash
  docker-compose run --rm wpcli wp db export backup.sql
  docker-compose run --rm wpcli wp db import backup.sql
  ```

---

## Email Testing with MailHog

1. **Install SMTP plugin** (e.g., WP Mail SMTP):

   ```bash
   docker-compose run --rm wpcli wp plugin install wp-mail-smtp --activate
   ```
2. **Configure in WP Admin** under SMTP settings:

   * SMTP Host: `mailhog`
   * SMTP Port: `1025`
   * Authentication: None
   * Encryption: None
3. Send a test email from WordPress; view it at [http://localhost:8025](http://localhost:8025).

---

## Redis Object Cache Setup

1. **Install Redis Object Cache plugin**:

   ```bash
   docker-compose run --rm wpcli wp plugin install redis-cache --activate
   ```
2. **Enable Redis**:

   ```bash
   docker-compose run --rm wpcli wp redis enable
   ```
3. **Verify**:

   ```bash
   docker-compose run --rm wpcli wp redis status
   ```
4. Ensure your `docker-compose.yml` has a `redis` service and WordPress PHP container can reach it (host: `redis`, default port 6379).

---

## PHPMyAdmin (Optional)

* Define PHPMyAdmin under a Compose profile, e.g.:

  ```yaml
  services:
    phpmyadmin:
      image: phpmyadmin/phpmyadmin
      environment:
        PMA_HOST: mariadb
        PMA_USER: root
        PMA_PASSWORD: ${DB_ROOT_PASSWORD}
      ports:
        - "8080:80"
      networks:
        - your_network_name
  ```
* Start with:

  ```bash
  docker-compose --profile tools up -d phpmyadmin
  ```
* Access at: [http://localhost:8080](http://localhost:8080)

---

## Management Commands

* **Start all services**

  ```bash
  docker-compose up -d
  ```
* **Stop all services**

  ```bash
  docker-compose down
  ```
* **Stop & remove containers + volumes (destructive)**

  ```bash
  docker-compose down -v
  ```
* **Restart specific service**

  ```bash
  docker-compose restart wordpress
  docker-compose restart nginx
  ```
* **View logs**

  ```bash
  docker-compose logs -f           # all services
  docker-compose logs -f wordpress  # WordPress container
  docker-compose logs -f nginx      # Nginx container
  docker-compose logs -f mariadb    # MariaDB logs
  ```

---

## Database Management

* **Backup (WP-CLI)**

  ```bash
  docker-compose run --rm wpcli wp db export backup.sql
  ```
* **Backup (mysqldump)**

  ```bash
  docker-compose exec mariadb mysqldump -u ${DB_USER} -p${DB_PASSWORD} ${DB_NAME} > backup.sql
  ```
* **Restore (WP-CLI)**

  ```bash
  docker-compose run --rm wpcli wp db import backup.sql
  ```
* **Restore (mysql)**

  ```bash
  docker-compose exec -i mariadb mysql -u ${DB_USER} -p${DB_PASSWORD} ${DB_NAME} < backup.sql
  ```
* **Access MySQL CLI**

  ```bash
  docker-compose exec mariadb mysql -u ${DB_USER} -p${DB_PASSWORD} ${DB_NAME}
  ```

> Replace `${DB_USER}`, `${DB_PASSWORD}`, `${DB_NAME}` with your `.env` values.

---

## File & Permissions

* **Themes:** `wordpress/wp-content/themes/`
* **Plugins:** `wordpress/wp-content/plugins/`
* **Uploads:** `wordpress/wp-content/uploads/`

If you run into permission issues (inside container or local mounting), you can:

```bash
sudo chown -R $USER:$USER wordpress/wp-content/
chmod -R 755 wordpress/wp-content/
```

Adjust ownership/group if necessary for your OS/docker setup.

---

## Troubleshooting

### WordPress Not Accessible

* Check hosts entry: ensure `127.0.0.1 wordpress.local` (or your chosen hostname) is present.
* Confirm containers are running: `docker-compose ps`.
* Inspect Nginx logs: `docker-compose logs nginx`.
* Verify WordPress container logs: `docker-compose logs wordpress`.

### Database Connection Errors

* Wait a bit: MariaDB can take 20–30 seconds to initialize on cold start.
* Check MariaDB logs: `docker-compose logs mariadb`.
* Verify credentials in `.env` match what `docker-compose.yml` expects.

### Permission Errors

* Run permission fix commands shown above.
* Ensure bind-mounted volumes do not override container-internal permissions unexpectedly.

### Email Not Arriving

* Confirm MailHog container is running: `docker-compose ps`.
* Check SMTP settings in WP match `mailhog:1025`.
* View emails at [http://localhost:8025](http://localhost:8025).

### Redis Issues

* Ensure Redis service is up.
* Check WordPress plugin status: `docker-compose run --rm wpcli wp redis status`.
* Confirm network connectivity (service name `redis` in Compose matches WP config).

### Other Common Fixes

* Rebuild images if Dockerfile or PHP extensions changed:

  ```bash
  docker-compose build --no-cache wordpress
  docker-compose up -d
  ```
* If ports conflict, adjust in `docker-compose.yml` (e.g., change host port mapping).
* Clear caches: restart containers, flush Redis if needed.

---

## Development Tips

* **Debug Logging**:

  * Ensure `WP_DEBUG` is enabled in `wp-config.php` (you can mount a customized `wp-config.php` or set via environment).
  * Monitor `wp-content/debug.log` for PHP/WordPress errors.
  * Check container logs: `docker-compose logs wordpress`.

* **Performance**:

  * Use Redis object cache during development for realistic caching behavior.
  * Nginx reverse proxy handles static file serving; ensure your static assets (CSS/JS) are loaded via Nginx.

* **Security**:

  * For production-like environments, change default passwords in `.env`.
  * Disable debug mode for production.
  * Use strong credentials, and consider SSL (e.g., mkcert or self-signed cert for `wordpress.local`).

* **Version Control**:

  * Add `docker-compose.yml`, `nginx/`, `mysql/custom.cnf`, `uploads.ini`, and other config files to Git.
  * Avoid committing sensitive credentials; consider a `.env.example` in repo and add `.env` to `.gitignore`.

* **Customization**:

  * You can add extra services (e.g., Adminer, Elasticsearch) by extending `docker-compose.yml`.
  * Modify PHP settings (`uploads.ini`) by mounting into the PHP container.

---

## Final Notes

* This setup is tailored for local development. For staging or production, adapt security, scaling, SSL, backups, etc.
* If something breaks, start small: check individual container logs, try rebuilding images, confirm network settings.
* Keep `.env` secrets secure. For team use, share a `.env.example` without real credentials.