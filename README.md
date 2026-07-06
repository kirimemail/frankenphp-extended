# FrankenPHP Extended

Extended [FrankenPHP](https://frankenphp.dev/) Docker images with additional PHP extensions, debugging tooling, and runtime utilities.

Built on [`dunglas/frankenphp:1.12`](https://github.com/php/frankenphp) (floating tag, latest 1.12.x) and published to Docker Hub as [`kirimemail/frankenphp-extended`](https://hub.docker.com/r/kirimemail/frankenphp-extended).

## Available Images

Two base variants for each PHP version:

| Variant | Base | User | Use case |
|---------|------|------|----------|
| **Debian** (`php8X`) | `dunglas/frankenphp:1.12-php8.X` | `root` | Production — full tooling, runs as root (non-root user applied downstream in your own Dockerfile) |
| **Alpine** (`php8X-alpine`) | `dunglas/frankenphp:1.12-php8.X-alpine` | `app` (UID 1000) | Development — smaller image, non‑root user with `sudo`, supervisor pre‑configured |

PHP versions: **8.2**, **8.3**, **8.4**, **8.5**

## Quick Start

```bash
# Pull from Docker Hub
docker pull kirimemail/frankenphp-extended:php83

# Run a container
docker run -p 80:80 -p 443:443 -v $(pwd)/app:/app kirimemail/frankenphp-extended:php83

# Use like any FrankenPHP image —
# your PHP app goes in /app, Caddyfile in /etc/caddy/
```

## Building Locally

```bash
# Debian variant
docker build -t frankenphp-extended:php83 -f Dockerfile.php83 .

# Alpine variant
docker build -t frankenphp-extended:php83-alpine -f Dockerfile.php83.alpine .
```

Replace `php83` with `php82`, `php84`, or `php85` for other versions.

## Docker Compose

```yaml
services:
  frankenphp:
    image: kirimemail/frankenphp-extended:php83-alpine
    ports:
      - "80:80"
      - "443:443"
      - "2019:2019"  # Caddy admin API (optional)
    volumes:
      - ./app:/app
      - ./Caddyfile:/etc/caddy/Caddyfile
      - ./supervisor:/etc/supervisor/conf.d  # Alpine only
    environment:
      - CADY_LOGGING_LEVEL=DEBUG
```

## Included PHP Extensions

| Extension | Purpose |
| ----------- | --------- |
| `pdo_mysql` | MySQL/MariaDB database access |
| `pdo_pgsql` | PostgreSQL database access |
| `mysqli` | MySQL native driver |
| `redis` | Redis key-value store |
| `memcached` | Memcached caching daemon |
| `mongodb` | MongoDB document database |
| `opcache` | PHP bytecode cache |
| `gd` | Image creation and manipulation |
| `imagick` | ImageMagick integration |
| `intl` | Internationalization (ICU) |
| `zip` | ZIP file handling |
| `bcmath` | Arbitrary precision arithmetic |
| `pcntl` | Process control (fork, signal) |
| `soap` | SOAP web services |
| `exif` | EXIF image metadata |
| `calendar` | Calendar conversion functions |
| `sockets` | Low-level socket communication |
| `yaml` | YAML serialization |
| `igbinary` | Alternative PHP serializer |
| `amqp` | AMQP messaging (RabbitMQ) |
| `imap` | IMAP/POP3 email access |
| `mailparse` | Email message parsing |
| `excimer` | Profiling/tracing *(not available on PHP 8.5)* |

## Included Debugging & Utility Tools

| Tool | In Debian | In Alpine | Purpose |
| ------ | ----------- | ----------- | --------- |
| `curl`, `wget` | ✅ | ✅ | HTTP requests, downloads |
| `nano` | ✅ | ✅ | Text editor |
| `strace` | ✅ | ✅ | System call tracing |
| `tcpdump` | ✅ | ✅ | Network packet capture |
| `jq` | ✅ | ✅ | JSON command-line processor |
| `traceroute` / `mtr` | ✅ | ✅ | Network route diagnostics |
| `ping` / `telnet` / `netcat` | ✅ | ✅ | Network connectivity testing |
| `dnsutils` / `bind-tools` | ✅ | ✅ | DNS lookup tools (`dig`, `nslookup`) |
| `procps` / `ps` | ✅ | ✅ | Process monitoring |
| `supervisor` | ✅ | ✅ | Process manager |
| `node`, `npm` | ✅ | ✅ | JavaScript runtime |
| `chokidar-cli` | ✅ | ✅ | File watcher (dev use) |
| `imagemagick` | ✅ | ✅ | Image processing |
| `ghostscript` | ✅ | ✅ | PDF/postscript processing |
| `p7zip`, `unzip`, `zip` | ✅ | ✅ | Archive extraction/creation |
| `qpdf` | ✅ | ✅ | PDF transformation |
| `make`, `time` | ✅ | ❌ | Build utilities |
| Perl toolchain | ❌ *(removed)* | ❌ | *(previously included for imapsync)* |

Only the Debian variant includes `build-essential` (available at build time, purged from the final image) and the NodeSource LTS Node.js repository.

## Configuration

### Custom php.ini

A shared `docker/php.ini` is copied into all images:

```ini
upload_max_filesize = 24M
post_max_size      = 32M
max_file_uploads   = 20
memory_limit       = 256M
max_execution_time = 300
max_input_time     = 300
```

Override any setting at runtime by mounting your own `.ini` to `/usr/local/etc/php/conf.d/`.

### Supervisor (Alpine)

The Alpine variant has a `supervisor` configuration directory at `/etc/supervisor/conf.d/` and runs as the `app` user with passwordless `sudo` access. Mount your supervisor configs to start background processes alongside FrankenPHP.

### PsySH

A `/config/psysh` directory is pre‑created for [PsySH](https://psysh.org/) configuration — a runtime PHP debugger and REPL.

## CI/CD

The repository includes a GitHub Actions workflow (`.github/workflows/docker-build.yml`) that:

- Builds **all 8 images** on push to `main`, tags matching `v*.*.*`, or manual trigger
- Builds on PRs (push only, no deploy)
- Builds for **`linux/amd64`** and **`linux/arm64`** via Docker Buildx + QEMU
- Caches layers with GitHub Actions cache (`type=gha`)
- Pushes to Docker Hub under the `kirimemail` organization

Trigger a manual build: **Actions** → **Build and Push Docker Images** → **Run workflow**.

## Image Tags

| Tag | Base | PHP |
| ----- | ------ | ----- |
| `php82` | Debian | 8.2 |
| `php82-alpine` | Alpine | 8.2 |
| `php83` | Debian | 8.3 |
| `php83-alpine` | Alpine | 8.3 |
| `php84` | Debian | 8.4 |
| `php84-alpine` | Alpine | 8.4 |
| `php85` | Debian | 8.5 |
| `php85-alpine` | Alpine | 8.5 |

## License

MIT — see [LICENSE](LICENSE).
