# PHP Base Image

[![Docker Image CI](https://github.com/chinayin-docker/php/actions/workflows/ci.yml/badge.svg?event=schedule)](https://github.com/chinayin-docker/php/actions/workflows/ci.yml)
[![Docker Image CI](https://github.com/chinayin-docker/php/actions/workflows/ci.yml/badge.svg?branch=master&event=status)](https://github.com/chinayin-docker/php/actions/workflows/ci.yml)
![Docker Image Version (latest semver)](https://img.shields.io/docker/v/chinayin/php?sort=semver)
![Docker Image Size (latest semver)](https://img.shields.io/docker/image-size/chinayin/php?sort=semver)
![Docker Pulls](https://img.shields.io/docker/pulls/chinayin/php)

### Supported tags and respective `Dockerfile` links

![](https://img.shields.io/docker/v/chinayin/php/7.2-fpm)
![](https://img.shields.io/docker/v/chinayin/php/7.2-cli)
![](https://img.shields.io/docker/v/chinayin/php/7.2-fpm-slim)
![](https://img.shields.io/docker/v/chinayin/php/7.2-cli-slim)

![](https://img.shields.io/docker/v/chinayin/php/7.4-fpm)
![](https://img.shields.io/docker/v/chinayin/php/7.4-cli)
![](https://img.shields.io/docker/v/chinayin/php/7.4-fpm-slim)
![](https://img.shields.io/docker/v/chinayin/php/7.4-cli-slim)

### Use PHP

You can use the image directly, e.g.

```
docker run --rm -it chinayin/php:7.2-fpm
docker run --rm -it chinayin/php:7.2-cli
```

The images are built daily and have the security release enabled, so will contain any security updates released more
than 24 hours ago.

You can also use the images as a base for your own Dockerfile:

```
FROM chinayin/php:7.2-fpm
FROM chinayin/php:7.2-cli
```

### Use PHP slim version

You can use the image directly, e.g.

```
docker run --rm -it chinayin/php:7.2-fpm-slim
docker run --rm -it chinayin/php:7.2-cli-slim
```

You can also use the images as a base for your own Dockerfile:

```
FROM chinayin/php:7.2-fpm-slim
FROM chinayin/php:7.2-cli-slim
```
