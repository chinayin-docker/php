#
#--------------------------------------------------------------------------
# php 7.4 fpm slim
#--------------------------------------------------------------------------
#

FROM chinayin/php:7.4-fpm
LABEL maintainer="chinayin <whereismoney@qq.com>"

ENV PHP_VERSION 7.4.16 \
    CYD_APP_NAME="7.4-cli-slim" \
    CYD_IMAGE_VERSION="7.4.16-cli-slim-r1"

RUN set -eux \
    ## install packages
    && install_packages \
        curl iputils-ping wget telnet vim apt-utils net-tools procps unzip dnsutils \
        libssl-dev libmcrypt-dev libzip-dev \
    && pecl channel-update pecl.php.net \
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
