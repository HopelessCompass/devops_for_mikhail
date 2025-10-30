# newapi/Dockerfile
# PHP 8.4 + Composer, API Platform (multi-stage)

### Stage: builder
FROM php:8.4-cli AS builder

RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y \
    git unzip zlib1g-dev libzip-dev libicu-dev libxml2-dev curl && rm -rf /var/lib/apt/lists/*

RUN docker-php-ext-install pdo pdo_mysql zip intl opcache pcntl || true
RUN pecl install redis && docker-php-ext-enable redis || true

COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

WORKDIR /app
COPY newapi/composer.json newapi/composer.lock ./
RUN composer install --no-dev --prefer-dist --no-interaction --optimize-autoloader

COPY newapi/ ./

### Stage: runtime
FROM php:8.4-fpm

RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y libicu-dev libxml2 && rm -rf /var/lib/apt/lists/*

RUN docker-php-ext-install pdo pdo_mysql zip intl opcache pcntl || true
RUN pecl install redis && docker-php-ext-enable redis || true

WORKDIR /var/www/newapi
COPY --from=builder /app /var/www/newapi

RUN chown -R www-data:www-data /var/www/newapi

EXPOSE 9000
CMD ["php-fpm"]
