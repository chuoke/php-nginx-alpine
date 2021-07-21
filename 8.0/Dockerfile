FROM alpine:edge

LABEL Maintainer="chuoke"

# mirrors.aliyun.com
# mirrors.tuna.tsinghua.edu.cn
ARG ALPINE_APK_REPO=mirrors.aliyun.com
ENV ALPINE_APK_REPO=${ALPINE_APK_REPO}

# swoole 在 testing 库
RUN echo "https://dl-cdn.alpinelinux.org/alpine/edge/testing/" >> /etc/apk/repositories

RUN if [ ${ALPINE_APK_REPO} ]; then \
        sed -i "s/dl-cdn.alpinelinux.org/${ALPINE_APK_REPO}/g" /etc/apk/repositories; \
    fi

RUN apk update \
    && apk upgrade \
    && apk add --no-cache openssl bash curl ca-certificates shadow \
    nodejs py3-setuptools

## Install about PHP

RUN apk update && \
    apk add --no-cache \
    php8 \
    php8-common \
    php8-fpm \
    # Laravel Requirements
    php8-bcmath \
    php8-ctype \
    php8-fileinfo \
    php8-json \
    php8-mbstring \
    php8-openssl \
    php8-pdo \
    php8-tokenizer \
    php8-xml \
    # Base Requirements
    php8-curl \
    php8-dom \
    php8-gd \
    php8-opcache \
    php8-xmlwriter \
    php8-simplexml \
    php8-posix \
    php8-opcache \
    # Other Requirements
    php8-zlib \
    php8-zip \
    php8-sodium \
    php8-phar \
    php8-exif \
    php8-iconv \
    php8-intl \
    php8-session \
    php8-xmlreader \
    php8-ftp \
    php8-sockets \
    php8-calendar \
    php8-mysqli \
    php8-pdo_mysql \
    php8-pdo_sqlite \
    php8-sqlite3 \
    php8-pcntl \
    php8-gettext \
    php8-imap \
    # pecl extensions
    php8-pecl-apcu \
    php8-pecl-redis \
    php8-pecl-imagick \
    # php8-pecl-xdebug \ Can't coexist with Swoole，see Swoole's doc
    php8-pecl-swoole

# Configure PHP-FPM
COPY php-fpm.conf /etc/php8/php-fpm.d/www.conf
COPY php.ini /etc/php8/conf.d/custom.ini

RUN ln -s /usr/bin/php8 /usr/bin/php

# Install composer
ENV COMPOSER_HOME /composer
ENV PATH ./vendor/bin:/composer/vendor/bin:$PATH
ENV COMPOSER_ALLOW_SUPERUSER 1
RUN php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" && \
    php composer-setup.php --install-dir=/usr/local/bin --filename=composer && \
    php -r "unlink('composer-setup.php');"

# Config Composer packagist repo
ARG COMPOSER_REPO_PACKAGIST=https://mirrors.aliyun.com/composer/
ENV COMPOSER_REPO_PACKAGIST=${COMPOSER_REPO_PACKAGIST}
RUN if [ ${COMPOSER_REPO_PACKAGIST} ]; then \
    composer config -g repo.packagist composer ${COMPOSER_REPO_PACKAGIST} \
;fi


# Install about nginx
RUN apk --no-cache add nginx && \
    if [ -f '/etc/nginx/conf.d/default.conf' ]; then \
        rm /etc/nginx/conf.d/default.conf \
    ;fi

# Configure nginx
COPY nginx.conf /etc/nginx/nginx.conf

# Install supervisor
RUN apk --no-cache add supervisor
# Configure supervisord
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

# Clean
RUN rm /var/cache/apk/*

# ensure www-data user exists
RUN set -x ; \
  addgroup -g 82 -S www-data ; \
  adduser -u 82 -D -S -G www-data www-data && exit 0 ; exit 1

ARG PUID=1000
ENV PUID ${PUID}
ARG PGID=1000
ENV PGID ${PGID}

RUN groupmod -o -g ${PGID} www-data && \
    usermod -o -u ${PUID} -g www-data www-data

RUN groupmod -o -g ${PGID} nginx && \
    usermod -o -u ${PUID} -g nginx nginx

COPY start-container /usr/local/bin/start-container
RUN chmod +x /usr/local/bin/start-container

# Setup document root
RUN mkdir -p /var/www/
RUN chown -R www-data:www-data /var/www/

# Add application
WORKDIR /var/www/

# Expose the port nginx is reachable on
EXPOSE 80

# CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]

ENTRYPOINT ["start-container"]
