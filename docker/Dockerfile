FROM php:8.2-fpm

## Diretório da aplicação
ARG APP_DIR=/var/www

## Versão da Lib do Redis para PHP
ARG REDIS_LIB_VERSION=6.0.2

### apt-utils é um extensão de recursos do gerenciador de pacotes APT
RUN apt-get update -y && apt-get install -y --no-install-recommends \
    apt-utils \
    supervisor

# dependências recomendadas de desenvolvido para ambiente linux
RUN apt-get update && apt-get install -y \
    curl \
    libzip-dev \
    zlib1g-dev \
    libpq-dev \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev \
    libonig-dev \
    libxml2-dev \
    zip \
    unzip

# Instalar extenções do php
RUN docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install pdo_mysql zip iconv mbstring simplexml exif pcntl bcmath gd sockets fileinfo

# habilita instalação do Redis
RUN pecl install redis-${REDIS_LIB_VERSION} \
    && docker-php-ext-enable redis

# Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

#RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

COPY ./docker/supervisord/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
#COPY ./docker/supervisord/conf /etc/supervisord.d/
# Supervisor permite monitorar e controlar vários processos (LINUX)
# Bastante utilizado para manter processos em Daemon, ou seja, executando em segundo plano

COPY ./docker/php/custom.ini /usr/local/etc/php/conf.d/custom.ini
COPY ./docker/php/extra-php-fpm.conf /etc/php8/php-fpm.d/www.conf

WORKDIR $APP_DIR
RUN cd $APP_DIR
RUN chown www-data:www-data $APP_DIR

COPY --chown=www-data:www-data . .
RUN rm -rf vendor
RUN composer install --no-interaction

RUN apt-get install nginx -y
RUN rm -rf /etc/nginx/sites-enabled/* && rm -rf /etc/nginx/sites-available/*
COPY ./docker/nginx/sites.conf /etc/nginx/sites-enabled/default.conf
COPY ./docker/nginx/error.html /var/www/html/error.html

RUN apt-get clean && rm -rf /var/lib/apt/lists/*

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisord.conf"]
