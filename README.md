# PHP Base Image

[![Docker Image CI](https://github.com/chinayin-docker/php/actions/workflows/ci.yml/badge.svg?event=schedule)](https://github.com/chinayin-docker/php/actions/workflows/ci.yml)
![Docker Image Version (latest semver)](https://img.shields.io/docker/v/chinayin/php?sort=semver)
![Docker Image Size (latest semver)](https://img.shields.io/docker/image-size/chinayin/php?sort=semver)
![Docker Pulls](https://img.shields.io/docker/pulls/chinayin/php)

While designed for web development, the PHP scripting language also provides general-purpose use.

### Supported tags and respective `Dockerfile` links

![](https://img.shields.io/docker/v/chinayin/php/7.2-fpm)
![](https://img.shields.io/docker/v/chinayin/php/7.2-cli)
![](https://img.shields.io/docker/v/chinayin/php/7.2-fpm-slim)
![](https://img.shields.io/docker/v/chinayin/php/7.2-cli-slim)

![](https://img.shields.io/docker/v/chinayin/php/7.4-fpm)
![](https://img.shields.io/docker/v/chinayin/php/7.4-cli)
![](https://img.shields.io/docker/v/chinayin/php/7.4-fpm-slim)
![](https://img.shields.io/docker/v/chinayin/php/7.4-cli-slim)

### Image Variants

- `php:<version>-cli`
- `php:<version>-fpm`
- `php:<version>-cli-slim`
- `php:<version>-fpm-slim`

### Use PHP

You can use the image directly, e.g.

```
docker run --rm -it chinayin/php:7.2-fpm
```

The images are built daily and have the security release enabled, so will contain any security updates released more
than 24 hours ago.

You can also use the images as a base for your own Dockerfile:

```
FROM chinayin/php:7.2-fpm
```

### Use PHP slim version

You can use the image directly, e.g.

```
docker run --rm -it chinayin/php:7.2-fpm-slim
```

You can also use the images as a base for your own Dockerfile:

```
FROM chinayin/php:7.2-fpm-slim
```
