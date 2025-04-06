# Frankenphp/Caddy extended build

Extended build with additional module, and php extension.

## Available Images

This repository provides extended FrankenPHP Docker images with various PHP versions:

- PHP 8.2: `Dockerfile.php82` and `Dockerfile.php82.alpine`
- PHP 8.3: `Dockerfile.php83` and `Dockerfile.php83.alpine`
- PHP 8.4: `Dockerfile.php84` and `Dockerfile.php84.alpine`

## Build Examples

### Building PHP 8.2 Image

```bash
# Standard build
docker build -t frankenphp-extended:php82 -f Dockerfile.php82 .

# Alpine variant
docker build -t frankenphp-extended:php82-alpine -f Dockerfile.php82.alpine .
```

### Building PHP 8.3 Image

```bash
# Standard build
docker build -t frankenphp-extended:php83 -f Dockerfile.php83 .

# Alpine variant
docker build -t frankenphp-extended:php83-alpine -f Dockerfile.php83.alpine .
```

### Building PHP 8.4 Image

```bash
# Standard build
docker build -t frankenphp-extended:php84 -f Dockerfile.php84 .

# Alpine variant
docker build -t frankenphp-extended:php84-alpine -f Dockerfile.php84.alpine .
```

## Using the Images

After building, you can run your FrankenPHP container:

```bash
docker run -p 80:80 -p 443:443 -v $(pwd):/app frankenphp-extended:php83
```

You can also pull the pre-built images from Docker Hub:

```bash
docker pull kirimemail/frankenphp-extended:php83
# or
docker pull kirimemail/frankenphp-extended:php83-alpine
```