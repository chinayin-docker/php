#
#--------------------------------------------------------------------------
# php 7.4 fpm slim
#--------------------------------------------------------------------------
#

FROM chinayin/php:7.4-fpm
LABEL maintainer="chinayin <whereismoney@qq.com>"

ENV PHP_VERSION 7.4.22 \
    CYD_APP_NAME="7.4-fpm-slim" \
    CYD_IMAGE_VERSION="7.4.22-buster-slim-r2"

## Configure non-root user.
ARG PUID=1000
ARG PGID=1000
ENV PUID=${PUID} PGID=${PGID}
RUN groupmod -o -g ${PGID} www-data && usermod -o -u ${PUID} -g www-data www-data

RUN set -eux \
    && pecl channel-update pecl.php.net \
    ## install packages
    && install_packages \
        curl iputils-ping wget telnet vim apt-utils net-tools procps unzip dnsutils \
        libssl-dev libmcrypt-dev libzip-dev \
    ## Install PHP opcache
    && docker-php-ext-configure opcache --enable-opcache && docker-php-ext-install -j$(nproc) opcache \
    && echo "opcache.enable=0" >> "$PHP_INI_DIR/conf.d/docker-php-ext-opcache.ini" \
    ## Install extensions
    && docker-php-ext-install -j$(nproc) pdo_mysql fileinfo exif zip bcmath\
    ## Install uuid
    && install_packages uuid-dev && pecl install uuid && docker-php-ext-enable uuid \
    ## Install redis
    && pecl install -o -f redis && docker-php-ext-enable redis \
    ## Install memcached
    && install_packages libmemcached-dev && pecl install igbinary msgpack memcached && docker-php-ext-enable memcached \
    ## Clean up
    && pecl clear-cache \
    && rm -rf /tmp/* /var/tmp/* \
    && rm -f /var/log/lastlog /var/log/faillog \
    ## Check PHP version
    && php -v
