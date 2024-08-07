#
#--------------------------------------------------------------------------
# php 7.2 fpm slim
#--------------------------------------------------------------------------
#

FROM chinayin/php:7.2-fpm
LABEL maintainer="chinayin <whereismoney@qq.com>"

ENV PHP_VERSION 7.2.34 \
    CYD_APP_NAME="7.2-fpm-slim" \
    CYD_IMAGE_VERSION="7.2.34-buster-slim-r2"

## Configure non-root user.
ARG PUID=1000
ARG PGID=1000
ENV PUID=${PUID} PGID=${PGID}
RUN groupmod -o -g ${PGID} www-data && usermod -o -u ${PUID} -g www-data www-data

COPY files/arms /usr/local/arms/arms-php-agent

RUN set -eux \
    && pecl channel-update pecl.php.net \
    ## install packages
    && install_packages \
        curl iputils-ping wget telnet vim apt-utils net-tools procps unzip dnsutils \
        libssl-dev libmcrypt-dev libzip-dev \
    ## Install extensions
    && docker-php-ext-install -j$(nproc) pdo_mysql fileinfo exif zip bcmath\
    ## Install uuid
    && install_packages uuid-dev && pecl install uuid && docker-php-ext-enable uuid \
    ## Install redis
    && pecl install -o -f redis && docker-php-ext-enable redis \
    ## Install memcached
    && install_packages libmemcached-dev && pecl install igbinary msgpack memcached && docker-php-ext-enable memcached \
    ## Install aliyun arms
    && mv /usr/local/arms/arms-php-agent/arms.ini /usr/local/etc/php/conf.d/arms.ini \
    ## Clean up
    && pecl clear-cache \
    && rm -rf /tmp/* /var/tmp/* \
    && rm -f /var/log/lastlog /var/log/faillog \
    ## Check PHP version
    && php -v
