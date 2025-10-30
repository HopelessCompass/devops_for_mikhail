# legacy/Dockerfile
# PHP 7.4 + Composer, сборка ассетов через npm (webpack) если есть

### Stage: builder
FROM php:7.4-cli AS builder

# system deps for composer / node build
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y \
    git unzip zlib1g-dev libzip-dev libicu-dev libxml2-dev \
    nodejs npm curl && rm -rf /var/lib/apt/lists/*

# php extensions build deps
RUN docker-php-ext-install pdo pdo_mysql zip intl opcache

# install redis extension
RUN pecl install redis && docker-php-ext-enable redis

# composer
COPY --from=composer:2 /usr/bin/composer /usr/bin/composer

WORKDIR /app
# copy composer files first for layer caching
COPY legacy/composer.json legacy/composer.lock ./
RUN composer install --no-dev --prefer-dist --no-interaction --optimize-autoloader

# copy frontend package.json if legacy builds frontend assets
COPY legacy/package.json legacy/package-lock.json ./ || true
RUN if [ -f package.json ]; then npm ci && npm run build --if-present; fi

# copy full app
COPY legacy/ ./

### Stage: runtime
FROM php:7.4-fpm-alpine

# install runtime system libs
RUN apk add --no-cache icu-libs libzip

# php extensions
RUN docker-php-ext-install pdo pdo_mysql zip intl opcache || true
# enable redis (from pecl in builder is not available here, install again if needed)
RUN apk add --no-cache autoconf g++ make libtool && pecl install redis && docker-php-ext-enable redis || true

WORKDIR /var/www/legacy
COPY --from=builder /app /var/www/legacy

RUN chown -R www-data:www-data /var/www/legacy

EXPOSE 9000
CMD ["php-fpm"]
