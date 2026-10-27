# FrankenPHP Extended

Extended [FrankenPHP](https://frankenphp.dev/) Docker images with additional PHP extensions, debugging tooling, and runtime utilities.

Built on [`dunglas/frankenphp:1.13`](https://github.com/php/frankenphp) (floating tag, latest 1.13.x) and published to Docker Hub as [`kirimemail/frankenphp-extended`](https://hub.docker.com/r/kirimemail/frankenphp-extended).

## Available Images

Two base variants for each PHP version:

| Variant | Base | User | Use case |
|---------|------|------|----------|
| **Debian** (`php8X`) | `dunglas/frankenphp:1.13-php8.X` | `root` | Production — full tooling, runs as root (non-root user applied downstream in your own Dockerfile) |
| **Alpine** (`php8X-alpine`) | `dunglas/frankenphp:1.13-php8.X-alpine` | `app` (UID 1000) | Development — smaller image, non‑root user with `sudo`, supervisor pre‑configured |

PHP versions: **8.3**, **8.4**, **8.5**

## Quick Start

```bash
# Pull from Docker Hub
docker pull kirimemail/frankenphp-extended:php83

# Run a container
docker run -p 80:80 -p 443:443 -v $(pwd)/app:/app kirimemail/frankenphp-extended:php83

# Use like any FrankenPHP image —
# your PHP app goes in /app, Caddyfile in /etc/frankenphp/Caddyfile
```

## Building Locally

```bash
# Debian variant
docker build -t frankenphp-extended:php83 -f Dockerfile.php83 .

# Alpine variant
docker build -t frankenphp-extended:php83-alpine -f Dockerfile.php83.alpine .
```

Replace `php83` with `php84` or `php85` for other versions.

### Build performance

- FrankenPHP ships a **ZTS** PHP build, so `install-php-extensions` has no prebuilt binaries and compiles every extension from source. That compilation dominates build time.
- The extension compilation is its own layer and every config `COPY` comes last, so editing `php.ini` or the Caddy hardening snippets rebuilds in seconds without recompiling anything.
- Build one platform at a time locally (`--platform linux/amd64`). Cross-building the other architecture falls back to emulation and is several times slower.
- `opcache` is already compiled into the base image and is deliberately not installed again.

## Docker Compose

```yaml
services:
  frankenphp:
    image: kirimemail/frankenphp-extended:php83-alpine
    ports:
      - "80:80"
      - "443:443"
      - "2019:2019"  # Caddy admin API - local debugging only. Note: the
                      # global-hardening snippet binds it to localhost, so
                      # publishing the port only works without that snippet.
    volumes:
      - ./app:/app
      - ./Caddyfile:/etc/frankenphp/Caddyfile
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

A shared `docker/php.ini` is copied into all images as `/usr/local/etc/php/conf.d/99-custom.ini`: upload/memory limits, error handling and the hardening described in [Hardening & Security](#hardening--security). The Alpine variant additionally installs `zz-dev.ini`, which re-enables `display_errors` for development.

Override any setting at runtime by mounting your own `.ini` into `/usr/local/etc/php/conf.d/` — a file sorting after `99-custom.ini` (e.g. `zz-local.ini`) wins. The Caddyfile `frankenphp` block can also set directives with `php_ini <key> <value>`.

### Supervisor (Alpine)

The Alpine variant has a `supervisor` configuration directory at `/etc/supervisor/conf.d/` and runs as the `app` user with passwordless `sudo` access. Mount your supervisor configs to start background processes alongside FrankenPHP.

### PsySH

A `/config/psysh` directory is pre‑created for [PsySH](https://psysh.org/) configuration — a runtime PHP debugger and REPL.

## Hardening & Security

### PHP — active by default

Every image ships `docker/php.ini` as `conf.d/99-custom.ini`. No action needed:

| Directive | Value | Effect |
| ----------- | ------- | -------- |
| `expose_php` | `Off` | No `X-Powered-By` header, no PHP version disclosure |
| `display_errors`, `display_startup_errors`, `html_errors` | `Off` | Errors never reach the response body (no leaked paths, DSNs or credentials) |
| `log_errors` / `error_log` | `On` / `/dev/stderr` | Errors remain visible through `docker logs` |
| `zend.exception_ignore_args` | `On` | Function arguments (tokens, passwords) are not copied into stack traces |
| `allow_url_include` | `Off` | No inclusion of remote code |
| `enable_dl` / `phar.readonly` / `ffi.enable` | `Off` / `On` / `Off` | No runtime native-code loading, no writable PHARs, no FFI |
| `user_ini.filename` | `""` | `.user.ini` files in the docroot cannot override configuration |
| `session.*` | strict mode, cookies only, `HttpOnly`, `SameSite=Lax`, `upload_progress.enabled=0` | Session fixation / theft and session-flooding hardening |
| `mail.add_x_header` | `Off` | No PHP binary path leaked into mail headers |

`session.cookie_secure` stays `0` so plain-HTTP development works; set it to `1` in the application once traffic is HTTPS-only.

The Alpine ("Development") variant adds `conf.d/zz-dev.ini`, which turns `display_errors` and exception arguments back on. Do not reuse that file in a production image.

### FrankenPHP / Caddy — opt-in

Two snippets are installed in every image under `/etc/frankenphp/hardening/`:

| File | Import into | Content |
| ------ | ------------- | --------- |
| `security-headers.caddy` | a site block | strips `Server` and `X-Powered-By`, sets `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`, `Permissions-Policy`, `Cross-Origin-Opener-Policy`, `Cross-Origin-Resource-Policy`, `Strict-Transport-Security` |
| `global-hardening.caddy` | the global options block | binds the admin API to `localhost:2019`, `max_header_size 64KB`, `read_header_timeout 10s`, `idle_timeout 120s` (Slowloris protection) |

They are opt-in because they change response behaviour (framing, HSTS, streaming). Enable them without replacing the bundled Caddyfile — the default config already splices these variables in:

```yaml
environment:
  - SERVER_NAME=:80
  - CADDY_GLOBAL_OPTIONS=import /etc/frankenphp/hardening/global-hardening.caddy
  - CADDY_SERVER_EXTRA_DIRECTIVES=import /etc/frankenphp/hardening/security-headers.caddy
```

Or import them from your own Caddyfile:

```caddyfile
{
	import /etc/frankenphp/hardening/global-hardening.caddy
}

example.com {
	import /etc/frankenphp/hardening/security-headers.caddy
	php_server
}
```

Review before production: `X-Frame-Options: SAMEORIGIN` blocks third-party framing, `Cross-Origin-Resource-Policy: same-site` blocks cross-site asset embedding, and `Content-Security-Policy` is deliberately not set (it must match your app's assets and API origins). `write_body` timeouts are left unset so SSE, Mercure and large downloads are not truncated.

### Runtime

- The Debian variants run as `root` so a downstream Dockerfile can create its own user and switch with `USER`. Do not run them as root in production. `frankenphp` has `CAP_NET_BIND_SERVICE` (Alpine) so an unprivileged user can still bind 80/443.
- The Alpine variants run as `app` (UID 1000) but grant **passwordless sudo** — a development convenience, not a production setting.
- Suggested runtime flags: `--security-opt no-new-privileges`, `--cap-drop ALL --cap-add NET_BIND_SERVICE`, `--read-only` with `tmpfs` for `/tmp`, `/data` and `/config`, plus `--memory` and `--pids-limit`.
- Use a floating tag (`php83`) for convenience but pin a digest or a `v*.*.*` release tag when reproducibility matters.

## CI/CD

The repository includes a GitHub Actions workflow (`.github/workflows/docker-build.yml`) that:

- Builds **3 PHP versions × 2 variants × 2 architectures** (12 jobs) on push to `main`, on `v*.*.*` tags, on manual dispatch and on pull requests (PRs build only, no push)
- Runs when a `Dockerfile.*`, anything under `docker/` (`php.ini`, Caddy hardening) or the workflow itself changes — config-only edits used to be ignored
- Uses **native runners per architecture** — `ubuntu-latest` for `linux/amd64` and `ubuntu-24.04-arm` for `linux/arm64` — instead of QEMU emulation. Emulated arm64 extension compilation was ~80% of the old 76 minute run; the same matrix now completes in about 5.5 minutes (measured: 323 s for 12 builds + 6 merges)
- Pushes a digest per architecture, then a follow-up merge job assembles the multi-arch manifest for `php83`, `php83-alpine`, …
- Caches layers per image **and** per architecture (`type=gha`, scope `<tag>-<variant>-<arch>`), so the previous single shared cache scope no longer made the jobs evict each other
- Pushes to Docker Hub under the `kirimemail` organization

Trigger a manual build: **Actions** → **Build and Push Docker Images** → **Run workflow**.

## Image Tags

| Tag | Base | PHP |
| ----- | ------ | ----- |
| `php83` | Debian | 8.3 |
| `php83-alpine` | Alpine | 8.3 |
| `php84` | Debian | 8.4 |
| `php84-alpine` | Alpine | 8.4 |
| `php85` | Debian | 8.5 |
| `php85-alpine` | Alpine | 8.5 |

## License

MIT — see [LICENSE](LICENSE).
