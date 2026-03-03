# syntax=docker/dockerfile:1.7

FROM composer:2 AS vendor
WORKDIR /app

COPY . /src
RUN set -eux; \
    if [ -f /src/composer.json ]; then APP_DIR=/src; \
    elif [ -f /src/oneclicktask-dev/composer.json ]; then APP_DIR=/src/oneclicktask-dev; \
    else echo "composer.json not found in build context"; exit 1; fi; \
    cp -a "$APP_DIR"/. /app; \
    composer install \
    --no-dev \
    --prefer-dist \
    --no-interaction \
    --no-progress \
    --optimize-autoloader \
    --no-scripts; \
    composer dump-autoload --optimize --no-dev --classmap-authoritative

FROM node:22-alpine AS assets
WORKDIR /app

COPY . /src
RUN set -eux; \
    if [ -f /src/package.json ]; then APP_DIR=/src; \
    elif [ -f /src/oneclicktask-dev/package.json ]; then APP_DIR=/src/oneclicktask-dev; \
    else echo "package.json not found in build context"; exit 1; fi; \
    cp -a "$APP_DIR"/. /app; \
    corepack enable; \
    pnpm install --frozen-lockfile; \
    pnpm build

FROM php:8.3-apache AS app
WORKDIR /var/www/html

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        libzip-dev \
        libpng-dev \
        libonig-dev \
        libxml2-dev \
        libsqlite3-dev \
        sqlite3 \
    && docker-php-ext-install -j"$(nproc)" \
        bcmath \
        exif \
        gd \
        mbstring \
        opcache \
        pdo \
        pdo_mysql \
        pdo_sqlite \
        zip \
    && a2enmod rewrite headers \
    && rm -rf /var/lib/apt/lists/*

ENV APACHE_DOCUMENT_ROOT=/var/www/html/public
RUN sed -ri -e 's!/var/www/html!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/sites-available/*.conf \
    && sed -ri -e 's!/var/www/!${APACHE_DOCUMENT_ROOT}!g' /etc/apache2/apache2.conf /etc/apache2/conf-available/*.conf

COPY --from=vendor /app /var/www/html
COPY --from=assets /app/public/build /var/www/html/public/build

RUN chown -R www-data:www-data /var/www/html/storage /var/www/html/bootstrap/cache

ENV APP_ENV=production \
    APP_DEBUG=false \
    APP_URL=https://oct.cv2.mct-dev.de

EXPOSE 80

CMD ["apache2-foreground"]
