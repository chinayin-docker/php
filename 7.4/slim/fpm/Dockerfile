#
#--------------------------------------------------------------------------
# php 7.4 fpm slim
#--------------------------------------------------------------------------
#

FROM chinayin/php:7.4.33-fpm
LABEL maintainer="chinayin <whereismoney@qq.com>"

ENV PHP_VERSION=7.4.33 \
    CYD_APP_NAME="7.4-fpm-slim" \
    CYD_IMAGE_VERSION="7.4.33-bullseye-slim"

## Configure non-root user.
ARG PUID=1000
ARG PGID=1000
ENV PUID=${PUID} PGID=${PGID}
RUN groupmod -o -g ${PGID} www-data && usermod -o -u ${PUID} -g www-data www-data

RUN set -eux \
  && pecl channel-update pecl.php.net \
  && docker-php-source extract \
  ## install packages
  && install_packages \
      curl iputils-ping wget telnet vim apt-utils net-tools procps unzip dnsutils tree \
      libssl-dev libmcrypt-dev libzip-dev \
  ## install all required dependencies
    # yaml
      libyaml-dev \
    # uuid
      uuid-dev  \
    # memcached
      libmemcached-dev libsasl2-dev libsasl2-modules \
    # intl
      libicu-dev \
    # gmp
      libgmp-dev \
  ## install extensions
  && docker-php-ext-install -j$(nproc) pdo_mysql fileinfo exif zip bcmath gmp \
  ## ext-opcache
  && ( \
    docker-php-ext-configure opcache --enable-opcache \
    && docker-php-ext-install -j$(nproc) opcache \
    && echo "opcache.enable=0" >> "$PHP_INI_DIR/conf.d/docker-php-ext-opcache.ini" \
  ) \
  ## ext-yaml
  && pecl install yaml && docker-php-ext-enable yaml \
  ## ext-uuid
  && pecl install uuid && docker-php-ext-enable uuid \
  ## ext-intl
  && ( \
    docker-php-ext-configure intl \
    && docker-php-ext-install -j$(nproc) intl \
    && docker-php-ext-enable intl \
  ) \
  ## ext-memcached
  && ( \
    pecl install igbinary msgpack \
    && docker-php-ext-enable igbinary msgpack \
    && pecl bundle -d /usr/src/php/ext memcached \
    && docker-php-ext-configure memcached  \
      --enable-memcached-igbinary  \
      --enable-memcached-sasl \
    && docker-php-ext-install -j$(nproc) memcached \
  ) \
  ## ext-redis
  && ( \
    pecl bundle -d /usr/src/php/ext redis \
    && docker-php-ext-configure redis --enable-redis-igbinary \
    && docker-php-ext-install -j$(nproc) redis \
  ) \
  ## Composer
  && ( \
    curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer \
  ) \
  ## git
  && ( \
    install_packages git \
  ) \
  ## Clean up
  && ( \
    docker-php-source delete \
    && pecl clear-cache \
    && rm -rf /tmp/* /var/tmp/* \
    && rm -f /var/log/lastlog /var/log/faillog \
  ) \
  ## Check version
  && ( \
    php -v \
    && composer -V \
    && git --version \
  )
