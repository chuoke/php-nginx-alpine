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
    nodejs py3-setuptools py3-pip

## Install about PHP

RUN apk update && \
    apk add --no-cache \
    php82 \
    php82-common \
    php82-fpm \
    # Laravel Requirements
    php82-bcmath \
    php82-ctype \
    php82-fileinfo \
    php82-json \
    php82-mbstring \
    php82-openssl \
    php82-pdo \
    php82-tokenizer \
    php82-xml \
    # Base Requirements
    php82-curl \
    php82-dom \
    php82-gd \
    php82-opcache \
    php82-xmlwriter \
    php82-simplexml \
    php82-posix \
    php82-opcache \
    # Other Requirements
    php82-zlib \
    php82-zip \
    php82-sodium \
    php82-phar \
    php82-exif \
    php82-iconv \
    php82-intl \
    php82-session \
    php82-xmlreader \
    php82-ftp \
    php82-sockets \
    php82-calendar \
    php82-mysqli \
    php82-pdo_mysql \
    php82-pdo_sqlite \
    php82-sqlite3 \
    php82-pcntl \
    php82-gettext \
    php82-imap \
    # pecl extensions
    php82-pecl-apcu \
    php82-pecl-redis \
    php8-pecl-imagick \
    # php82-pecl-xdebug \ Can't coexist with Swoole，see Swoole's doc
    php82-pecl-swoole

# Configure PHP-FPM
COPY php-fpm.conf /etc/php82/php-fpm.d/www.conf
COPY php.ini /etc/php82/conf.d/custom.ini

RUN ln -s /usr/bin/php82 /usr/bin/php

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
# RUN apk --no-cache add supervisor
RUN pip install supervisor
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
