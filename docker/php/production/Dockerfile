FROM php:8.2.17-fpm-alpine3.19 as base

RUN apk add --no-cache \
    # for zip extension
    libzip libzip-dev \
    # for composer \
    unzip \
    # for sockets extension
    linux-headers \
    # for gd extension
    freetype libjpeg-turbo freetype-dev libjpeg-turbo-dev \
    # for healthcheck
    fcgi \
    # for redis extension
    $PHPIZE_DEPS && \
    # configure and install extensions
    docker-php-ext-configure pcntl --enable-pcntl && \
    docker-php-ext-configure gd --with-freetype --with-jpeg && \
    docker-php-ext-install -j$(nproc) bcmath pdo_mysql zip opcache sockets pcntl gd && \
    pecl install redis &&  \
    docker-php-ext-enable redis &&  \
    # clean up
    apk del --purge libzip-dev freetype-dev libjpeg-turbo-dev $PHPIZE_DEPS

RUN mv "$PHP_INI_DIR/php.ini-production" "$PHP_INI_DIR/php.ini"

FROM base as app

COPY ./docker/php/production/custom-php.ini "$PHP_INI_DIR/conf.d"
COPY ./docker/php/www.conf /usr/local/etc/php-fpm.d/www.conf
COPY ./docker/php/php-fpm-healthcheck /usr/local/bin/

RUN chmod +x /usr/local/bin/php-fpm-healthcheck

HEALTHCHECK CMD ["php-fpm-healthcheck"]

RUN mkdir -p /var/www/html

RUN mkdir -p /var/www/.config/psysh && \
    chown www-data:www-data -R /var/www/.config/psysh

COPY --from=composer:2.6.6 /usr/bin/composer /usr/bin/composer

COPY --chown=www-data . /var/www/html

USER www-data

WORKDIR /var/www/html

RUN composer install --quiet --optimize-autoloader --no-dev --prefer-dist
