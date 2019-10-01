ARG PHP_VERSION
FROM php:${PHP_VERSION}
LABEL maintainer="@小小只 <soinco@qq.com>"

ARG DEFAULT_TIMEZONE
ARG timezone
ARG CODE_DIR
ARG PHP_HOST_PORT
ARG PHP_HOST_PORT1
ARG PHP_HOST_PORT2
ARG PHP_HOST_PORT3
ARG PHP_PHP_CONF_FILE
ARG PHP_LOG_DIR
ARG PECL_EXTENSIONS
ARG PHPREDIS_VERSION
ARG SWOOLE_VERSION
ARG APP_ENV

ARG work_user=php

ENV APP_ENV=${APP_ENV:-"prod"} \
    TIMEZONE=${timezone:-"Asia/Shanghai"} \
    PHPREDIS_VERSION=5.0.2 \
    SWOOLE_VERSION=4.4.7 \
    COMPOSER_ALLOW_SUPERUSER=1

# Libs -y --no-install-recommends
RUN apt-get update \
    && apt-get install -y \
    curl wget git zip unzip less vim procps lsof tcpdump htop openssl \
    libz-dev \
    libzip-dev \
    libssl-dev \
    libnghttp2-dev \
    libpcre3-dev \
    libjpeg-dev \
    libpng-dev \
    libicu-dev \
    libxml2-dev \
    libfreetype6-dev \
    libtidy-dev \
    libssl-dev \
    libxslt-dev \
    libc-client-dev \
    argon2 \
    libargon2-dev \
    # Install PHP extensions
    && docker-php-ext-install \
    bcmath intl dba gd gettext mysqli pcntl shmop soap pdo pdo_mysql mbstring sockets session zip sysvmsg sysvsem sysvshm xmlrpc tidy tokenizer xml xsl fileinfo gettext iconv wddx exif \
    && rm -rf /var/lib/apt/lists/*

# Install composer
RUN curl -sS https://getcomposer.org/installer | php \
    && mv composer.phar /usr/local/bin/composer \
    && composer self-update --clean-backups \
    # Install redis extension
    && wget http://pecl.php.net/get/redis-${PHPREDIS_VERSION}.tgz -O /tmp/redis.tar.tgz \
    && pecl install /tmp/redis.tar.tgz \
    && rm -rf /tmp/redis.tar.tgz \
    && docker-php-ext-enable redis \
    # Install swoole extension
    && wget https://github.com/swoole/swoole-src/archive/v${SWOOLE_VERSION}.tar.gz -O swoole.tar.gz \
    && mkdir -p swoole \
    && tar -xf swoole.tar.gz -C swoole --strip-components=1 \
    && rm swoole.tar.gz \
    && ( \
    cd swoole \
    && phpize \
    && ./configure --enable-mysqlnd --enable-sockets --enable-openssl --enable-http2 \
    && make -j$(nproc) \
    && make install \
    ) \
    && rm -r swoole \
    && docker-php-ext-enable swoole \
    # Clear dev deps
    && apt-get clean \
    && apt-get purge -y --auto-remove -o APT::AutoRemove::RecommendsImportant=false \
    # Timezone
    && cp /usr/share/zoneinfo/${TIMEZONE} /etc/localtime \
    && echo "${TIMEZONE}" > /etc/timezone \
    && echo "[Date]\ndate.timezone=${TIMEZONE}" > /usr/local/etc/php/conf.d/timezone.ini

ADD . /Sol/code
RUN  cd /Sol/code \
    && composer install \
    && composer clearcache

WORKDIR /Sol/code
EXPOSE ${PHP_HOST_PORT} ${PHP_HOST_PORT1}  ${PHP_HOST_PORT2}  ${PHP_HOST_PORT3}

CMD ["php", "/Sol/code/think", "run"]