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
docker build -t frankenphp-extended:8.2 -f Dockerfile.php82 .

# Alpine variant
docker build -t frankenphp-extended:8.2-alpine -f Dockerfile.php82.alpine .
```

### Building PHP 8.3 Image

```bash
# Standard build
docker build -t frankenphp-extended:8.3 -f Dockerfile.php83 .

# Alpine variant
docker build -t frankenphp-extended:8.3-alpine -f Dockerfile.php83.alpine .
```

### Building PHP 8.4 Image

```bash
# Standard build
docker build -t frankenphp-extended:8.4 -f Dockerfile.php84 .

# Alpine variant
docker build -t frankenphp-extended:8.4-alpine -f Dockerfile.php84.alpine .
```

## Using the Images

After building, you can run your FrankenPHP container:

```bash
docker run -p 80:80 -p 443:443 -v $(pwd):/app frankenphp-extended:8.3
```

Replace the tag with your preferred PHP version.